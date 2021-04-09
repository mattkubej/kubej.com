---
title: Graphs with Spidey DB
date: '2021-04-09T08:00:00Z'
description: Playing with C, sockets, SvelteKit, and D3
---

A recent challenge and exploration I engaged in involved building an in-memory
graph database in C.  I sought inspiration from redis, but I have little
experience in C outside of academia and little experience with graph databases.
So, this served itself as a great learning experience to build something of
medium complexity in unfamiliar territory.  Additionally, I have always been impressed by the visualizations that D3 has allowed people to construct, so I desired to give that a swing at illustrating my graph.  Lastly, SvelteKit just had a beta release, so I thought I could learn svelte while I was at it to provide the user interface and serve D3.

Listed below is a small demo of what I pulled together.

IMAGE HERE

Now go into how I overengineered this for educational purposes and fun.

### Building the graph

The journey began by constructing a graph data structure within C.  I desired
to replicate a redis-like experience, which allowed for speedy sets and gets.
Additionally, I wanted to be able to find the nearest neighbors of a vertex
quickly.  With these considerations in mind, I settled on an adjacency list.
This allowed for constant time lookups of vertices.  As for gathering
neighbors, this would take O(v+e) time, where v is the number of neighbors and
e is the number of edges between the root vertex and neighboring vertices.

An adjacency list is effectively a hash map of linked lists. Each entry within
the hash map would represent a vertex and the entry would contain a linked
list, which represents the edges coming from the vertex and to directly
neighboring vertices.  Additionally, these entries could contain the data
associated with the vertex.

```
Example graph:
(a) <---> (b) <---> (c)

Adjacency list representation:
[a]: (b) -->
[b]: (a) --> (c) -->
[c]: (c) -->
```

In the example above, the graph has three vertices (a, b, and c).  Four edges
exists, which consists of a to b, b to a, b to c, and c to b.  The adjaceny
list above illustrautes what a hash map might look like to represent that
particular graph.  It would have an entry for each vertex, where the key is
represented by square brackets.  The value associated with each entry is a
linked list of the edges stemming off of that vertex.  The ordering of the
edges does not matter in my circumstance, because I just want to be able to add
and retrieve them quickly.  Thus, in favor of performance, to add an edge to a
list consists of placing it on the head, so I do not need to forgo iterating
across the list to append to it.

Retrieving a vertex from the adjacency list can be done in constant time due to
the utilization of a hash map data structure.  To find the neighboring edges of
a vertex involves iterating through the linked list of edges associated with
that vertex, which results in O(e) time, where e is the number of edges.

