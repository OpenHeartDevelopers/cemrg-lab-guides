---
title: Rust / cargo utils
parent: New machine setup
nav_order: 3
---

# Rust / cargo utilities

Faster, friendlier replacements for some core utilities.

## Install Rust

Follow [rustup.rs](https://rustup.rs). After install, source cargo:

```shell
. "$HOME/.cargo/env"
```

## Install the utilities

```shell
cargo install exa fd-find ripgrep bat
```

| Name    | Replaces | Description                       |
| ------- | -------- | --------------------------------- |
| exa     | `ls`     | Better listing, git-aware         |
| fd      | `find`   | Sensible defaults                 |
| ripgrep | `grep`   | Fast recursive search             |
| bat     | `cat`    | Syntax highlighting + paging      |

Add the following alias to your `.bashrc` to make `exa` more useful:

```shell
alias exa='exa -la --group-directories-first'
```
