---
title: Dotfiles (myconfig)
parent: New machine setup
nav_order: 5
---

# Dotfiles — `myconfig`

```shell
mkdir -p ~/utils
git clone git@github.com:alonsoJASL/myconfig.git ~/utils/myconfig
cd ~/utils/myconfig
```

Wire up each piece:

- **rofi**: `cp -R rofi ~/.config/`
- **Starship**: `cp starship.toml ~/.config/`
- **Kitty**: copy contents of `kitty/` into `~/.config/kitty/`
- **Conky**: copy contents of `conky/` into `~/.config/conky/`, then set `~/.config/conky/start_conky.sh` as a startup application.
- **bashrc**: cherry-pick what you need from `.bashrc` into your own.
- **Colorscript**: apply the patch (see below).

## Colorscript and wallpapers (optional)

```shell
mkdir -p ~/.installs
cd ~/.installs
git clone https://gitlab.com/dwt1/shell-color-scripts.git
git clone https://gitlab.com/dwt1/wallpapers.git
```

Symlink `colorscript` and have it run on shell start:

```shell
sudo ln -s ~/.installs/shell-color-scripts/colorscript.sh /usr/local/bin/colorscript
echo 'colorscript -r' >> ~/.bashrc
```

## Keyboard shortcuts

Set these in your window manager:

| Shortcut                                    | Action                              |
| ------------------------------------------- | ----------------------------------- |
| <kbd>SUPER</kbd>+<kbd>R</kbd>               | `rofi -show run -normal-window`     |
| <kbd>SUPER</kbd>+<kbd>W</kbd>               | `rofi -show window -normal-window`  |
| <kbd>SUPER</kbd>+<kbd>Return</kbd>          | Open `kitty`                        |
| <kbd>SUPER</kbd>+<kbd>Q</kbd>               | Close the focused window            |
| <kbd>SUPER</kbd>+<kbd>F</kbd>               | Open `pcmanfm`                      |

## Optional: remove the Ubuntu dock

```shell
sudo apt remove gnome-shell-extension-ubuntu-dock
```
