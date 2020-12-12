---
title: Introducing auto-nix-shell for consistent development environments
published: true
image: https://i.imgur.com/yMcjOog.png
---

![](https://i.imgur.com/SREw8gz.png)

I created [auto-nix-shell](https://github.com/chrismwendt/auto-nix-shell) to automatically enter/exit `nix-shell` when `cd`ing in or out of a directory that has a Nix config file (`default.nix`). It even checks for config changes before executing the next shell command (using [`fish_preexec`](https://github.com/fish-shell/fish-shell/pull/1666) and [bash-preexec](https://github.com/rcaloras/bash-preexec)) and reloads the configuration when you `git checkout <some other commit>`
