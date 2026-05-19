---
title: cemrg-heartbuilder
parent: Development tools
nav_order: 1
---

# cemrg-heartbuilder

```shell
cd ~/dev/python/
git clone git@github.com:OpenHeartDevelopers/cemrg-heartbuilder.git
cd cemrg-heartbuilder/

conda create -n cemrg-heartbuilder python=3.10 -y
conda activate cemrg-heartbuilder

poetry config virtualenvs.create false --local
poetry install

./setup.sh
```
