---
title: Commit message cheatsheet 
date: '2020-07-14T01:39:48Z'
description: Echoing a quick reference to terminal
---

I dig the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/) 
and the [Angular convention](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines) 
for git commit messages, but do not incorporate these guidlines as often as I would like within 
my own projects and defer to less informative messages.  To encourage myself to 
employ this convention more frequently I created a small script that prints a 
cheatsheet to terminal, so I can quickly reference notes highlighting the 
specification. Ultimately, I hope to maintain more meaningful git logs with my 
projects going forward.

Listed below is an example execution of ccc (Conventional Commit Cheatsheet).

```sh
➜ ccc

Conventional Commit Cheatsheet

STRUCTURE
  <type>[optional scope]: <description>

  [optional body]

  [optional footer(s)]

TYPES
  build, chore, ci, docs, feat, fix, perf, refactor, style, test

EXAMPLES
  - Commit message with no body
    docs: add install instructions to README

  - Commit message with scope
    feat(search): add filter capability

  - Commit message with both '!' and breaking change footer
    refactor!: drop support for Node 11 and lower

    BREAKING CHANGE: using features introduced in Node 12
```

You can find ccc at: [https://github.com/kubejm/ccc](https://github.com/kubejm/ccc)
