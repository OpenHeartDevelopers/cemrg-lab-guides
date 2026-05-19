---
title: OPKSSH login
parent: Imperial
nav_order: 1
---

# Log in to an Imperial computer via SSH (OPKSSH)

Imperial uses [OPKSSH](https://github.com/openpubkey/opkssh) for SSO-backed SSH access.

## One-time setup

Install the binary for your platform:

```shell
# Linux
curl -L https://github.com/openpubkey/opkssh/releases/latest/download/opkssh-linux-amd64 -o opkssh
chmod +x opkssh

# macOS
curl -L https://github.com/openpubkey/opkssh/releases/latest/download/opkssh-osx-amd64 -o opkssh
chmod +x opkssh
```

Initialise the local config:

```shell
./opkssh login --create-config
```

Drop Imperial's config into the file that was just created:

```shell
curl https://rhn.cc.ic.ac.uk/install/scripts/standalone/sshgw/opkssh/config.yml -o ~/.opk/config.yml
```

## Every time you want to log in

> The certificate is valid for 24 hours, so this is roughly a daily ritual.

```shell
opkssh login
```

Follow the browser prompts. This drops a fresh keypair at `~/.ssh/id_ecdsa`. Then:

```shell
ssh <username>@opkssh.ic.ac.uk -i ~/.ssh/id_ecdsa
```
