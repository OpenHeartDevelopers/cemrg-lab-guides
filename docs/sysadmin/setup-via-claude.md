---
title: Provision a new machine with Claude Code
parent: Sysadmin
nav_order: 3
---

# Provision a new machine with Claude Code

The end-to-end [lab machine setup runbook](lab-machine-setup.html) is written to be followed by a human — but Claude Code can drive it for you. This page is the recipe: install Claude on the new box, point it at the runbook, hand off.

The result is the same: a fully-provisioned lab workstation. The difference is you don't manually type `sudo bash scripts/00-base.sh`, `sudo bash scripts/10-users.sh`, … — Claude does, pausing at the decisions that need a human (config values, password capture, reboots).

{: .note }
> Claude Code costs API tokens. A full provisioning session uses ~$1–3 of usage on Sonnet/Opus depending on how many back-and-forths happen. Sensible budget for a one-time machine install.

## Prerequisites

- A fresh Ubuntu 24.04 install (Server or Desktop).
- A user account with `sudo` (the installer creates one during Ubuntu setup).
- Internet from the box (apt + curl + git all need to work).
- An Anthropic account at [claude.ai](https://claude.ai) — sign in once before starting if you haven't already.

## 1. Install Claude Code

```shell
curl -fsSL https://claude.ai/install.sh | bash
exec $SHELL    # reload PATH so `claude` is found
```

The installer drops the `claude` binary into `~/.local/bin/`, adds it to your shell's PATH, and installs a few helper directories under `~/.claude/`. No sudo, no Node.js to manage.

## 2. Authenticate

```shell
claude
```

On first launch it prints a device code and a URL like `https://claude.ai/device`. Open the URL in your laptop's browser, paste the code, approve. You're in.

Exit with `Ctrl-D` or `/exit`.

## 3. Open Claude in a sensible working directory

Claude reads and writes files relative to the directory it was launched from. Pick a parent for the kit:

```shell
mkdir -p ~/setup && cd ~/setup
claude
```

## 4. Paste the bootstrap prompt

Copy this verbatim into Claude's prompt. Fill in nothing — Claude will ask you for the four config values it needs.

```text
You are helping me set up a new shared lab workstation for the
CEMRG group at Imperial College London. The complete runbook is
published at:

  https://openheartdevelopers.github.io/cemrg-lab-guides/docs/sysadmin/lab-machine-setup.html

Please fetch that page with WebFetch and follow it end-to-end.

Workflow I want from you:
  1. Read the runbook in full so you understand all phases.
  2. Briefly summarise the prerequisites and check they're satisfied
     on this box (Ubuntu 24.04 confirmed, NVMe topology, GPU detection).
  3. Ask me for per-machine config values you need:
       - HOSTNAME
       - LAB_USERS (username:admin_flag list)
       - ENABLE_STORAGE_POOL and the device list if yes
       - ENABLE_GPU and which BLOCKED_GPU_UUIDS if any
  4. Pull the script kit: git clone https://github.com/all224/ic-lab-setup.git
     (private — you'll need to be added as a collaborator, or you can
     copy the tarball from /data/shared/ic-lab-setup.tar.gz on any
     existing lab box).
  5. Write /etc/ic-lab-setup.conf with the values I gave you.
  6. Run the phases in order: 00 → 10 → 20 → 40 → 30 → 50 → 99.
     After each script: summarise what happened in one paragraph
     and confirm before moving on.
  7. Run 99-verify.sh at the end and report the result.

Hard rules:
  - STOP and ask me before any reboot.
  - STOP and ask before any apt remove/install not specified by the runbook.
  - Do NOT push to any git repo without explicit confirmation.
  - Capture and show me the INIT_PASSWORD output of 10-users.sh so
    I can DM the passwords to lab members privately.

Tone: concise. Don't narrate options I won't pursue. Don't re-explain
runbook content I just read.
```

## What Claude does, what you do

| Phase | Claude | You |
|---|---|---|
| Read runbook | Fetches the page, summarises | Confirm it looks like the right doc |
| Prereq check | Runs `lsb_release -a`, `lsblk`, `lspci \| grep NVIDIA` | Skim the output |
| Config values | Asks the four config questions | Provide hostname, user list, storage choice, GPU UUIDs to block |
| Clone kit | `git clone` or scp the tarball | Help with auth if the repo is private and Claude isn't a collaborator |
| Write config | Creates `/etc/ic-lab-setup.conf` | Review for typos |
| Run 00-base | Hostname, SSH, xrdp, ufw, fail2ban | Confirm before moving on |
| Run 10-users | Creates accounts, prints init passwords | **Capture the password table — DM each user privately** |
| Run 20-storage | Wipes target NVMes (with confirmation), builds btrfs pool | Confirm devices before wipe |
| Run 40-nvidia | Installs `-open` driver + kmod | None until reboot question |
| Run 30-containers | Podman + CDI | None |
| Run 50-gpu-tracking | Collector + reporter | None |
| Run 99-verify | Green-tick report | Read result |
| Reboot | **Asks first** | Approve, then reconnect |

## When the kit asks for a reboot

Claude will pause and ask. After you approve and the box comes back, reconnect via SSH and re-launch `claude` from the same directory — Claude Code preserves session state in `~/.claude/`, so you can pick up where you left off:

```shell
ssh you@<new-machine>
cd ~/setup
claude --continue        # resumes the last session
```

## Tips

- **Pre-generate SSH pubkey** on your laptop so you can paste it into the new box's `~/.ssh/authorized_keys` once your account is created. Then you don't have to type the initial password every time you reconnect.
- **Have the GPU UUIDs ready** before running the kit if any cards are flaky. After the driver phase finishes you can run `nvidia-smi --query-gpu=uuid --format=csv` to read them off.
- **Be in a `tmux` or `screen` session** during the install. If your SSH drops mid-install, you reconnect to the same tmux pane and Claude is still there.

## Troubleshooting

### Claude can't WebFetch the runbook

Public Pages can occasionally rate-limit. Either retry, or paste the runbook content directly into the conversation:

```shell
curl -sL https://openheartdevelopers.github.io/cemrg-lab-guides/docs/sysadmin/lab-machine-setup.html | \
  pandoc -f html -t markdown
```

Then paste the markdown.

### Claude wants to run a script we haven't reviewed

Say no. The bootstrap prompt's "Hard rules" tell Claude not to apt-install or push without confirmation, but it can still propose things outside the runbook. Treat anything not in the [lab machine setup](lab-machine-setup.html) script list as a red flag, ask why, then decide.

### Claude says the repo is private and it can't clone

The kit is currently at `all224/ic-lab-setup`, private. Two ways to give Claude access:

1. **Tarball fallback** (works from any existing lab box):

   ```shell
   scp you@ic-ada:/data/shared/ic-lab-setup.tar.gz ~/setup/
   cd ~/setup && tar xzf ic-lab-setup.tar.gz
   ```

   Tell Claude `the kit is unpacked at ~/setup/ic-lab-setup; skip the clone step`.

2. **GitHub auth in Claude's container**: install `gh`, run `gh auth login` in a separate shell, then Claude inherits the credentials when it runs `git clone`. See [GitHub CLI on Ubuntu](https://github.com/cli/cli#linux--bsd) for install.

## See also

- [Lab machine setup (end-to-end)](lab-machine-setup.html) — the runbook itself
- [Create a user](create-users.html) — manual user-creation reference for individual additions later
