---
title: ITK-SNAP and c3d
parent: Development tools
nav_order: 4
---

# ITK-SNAP (and c3d)

Download the tarball from [itksnap.org](http://www.itksnap.org/) and move it into `~/installs`:

```shell
mv ~/Downloads/itksnap-4.2.2-20241202-Linux-x86_64.tar.gz ~/installs/
cd ~/installs
tar xvf itksnap-4.2.2-20241202-Linux-x86_64.tar.gz
```

Add aliases to `.bashrc` pointing to the `itksnap` and `c3d` binaries inside the extracted folder. A symlink does **not** work here — ITK-SNAP resolves resources relative to the binary location, so use `alias`.
