---
title: Stowing configs
date: "2020-04-10T01:10:03Z"
description: Simply maintaining personal configuration
---

I have been managing my dotfiles and configuration for years within git, but 
only recently encountered [stow](https://www.gnu.org/software/stow/).  It has 
greatly simplified managing my configuration and put any previous headaches to
rest.

Stow serves as a symlink farm manager.  It allows you to store your files in a 
separate directory, but makes them appear as though they exist in the location 
your system expects them.

For example, my system expects my tmux configuration to exist in 
`~/.tmux.conf`, but it would be undesirable to use my home directory as a git 
repository.  With stow, I can place my tmux configuration in 
`~/configs/tmux/.tmux.conf` and establish a symlink to `~/.tmux.conf` by 
changing directories to the configs folder and executing stow on the tmux 
folder by running `cd ~/configs && stow tmux`.  Stow simply mirrors and 
symlinks the contents of the specified package as the first argument.

My configs directory looks as follows:

```bash
➜ tree -L 4 -a -I .git ~/configs
/home/matt/configs
├── compton
│   └── .config
│       └── compton.conf
├── fish
│   └── .config
│       └── fish
│           └── config.fish
├── i3
│   └── .config
│       └── i3
│           └── config
├── kitty
│   └── .config
│       └── kitty
│           └── kitty.conf
├── nvim
│   └── .config
│       └── nvim
│           ├── coc-settings.json
│           └── init.vim
├── polybar
│   └── .config
│       └── polybar
│           └── config
├── tmux
│   └── .tmux.conf
└── zathura
    └── .config
        └── zathura
            └── zathurarc

21 directories, 9 files
```

I would simply go into the `~/configs` directory and execute stow as follows:

```bash
➜ cd ~/configs
➜ stow compton
➜ stow fish
➜ stow i3
➜ stow kitty
➜ stow nvim
➜ stow polybar
➜ stow tmux
➜ stow zathura
```

Removing a symlink is just as easy.  I could remove the nvim symlink as follows:

```bash
➜ cd ~/configs
➜ stow -D nvim
```

If you are wondering even why you would want to source control your personal 
configuration, then here are a few reasons to consider:

* Fearlessly test-drive settings with the ability to revert back
* Retain a history of changes
* Provide portability of your configuration across machines

You can find my personal configurations at: 
[https://github.com/kubejm/configs](https://github.com/kubejm/configs)
