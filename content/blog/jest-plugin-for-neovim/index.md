---
title: Jest plugin for Neovim
date: '2020-10-03T14:52:19Z'
description: Extensibility with Lua
---

During the recent VimConf, a core contributor of Neovim presented on Neovim's
built in language server protocol (LSP).  The presentation can be found on
youtube with the title:
"[Vimconf.live: Neovim Builtin LSP](https://www.youtube.com/watch?v=C9X5VF9ASac)".
The demonstration and customization available to this feature seemed superior to
[coc.nvim](https://github.com/neoclide/coc.nvim), which I use today.  So, I
decided to give it a test drive.

After working with it on projects of various languages for over a week, the
Neovim LSP felt more performant than coc.nvim.  It seemed smoother and less
jarring overall.  Additionally, I found the ease of customization with
preexisting plugins, such as
[completion-nvim](https://github.com/nvim-lua/completion-nvim) and
[diagnostic-nvim](https://github.com/nvim-lua/diagnostic-nvim), provided a
cleaner and more appealing programming experience.

However, I encountered a small pitfall in my transition from coc.nvim to
Neovim's LSP.  A jest extension called
[coc-jest](https://github.com/neoclide/coc-jest) no longer worked for me due to
the dependency on coc.nvim.  This plugin allows you to run Jest within your
editor, which helps streamline development and provide quick feedback.
I spend the majority of my time programming with JavaScript and TypeScript with
frequent use of Jest, so the absence of this functionality became quite the
annoyance.  As a result, I decided to try my hand at constructing a similiar
plugin without the dependency on coc.nvim, so that this functionality could
standalone.

Without previous experience writing a plugin for Neovim, I sought guidance.
Neovim's documentation had me covered and provided an example, which serves as
a template for getting started.  The documentation can be found at
[Nvim documentation: lua](https://neovim.io/doc/user/lua.html) under the
subsection entitled "LUA PLUGIN EXAMPLE".  Alternatively, in Neovim you can
type `:help lua-require-example` to view the documentation.  With the Neovim
documentation in hand as well as the
[Lua Reference Manual](https://www.lua.org/manual/5.4/), I was off!  Once, I
uncovered the appropriate APIs for executing nvim commands and invoking the
terminal, then the plugin came together rather quickly.  The only issue that
caught me by surprise was Lua's unique way of pattern matching, which differed
from regular expressions and felt somewhat limiting.  However, I reached a
solution, which got the job done.

In short, the plugin exposes commands for invoking Lua functions that
orchestrate Nvim in a manner that runs Jest on the entire project, the file in
the current buffer, or even the test name under the user's cursor.  The Jest
execution outputs within a split buffer.

You can find jest.nvim at: [https://github.com/kubejm/jest.nvim](https://github.com/kubejm/jest.nvim)
