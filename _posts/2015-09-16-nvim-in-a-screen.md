---
layout: post
title: NVIM in a Screen, and Escaping Immediately
tags: nvim screen config
eye_catch: /assets/img/logo-neovim.svg
---

As previously mentioned, [Neovim](http://neovim.io) is a great Vim clone that works well as an IDE [when you have it configured]({% post_url 2015-07-29-nvim-config %}).

Recently, I've been using it inside `screen`, a great utility for multi-tasking remotely.  Unfortunately, whenever pressing escape to leave insert vim's mode, there is a sizeable 300ms delay during which pressing another key (such as `:` to enter the `w`rite command) causes a rubbish character to be inserted.

<!--more-->

This can be fixed through use of a suitable `.screenrc` file, such as the one below:

<div class="code-name" title=".screenrc"></div>
```bash
# Don't wait for further input when escape is pressed
maptimeout 1

# Use the 256 color terminal
term xterm-256color
```

