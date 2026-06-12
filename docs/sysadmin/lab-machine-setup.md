---
title: Lab machine setup (end-to-end)
parent: Sysadmin
nav_order: 2
---

# Setting up a new lab workstation

This runbook gets a fresh Ubuntu 24.04 box to the same state as `ic-ada`: SSH and RDP exposed safely, all lab users provisioned with rootless container access, an optional multi-NVMe btrfs pool at `/data`, NVIDIA driver correctly installed for any consumer Blackwell GPUs, and a per-user GPU usage tracker. Companion to [Create a user](create-users.html) — that page is for adding one user to an already-set-up machine; this page is for everything else.

Companion script kit: [`all224/ic-lab-setup`](https://github.com/all224/ic-lab-setup) (private — currently under the lab admin's personal account; ask `all224` for access, or grab the tarball from `/data/shared/ic-lab-setup.tar.gz` on any lab box). Every step on this page is implemented by an idempotent script in that repo; re-running a script never destroys work already done.

{: .note }
> Read [Lessons learned](#lessons-learned) before deviating from the kit. Several of those points are non-obvious and we hit them the hard way on `ic-ada` in May 2026.

## Quickstart

1. Install **Ubuntu 24.04 Server or Desktop**. Give the OS its own NVMe (the `/` drive); leave other NVMes untouched — they become the data pool.

2. Pull the kit:

   ```shell
   sudo apt install -y git
   git clone https://github.com/all224/ic-lab-setup.git ~/ic-lab-setup
   # ...or fall back to the tarball on any lab machine:
   #   scp <admin>@ic-ada:/data/shared/ic-lab-setup.tar.gz . && tar xzf ic-lab-setup.tar.gz
   cd ~/ic-lab-setup
   ```

3. Create the per-machine config:

   ```shell
   sudo cp config/machine.conf.template /etc/ic-lab-setup.conf
   sudoedit /etc/ic-lab-setup.conf
   ```

   Set `HOSTNAME`, `LAB_USERS`, `ENABLE_GPU`, `ENABLE_STORAGE_POOL`, and the device list. Mark admins with `:1`.

4. Run the phases in order:

   ```shell
   sudo bash scripts/00-base.sh         # hostname, SSH, xrdp, ufw, fail2ban
   sudo bash scripts/10-users.sh        # lab group + accounts
   sudo bash scripts/20-storage.sh      # btrfs pool (skipped if disabled)
   sudo bash scripts/40-nvidia.sh       # NVIDIA -open driver + container toolkit
   sudo bash scripts/30-containers.sh   # podman + GPU CDI (AFTER 40)
   sudo bash scripts/50-gpu-tracking.sh # usage tracker
   sudo bash scripts/99-verify.sh       # green ticks across the board
   ```

5. Reboot once at the end. Re-run `99-verify.sh` to confirm everything came back.

6. Capture the initial passwords printed by `10-users.sh` and distribute them through a private channel (Imperial email or Teams DM, never a shared channel).

{: .warning }
> The init passwords printed by `10-users.sh` are shown **once**. If you don't capture them, the only recovery is `sudo passwd -e <user>` to force a reset.

## What this kit assumes

| Resource | Expectation |
|---|---|
| OS | Ubuntu 24.04 LTS, kernel 6.11+ (HWE) |
| Root drive | Its own dedicated NVMe/SSD |
| Data drives | Zero or more empty NVMe drives (no data on them) |
| GPUs | Zero or more NVIDIA cards. Blackwell (RTX 50-series) requires the **open** kernel module. |
| Network | Imperial College LAN reachable (172.22.0.0/16 typical) |
| Sudo | You can run scripts as root |

The kit does **not** handle the OS install itself, LDAP/AD integration, Imperial DNS registration, or BMC/IPMI configuration.

## Configuration reference

`/etc/ic-lab-setup.conf`:

```shell
HOSTNAME="ic-newbox"
LAB_GROUP="lab"
SHARED_DIR_PATH="/data/shared"

LAB_USERS=(
  "alice:0"      # regular member
  "bob:1"        # admin
)

ENABLE_STORAGE_POOL=true
STORAGE_DEVICES=( "/dev/nvme1n1" "/dev/nvme2n1" "/dev/nvme3n1" )
STORAGE_DATA_PROFILE="single"     # max capacity, no redundancy
STORAGE_META_PROFILE="raid1"

ENABLE_GPU=true
NVIDIA_DRIVER_PACKAGE="nvidia-driver-595-open"
NVIDIA_KMOD_METAPACKAGE="linux-modules-nvidia-595-open-generic-hwe-24.04"

BLOCKED_GPU_UUIDS=()              # GPU-... UUIDs to disable at boot

UFW_TCP_PORTS=(22 3389)
```

Re-run any script after editing — every change is picked up immediately.

## What each script does

### `00-base.sh` — base services

Sets the hostname, fixes `/etc/hosts`, installs and starts `openssh-server`, `xrdp`, `ufw`, `fail2ban`. Writes an SSH config drop-in that allows both password and key auth but disables root login. UFW is default-deny with `22/tcp` rate-limited and `3389/tcp` allowed.

### `10-users.sh` — accounts

Creates the `lab` POSIX group, then iterates `LAB_USERS`. For each user: `adduser --disabled-password` if missing, then generate a random 14-character alphanumeric initial password (if `passwd -S` shows no usable password) and `chage -d 0` to force change on first login. Adds to `lab`; admins additionally added to `sudo`.

Prints `USERNAME / STATUS / INIT_PASSWORD / ADMIN? / PRIOR_PW`. Re-running is safe — existing passwords are never overwritten.

### `20-storage.sh` — btrfs pool (optional)

Skipped if `ENABLE_STORAGE_POOL=false`. Otherwise:

- Refuses to wipe any device whose existing mount contains files other than `lost+found`.
- Creates one btrfs filesystem labelled `lab-data` spanning all listed devices.
- Mounts at `/data`, persists in `/etc/fstab` with `noatime,compress=zstd:3`.
- Creates `/data/shared` (group=`lab`, 2775 setgid), `/data/users/<u>` (0700 per user), and a `/srv/lab → /data/shared` symlink.

{: .note }
> Default profile is `data=single, metadata=raid1`. Data has **no redundancy** — a single-drive failure loses files on that drive (btrfs checksums tell you which). Use Imperial research storage for anything you can't regenerate.

### `30-containers.sh` — rootless container stack

- Flips `kernel.apparmor_restrict_unprivileged_userns=0` (Ubuntu 24.04 hardening blocks rootless containers otherwise).
- Installs `podman`, `podman-docker` (so `docker` works as an alias), `slirp4netns`, `uidmap`, `fuse-overlayfs`.
- Ensures every lab user has `/etc/subuid` / `/etc/subgid` ranges.
- If GPU enabled, regenerates `/etc/cdi/nvidia.yaml`.

{: .important }
> Run `40-nvidia.sh` **before** this script. CDI generation needs `nvidia-container-toolkit` which the driver phase installs.

### `40-nvidia.sh` — NVIDIA driver

- Refuses to install a proprietary driver if a Blackwell GPU is detected (PCI IDs `10de:2b8x`).
- Installs the configured driver and its kernel-module package for the running kernel.
- Explicitly installs the kernel-version-specific kmod (`linux-modules-nvidia-595-open-${KVER}`) as belt-and-braces — see [Lessons learned § 2](#2-apt-upgrade-does-not-pull-the-new-kernels-nvidia-kmod).
- Tries `modprobe nvidia` without rebooting.
- If `BLOCKED_GPU_UUIDS` is non-empty, hands off to `41-block-gpus.sh`.

### `41-block-gpus.sh` — block specific cards by UUID

Generates `/usr/local/bin/nvidia-block-faulty-gpus` and a systemd unit (`Before=nvidia-persistenced.service`) that, on every boot:

1. Locates each blocked UUID's current PCI BDF (these reshuffle across reboots; UUID is the only stable identifier).
2. Sets `driver_override=(none)` so nothing can re-bind it.
3. Unbinds it from the nvidia driver.
4. PCI-removes the device (`echo 1 > /sys/bus/pci/devices/<BDF>/remove`).
5. Regenerates the CDI spec.

{: .warning }
> The `Before=nvidia-persistenced.service` ordering is **critical**. If persistenced opens the device first, the unbind hangs in uninterruptible kernel sleep on "non-zero usage count" and you can't even `kill -9` the writer.

### `50-gpu-tracking.sh` — usage tracker

- `/usr/local/bin/gpu-collect` (Python) — runs every 30 s via systemd timer. Writes per-GPU samples, per-process samples (PID → username, container ID), and dmesg Xid/NVRM errors into SQLite at `/var/lib/gpu-usage/usage.db`.
- `/usr/local/bin/gpu-report` (Python) — any user can run.
- `gpu-collect.service` + `.timer` systemd units, enabled.

### `99-verify.sh` — sanity check

Reads the config and pokes the system to confirm hostname / services / firewall / users / storage mount / container stack / GPU stack match what config says they should be. Returns non-zero on any failure — useful between runs.

## Day-2 operations

### View GPU usage

```shell
gpu-report --live              # snapshot right now
gpu-report                     # last 24h: per-user, per-GPU, errors
gpu-report --hours 168         # last week
gpu-report --user alice
gpu-report --gpu 4
gpu-report --errors
```

Underlying data: `sqlite3 /var/lib/gpu-usage/usage.db` for ad-hoc queries.

### Add a new lab user

Append to `LAB_USERS` in `/etc/ic-lab-setup.conf`, then:

```shell
sudo bash scripts/10-users.sh
sudo bash scripts/30-containers.sh   # subuid/subgid for the new user
```

Capture the printed `INIT_PASSWORD` and share privately. For details on user creation when you're not using the kit, see [Create a user](create-users.html).

### Block (or unblock) a GPU

Edit `BLOCKED_GPU_UUIDS` in config, then:

```shell
sudo bash scripts/41-block-gpus.sh
```

To unblock: remove the UUID, re-run, reboot so the kernel re-enumerates.

### Tighten SSH (after admins add their pubkeys)

```shell
sudoedit /etc/ssh/sshd_config.d/10-ic-lab.conf
# set PasswordAuthentication no
sudo sshd -t && sudo systemctl reload ssh
```

## Lessons learned

These are the non-obvious things that bit us setting up `ic-ada`. Read these *before* deviating from the kit.

### 1. Blackwell consumer GPUs require the OPEN kernel module

The proprietary kernel module loads cleanly but refuses to initialise any Blackwell device (RTX 50-series, PCI dev IDs `10de:2b8x`). `nvidia-smi` reports "No devices were found" and dmesg shows:

```
NVRM: The NVIDIA GPU 0000:XX:00.0 (PCI ID: 10de:2b85)
NVRM: installed in this system requires use of the NVIDIA open kernel modules.
NVRM: GPU 0000:XX:00.0: RmInitAdapter failed! (0x22:0x56:897)
```

`40-nvidia.sh` refuses to install a non-`-open` driver if it detects a Blackwell device.

### 2. `apt upgrade` does NOT pull the new kernel's NVIDIA kmod

`linux-modules-nvidia-595-open-6.17.0-29-generic` is a **new package name**, not an upgrade of an existing one, when the kernel jumps `-19 → -29`. `apt upgrade` is conservative and won't install new packages, so you reboot into a new kernel with no NVIDIA module and `nvidia-smi` fails.

Fix: `apt full-upgrade` after kernel changes, or explicitly install the kernel-version-specific kmod. The driver script does the latter as belt-and-braces.

### 3. PCI BDFs reshuffle across reboots on bifurcated boards

On the GENOA2D24G-2L+ (and likely other boards that bifurcate `x16` slots), PCIe bridge enumeration order can vary between boots. We observed the same physical card move from `A3:00.0` → `A5:00.0` → `A3:00.0` → `A1:00.0` over four reboots.

**Never identify a GPU by its PCI BDF in persistent config**. Use the GPU UUID (stable hardware identifier).

### 4. The GPU block service must run `Before=nvidia-persistenced.service`

`nvidia-persistenced` opens `/dev/nvidiaN` for every visible GPU on startup. If you try to unbind or remove a device that persistenced has open, the kernel queues the operation and blocks indefinitely on "non-zero usage count". The writer sits in uninterruptible kernel sleep (D state) — `kill -9` won't free it.

Fix: order the blocker before persistenced. The systemd unit installed by `41-block-gpus.sh` has `Before=nvidia-persistenced.service` for this reason. Don't change it.

### 5. PCIe link width can drop dramatically at idle

GPUs aggressively park their PCIe link at low gen/width when idle. On a healthy Gen5 x16 link, `nvidia-smi --query-gpu=pcie.link.gen.current,pcie.link.width.current` at idle commonly reads `1, 4` (Gen 1 × 4). This is power saving, not failure. The link retrains on demand.

Verify under real CUDA load, not `nvidia-smi --query` polls — those don't push enough traffic to ramp the link.

### 6. Boards with bifurcated `x16` slots may limit GPUs to `x4`

Some boards split each physical `x16` slot into four electrically-`x4` lanes to fit more GPUs. This is a chassis design choice, not fixable in software. Confirm by checking the upstream PCIe bridge's `LnkCap`:

```shell
lspci -vvv -s <BDF-of-bridge> | grep LnkCap
```

If `LnkCap: Width x4`, you're stuck at `x4` regardless. Affects multi-GPU DDP gradient sync; doesn't affect single-GPU compute.

### 7. `podman-docker` removes `docker-ce`

Installing `podman-docker` (which provides `/usr/bin/docker` as a podman wrapper) conflicts with `docker-ce`. Apt resolves by removing `docker-ce`. **For a shared lab box this is usually correct** — every user gets rootless containers — but be aware. If you specifically need a Docker daemon for long-running services, install `podman` only (not `podman-docker`) and tell users to use `podman` directly.

### 8. `tr ... | head -c N` triggers SIGPIPE under `set -o pipefail`

Under `set -euo pipefail`, `tr ... | head -c 14` exits non-zero (141 from SIGPIPE) and aborts the script after the assignment. The kit uses `openssl rand | tr -dc | cut -c1-N` instead — no SIGPIPE risk.

### 9. `nvidia-ctk cdi generate` enumerates PCI directly, not just NVML

Unbinding a card from the nvidia driver makes it disappear from `nvidia-smi -L`, but `nvidia-ctk` still sees the PCI device and includes it in the CDI spec. To hide a card from both, you must PCI-remove it. `41-block-gpus.sh` does this on every boot since PCI remove is not persistent across reboots.

## Troubleshooting

### `nvidia-smi` says "Failed to initialize NVML"

Module isn't loaded (`lsmod | grep nvidia` is empty), or proprietary driver on Blackwell. Recovery:

```shell
sudo apt install -y nvidia-driver-595-open linux-modules-nvidia-595-open-${kernel}-generic
sudo modprobe nvidia
nvidia-smi -L
```

### `nvidia-smi` says "No devices were found"

Most likely: proprietary driver on Blackwell. See [Lessons learned § 1](#1-blackwell-consumer-gpus-require-the-open-kernel-module).

### `podman run` says "Could not create namespace"

Rootless userns setting got reset:

```shell
echo 'kernel.apparmor_restrict_unprivileged_userns=0' | \
  sudo tee /etc/sysctl.d/99-rootless-userns.conf
sudo sysctl --system
```

### Container can't see GPU

```shell
nvidia-ctk cdi list                                          # should show your GPUs
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml   # regenerate
```

### `gpu-report` shows empty data

The collector hasn't run yet. Check:

```shell
systemctl status gpu-collect.timer
sudo journalctl -u gpu-collect.service --since "10 min ago"
```

### Cannot SSH from off-campus

Imperial RFC1918 ranges (172.22.0.0/16 typical) require:

- Imperial network connectivity, or
- Zscaler Client Connector active **with the subnet in tunnel scope**, or
- An SSH jump via a box whose subnet *is* in Zscaler scope: `ssh -J user@ic-croderog user@172.22.X.X`

### Block script hung on "non-zero usage count"

The writer is in D state — `kill -9` won't free it. You can't stop persistenced cleanly because it has the device open. **Reboot is the cleanest reset** — the persistent systemd unit's ordering will avoid the hang next time.

## Files installed

| Path | Source | Purpose |
|---|---|---|
| `/etc/ic-lab-setup.conf` | per-machine | Config sourced by every script |
| `/etc/ssh/sshd_config.d/10-ic-lab.conf` | `00-base.sh` | SSH policy |
| `/etc/sysctl.d/99-rootless-userns.conf` | `30-containers.sh` | Rootless containers |
| `/etc/cdi/nvidia.yaml` | `30-containers.sh` / `41-block-gpus.sh` | Container GPU access |
| `/etc/skel/README-ic-lab.txt` | `10-users.sh` | First-login welcome |
| `/etc/systemd/system/gpu-collect.{service,timer}` | `50-gpu-tracking.sh` | Usage collector |
| `/etc/systemd/system/nvidia-block-faulty-gpus.service` | `41-block-gpus.sh` | GPU blocker |
| `/usr/local/bin/{gpu-collect,gpu-report}` | `50-gpu-tracking.sh` | Tracker tooling |
| `/usr/local/bin/nvidia-block-faulty-gpus` | `41-block-gpus.sh` | Blocker binary |
| `/var/lib/gpu-usage/usage.db` | runtime | SQLite usage history |
| `/var/log/ic-lab-setup/*.log` | runtime | Per-script logs |
| `/data` (mount), `/data/shared`, `/data/users/*` | `20-storage.sh` | Lab storage |
| `/srv/lab` (symlink) | `20-storage.sh` | Shared dir convenience path |
