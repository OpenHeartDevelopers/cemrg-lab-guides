---
title: apt installs
parent: New machine setup
nav_order: 2
---

# Install from `apt`

```shell
sudo apt install build-essential imagemagick pcmanfm rofi conky tldr neovim texlive libreoffice -y
tldr -u   # refresh the tldr database
```

| Name             | apt id            | Description              |
| ---------------- | ----------------- | ------------------------ |
| Build Essentials | `build-essential` | C, C++ toolchain         |
| ImageMagick      | `imagemagick`     | Image analysis utilities |
| pcmanfm          | `pcmanfm`         | File manager             |
| rofi             | `rofi`            | Run/window prompt        |
| conky            | `conky`           | Desktop overlays         |
| tldr             | `tldr`            | Friendlier `man` pages   |
| NeoVim           | `neovim`          | Text editor              |
| TeX Live         | `texlive`         | LaTeX libraries          |
| LibreOffice      | `libreoffice`     | Office suite             |

## Sometimes also needed

```shell
sudo apt install qtwayland5 -y
```