Due to my decision to use C for educational purposes, this required me to build
my own hash map data structure to support my adjacency, which is a luxury
experienced within the standard libraries of many other programming languages.
I came across a great write up on this and effectively replicated the hash map
defined within [!TITLE
HERE!](https://www.cs.yale.edu/homes/aspnes/pinewiki/C(2f)HashTables.html).

### Networking in C

With some success in representing my graph, I moved onto the next piece of the
puzzle, which is how do I communicate with my graph database?  Due to my
previous experience with [RESP](https://redis.io/topics/protocol) and the
simplicitly it offered for communication, I selected this as a communication
protocol over TCP. An additional benefit of this decision meant I could use the
`redis-cli` packaged with redis as my client for communicating with my graph
database, which meant I did not have to bother with constructing a client
during development to communicate with my database.  This served as a large
time saver.

I followed the same pattern and strategy as outlined by redis' RESP
documentation, but I invented my own commands to instruct my graph database.
Listed below outlines the commands I intended to support.  With these in
combination, I could construct a graph and search for neighboring nodes of a
vertex.

| command      | description                          |
| ---          | ---                                  |
| GETVERTEX    | retrieve the value of a vertex       |
| SETVERTEX    | set the value of a vertex            |
| DELVERTEX    | delete a vertex and associated edges |
| SETEDGE      | set an edge, linking two vertices    |
| DELEDGE      | delete an edge, linking two vertices |
| GETNEIGHBORS | retreive the neighbors a vertex      |

For awareness, I did introduce the `COMMAND` command as a means to please
`redis-cli`.  It invokes this command to redis on start-up to learn what
commands are available and it will block the start-up until the message is
received.  So, by introducing this command, it allowed me to connect to my
database without issue.

With a specification for communication and understanding of how I would manage
my graph, I need to be able to accept and handle client communications over the
network.  This required me to roll up my sleeves a little bit and learn socket
programming in C.  One guide I found to be a staple and frequently found in
academia is [Beej's guide](link here), which it proved to be extremely useful.
I will not go into detail of socket programming here, but I leaned heavily on
this guide to accept incoming connections, manage multiple clients, process
requests, and serve responses.  In terms of the codebase, this is likely the
most complex portion of the implementation.  If you would like to learn more,
then I would recommend reading Beej's guide.

Listed below is an example output of some communication with the graph database via `redis-cli`:

```
```

### Searching for Neighbors

The last piece to round out the graph database implementation involved the
ability to search for nearby neighbors of a vertex.  The algorithm I selected
to accomplish this involved using breadth first search with a queue.  The
following list details in the algorithim in short.

1. Enqueue the root vertex into an unvisited queue
2. Dequeue an unvisited vertex
3. Mark the dequeued vertex as visited
4. Iterate through the edges of the dequeued vertex and enqueue each
   neighboring vertex in a queue
5. Repeat steps 2 through 4 until the unvisted queue is empty

During the execution of this algorithim, I saved the processed vertices and
edges to represent the graph subset describing those neighboring the root
vertex. After completion, I could then convert the graph to a RESP
representation to communicate the result back to the requesting client.

Listed below is an example output of the `GETNEIGHBORS` command via `redis-cli`:

```
```

### Visualizing the Graph

Now that I had roughly working graph I wanted to see if I could put it to use
by having a consuming client illustrate the graph with a visualization.  Due to
wanting to explore SvelteKit and D3, this meant communicating with my graph
database via Node and TypeScript.  I previously wrote a basic Redis clone with
TypeScript called [redis-ts](link here), which I could repurpose some of that
code for connecting to my server, producing RESP, and processing RESP.  This
allowed me to build a simple API within TypeScript for talking with my
database.  The code in question can be found [here](link here).  However, if
you're curious about the Redis clone, then I'd recommend taking a look at my
previous blog post called [...](link here).

With this in place, I could programmatically build, retrieve, and search my
graph via TypeScript.  However, I wanted to be able to see this within my
browser.  I set out to come up to speed on SvelteKit, which had a beta release
on ~date here~.  I have previous experience with other utilities such as React
for building user interfaces, but have been hearing more about Svelte and
wanted to form my own opinion of the technology, so I dove right in.

SvelteKit takes a different approach than other popular frameworks at the
moment by...  This allowed me to quickly construct self-contained and
composable components.  Additionally, due to their syntactic decision making
and design, Svelte allowed me to visually present my graph on a webpage with
little code.  Overall, it provided very little barrier of entry to produce
satisfying results.

However, SvelteKit is in beta and I did hit a number of snags or oddities along
the way, which I would anticipate to be resolved in future releases.  For
example, I found nesting folders and files within the `src/lib` folder to
varying degrees produce transpilation errors or timing issues with the loading
of my CSS.  The most recent one I encountered was challenges building
production assets using their provided `build` script.  I hope to find
opportunities to contribute back in this space even though I'm new to this
solution, but I was able to have a lot of succes with it considering it is
beta.

As for the D3 portion of the project, I referenced examples on
[observable](link here) and documentation on [D3's](github), then retrofitted
the examples to be more Svelte-esque.  D3 produces great results, but the
guidance and documentation for constructing these visualizations seems rather
scattered and challenging to navigate.  This required a lot of trial and error
while fumbling around the documentation to piece together the various APIs.
Similarly, there are not many examples of D3 in Svelte at the moment, but I did
come across an interesting posting which pushed me in a good direction to
employ Svelte's expression language in accordance with D3 rather than utilizing
DOM manipulation via JavaScript.  The write up on this can be [here](link).

## Concluding Thoughts

Building this project allowed me the opportunity to explore some less familiar
areas and problems I less frequently encounter due to the selection of C.  This
provided me practice at building data structures like hash maps, queues, linked
lists, and graphs, while trying to optimize them for my use case.  Unless I am
really squeezing for performance or I am technically restricted in some
fashion, then I think I am rather unlikely to select C for many of my personal
projects going forward.  It gives you a lot of control, but the overhead of
building your own data structures and memory management becomes quite
cumbersome quickly.

As for SvelteKit, I think it has the opportunity to present a lot of value due
to it's low barrier of entry, but it needs more time to grow out of beta.  If I
were to build the same user experience with something like React, I could
likely build something more production-ready, but it would require more code.
For simple web applications, SvelteKit offers a lot of promise once it works
out the kinks.

You can find Spidey DB at:
[https://github.com/kubejm/spidey-db](https://github.com/kubejm/spidey-db)

You can find the Spidey DB demo at:
[https://github.com/kubejm/spidey-db-demo](https://github.com/kubejm/spidey-db-demo)
