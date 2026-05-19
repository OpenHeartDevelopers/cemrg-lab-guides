---
title: Web installs
parent: New machine setup
nav_order: 1
---

# Things to install from the web

## 1. Sign into GitHub

- Sign in, then make an [SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
- [Add it to GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).


## 3. Conda + Poetry

```shell
mkdir -p ~/installs
cd ~/installs
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh   # follow prompts

# Sometimes you also need to accept the channel ToS:
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

## 4. Dropbox

Install from the [official Linux page](https://www.dropbox.com/en_GB/install-linux). If the machine has a separate data disk, install Dropbox into `/data`.


## Optionals 

### O1. Starship prompt

- Install a [NerdFont](https://www.nerdfonts.com/font-downloads) such as FiraCode; move the files into `~/.fonts/`.
- Follow the install instructions at [starship.rs](https://starship.rs).

### O2. Brave browser

Download from [brave.com](https://brave.com) and set it as the default browser. This one is based on Chromium but without a lot of the bloat. 

### O3. Kitty terminal

Terminal with tabs and fast. 

```shell
# Install from https://sw.kovidgoyal.net/kitty/binary/
sudo ln -s ~/.local/kitty.app/bin/kitty  /usr/local/bin/kitty
sudo ln -s ~/.local/kitty.app/bin/kitten /usr/local/bin/kitten
```

### O4. 1Password

Download the `.deb` from [1password.com](https://1password.com/downloads/linux), then:

```shell
sudo apt install ./1password-latest.deb
```
