---
title: Redis clone in TypeScript
date: '2020-09-01T18:12:16Z'
description: Exploring RESP and simulating Redis
---

I recently became interested in exploring in-memory databases and how they
worked.  I have utilized Redis for quite some time and it offered great
documentation on the [available commands](https://redis.io/commands) and the
[REDIS (REdis Serialization Protocol](https://redis.io/topics/protocol).  So,
I decided to attempt to use this material to build a clone of Redis as an
exercise and continue working with TypeScript.

The project consisted of several interesting challenges, which included:

* Decoding RESP
* Encoding RESP
* Handling client connections
* Executing requests from clients
* Responding back to clients

## RESP

The binary-safe protocol defined by Redis offers an easy to parse and
human-readable format that requires little code to interpret.  The data types
supported by the protocol include simple strings, errors, integers, bulk
strings, and arrays.  Each data type is prefixed by a byte indicating the
data's type, which is then followed by the associated data and terminated with
`\r\n` (CRLF).  For example, `+OK\r\n` serves as a simple string with an OK
message.  The specification offered by Redis details this protocol quite
succinctly.

The client and server utilize RESP as follows to communicate:

1. Server receives command from client as a RESP bulk string
2. Server decodes RESP bulk string into internal data structure
3. Server interprets internal data structure to executure associated command
4. Server responds back to client with appropriate RESP data

The simplest command received to Redis would be the PING command, which would
look like `*1\r\n$4\r\nPING\r\n`.  This particular RESP data represents an
array with a single bulk string consisting of four characters with the word
PING.  A simple way of parsing this request involves reading each character
one-by-one from left to right.

I effectively utilized the following procedure in my implementation:

1. Read the first byte and identify the data type
2. Read the next subset of characters until the CRLF
3. Recurse as needed until the entire payload is read

The following snippet, which I trimmed down, provides insight into a portion of
the implementation, which decodes the various data types.  The `parse` function
identifies the data type based on the prefix, which in turn invokes
`simpleString` to read the data from the payload with the `readNext`
function until it reaches the terminating CRLF.

```typescript
type Token = {
  value: Data;
  offset: number;
};

export function decode(value: Buffer): Data {
  const token = parse(value);
  return token.value;
}

function parse(value: Buffer, offset = 0): Token {
  const prefix = String.fromCharCode(value.readUInt8(offset++));

  switch (prefix) {
    case Prefix.SimpleString:
      return simpleString(value, offset);
    default:
      throw new Error('Protocol error');
  }
}

function simpleString(value: Buffer, offset: number): Token {
  const token = readNext(value, offset);

  return {
    value: token.value,
    offset: token.offset,
  };
}

function readNext(value: Buffer, offset = 0): Token {
  let tokenValue = '';
  let char = '';

  while (char !== CR && offset < value.length) {
    tokenValue += char;
    char = String.fromCharCode(value.readUInt8(offset++));
  }

  try {
    const nb = value.readUInt8(offset++);
    const nextByte = String.fromCharCode(nb);
    if (nextByte !== LF) throw new Error();
  } catch (e) {
    throw new Error('Protocol error: must terminate with CRLF');
  }

  return {
    value: tokenValue,
    offset,
  };
}
```

Encoding values is effectively the reverse and even more straightforward,
which includes prefixing data with the appropriate data type and terminating
the data with CRLFs when needed.

For example for simple string encoding:

```typescript
export function encodeSimpleString(value: string): string {
  return `+${value}\r\n`;
}
```

## Commands

For managing commands within my codebase I figured this might be a good
opportunity to use the
[command pattern](https://en.wikipedia.org/wiki/Command_pattern).  I would
create a single file per command, which I would register with the server and
the server would execute the appropriate command by mapping the received RESP
request to the command.  I effectively stored commands in a map, where the key
was the command name and the value was the instantiated command object.  The
server would lookup the appropriate command and pass the request to it for
execution.

Registering the commands on server startup looked as follows:

```typescript
export default class RedisServer {
  private commands: Map<string, cmd.RedisCommand>;

  constructor() {
    this.commands = new Map();
    this.registerCommands();
  }

  private registerCommands() {
    this.commands.set('echo', new cmd.Echo());
    this.commands.set('get', new cmd.Get());
    this.commands.set('ping', new cmd.Ping());
    this.commands.set('quit', new cmd.Quit());
    this.commands.set('set', new cmd.Set());

    // cloning map to avoid circular dependency
    this.commands.set('command',
        new cmd.Command(new Map(this.commands)));
  }
}
```

After receiving a well-formed RESP encoded request, the server could invoke
`this.commands.get(commandName).execute(request)` to execute the appropriate
command associated with the request.  The actual implementation contains
additional error handling beyond this inline snippet.

To continue with the example of PING from above, the implementation of the
command may look as follows:

```typescript
export default class Ping extends RedisCommand {
  constructor() {
    super('ping', -1, ['stale', 'fast'], 0, 0, 0);
  }

  execute(client: Socket, request: Data[]): void {
    if (request.length > 2) {
      const error =
        `wrong number of arguments for '${this.name}' command`;
      throw new Error(error);
    }

    const message = request[1] ? String(request[1]) : 'PONG';
    const reply = encodeSimpleString(message);
    client.write(reply);
  }
}
```

The command would validate the request is well-formed, interpret it, and
reply.  In this case, PING validates it has no more than two arguments
(including the command name itself).  It then replies with a simple string
containing the word "PONG" or the second argument provided.

## Result

Even though this post only highlights some of the more interesting tidbits of
the implementation, I ended up with a solution that handled several basic
commands and could communicate via the
[redis-cli](https://redis.io/topics/rediscli).

It supports the following commands:

* COMMAND
* ECHO
* GET
* PING
* QUIT
* SET (supports nx and xx)

You can find redis-ts at: [https://github.com/kubejm/redis-ts](https://github.com/kubejm/redis-ts)
