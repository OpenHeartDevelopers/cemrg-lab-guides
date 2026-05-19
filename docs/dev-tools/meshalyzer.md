---
title: Meshalyzer
parent: Development tools
nav_order: 6
---

# Meshalyzer

```shell
sudo apt install libfuse-dev

mv ~/Downloads/Meshalyzer-5.4-x86_64.AppImage ~/installs/
cd ~/installs
mkdir Meshalyzer
mv Meshalyzer-5.4-x86_64.AppImage Meshalyzer/
cd Meshalyzer

chmod +x Meshalyzer-5.4-x86_64.AppImage
sudo ln -s "$(pwd)/Meshalyzer-5.4-x86_64.AppImage" /usr/local/bin/meshalyzer
```
