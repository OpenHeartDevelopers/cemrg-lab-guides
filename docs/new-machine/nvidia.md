---
title: Nvidia driver fix
parent: New machine setup
nav_order: 6
---

# Replace nouveau with the Nvidia driver

If a fresh install boots into the `nouveau` driver, swap it for the proprietary Nvidia one.

```shell
ubuntu-drivers devices

# Install the recommended driver:
sudo ubuntu-drivers autoinstall

# OR, pin a specific version from the previous output, e.g.:
# sudo apt install nvidia-driver-535

sudo reboot
```

After reboot, verify:

```shell
nvidia-smi
lsmod | grep nouveau   # should print nothing
```
