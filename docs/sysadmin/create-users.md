---
title: Create a user
parent: Sysadmin
nav_order: 1
---

# Create a new user on a shared lab machine

Create the account. Arthur recommends `adduser` over `useradd` because it doesn't interfere with the GUI:

```shell
sudo adduser <new_username>
# or, lower-level:
sudo useradd <new_username>
```

> The user may already exist without a home folder, in which case they also can't SSH in.

Create the home folder if missing:

```shell
sudo mkhomedir_helper <new_username>
ls /home/<new_username>   # confirm
```

Add to the SSH groups so they can log in remotely:

```shell
sudo usermod -a -G ssh  <new_username>
sudo usermod -a -G sshd <new_username>
```

Grant superuser rights (only when justified):

```shell
sudo usermod -a -G sudo <new_username>
```
