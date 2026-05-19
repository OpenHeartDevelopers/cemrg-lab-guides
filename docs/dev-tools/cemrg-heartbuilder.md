---
title: cemrg-heartbuilder
parent: Development tools
nav_order: 1
---

# cemrg-heartbuilder

```shell
cd ~/dev/python/
git clone https://github.com/OpenHeartDevelopers/cemrg-heartbuilder.git
cd cemrg-heartbuilder/
```
## Install environment 

**Option 1: Conda**
```shell
conda create -n cemrg-heartbuilder python=3.10 -y
conda activate cemrg-heartbuilder
```

**Option 2: Poetry** 
```shell
poetry config virtualenvs.create false --local
poetry install
```

**Install with script provided**
```shell
./setup.sh
```
