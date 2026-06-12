---
title: Lab roster (machine access)
parent: Sysadmin
nav_order: 4
---

# Lab roster — who has accounts on which machines

This page is the canonical reference for which Imperial usernames should have accounts on each shared CEMRG workstation. The actual source of truth is `/etc/ic-lab-setup.conf` on every box (which is read by the [setup kit](lab-machine-setup.html)) — this page is the central index that keeps the per-box configs in sync.

{: .warning }
> **This page is public.** Imperial usernames are CID-like and already discoverable via Imperial's directory, so listing them here is acceptable. But do **not** add anything more sensitive: no full names beyond what's already on the directory, no email addresses, no project assignments, no supervisor info, no machine IPs, no admin contact emails. If you need to track those, use a private channel.

## How to read the tables

- `username` — Imperial-style short username (CID or similar). This is what the user types as their Linux account name.
- ✅ in a machine column means the user should have an account on that box.
- ⚙️ means they have sudo (admin) rights on that box. Sudo implies an account.
- Empty cell means no account.

## Current roster

{: .note }
> Last updated: **2026-06-12**. If a discrepancy with a real machine is found, the per-machine `/etc/ic-lab-setup.conf` wins — update this page to match, not the other way around.

| username    | ic-ada | ic-croderog | ic-abd | Notes |
|-------------|:------:|:-----------:|:------:|-------|
| ak5122      |   ✅   |             |        |       |
| alee9       |   ✅   |             |        |       |
| alewalle    |   ✅   |             |        |       |
| all224      |   ⚙️   |     ⚙️       |   ⚙️   | Lab admin |
| aqayyum     |   ✅   |     ✅      |        |       |
| beizhou     |   ⚙️   |             |        | Pre-existing admin account on ic-ada |
| bzhou6      |   ✅   |             |        |       |
| ccorrad1    |   ✅   |             |        |       |
| croderog    |   ⚙️   |     ⚙️       |   ⚙️   | Lab admin |
| dugurlu     |   ✅   |             |        | Added 2026-06-12 |
| ea2825      |   ⚙️   |     ⚙️       |   ⚙️   | Lab admin |
| jsolisle    |   ✅   |             |        |       |
| lcicci      |   ✅   |             |        |       |
| mb5613      |   ✅   |             |        |       |
| rkb23       |   ✅   |             |        |       |
| sm5223      |   ✅   |             |        |       |
| sq15        |   ✅   |             |        |       |
| sw4924      |   ✅   |             |        |       |
| tmb119      |   ✅   |             |        |       |
| wz422       |   ✅   |             |        |       |

ic-croderog and ic-abd columns above are placeholders — fill in once each machine's `/etc/ic-lab-setup.conf` has been audited. Until then, treat empty cells in those columns as "unknown, not no".

## How to update this page when adding or removing a lab member

1. On each affected machine, edit `/etc/ic-lab-setup.conf` and add or remove the line in `LAB_USERS=(...)`.
2. Run `sudo bash scripts/10-users.sh` from the kit. (Idempotent — for an addition, it creates the account and prints the initial password; for a removal, **note that `10-users.sh` does not delete accounts** by design; manually `sudo deluser --remove-home <user>` first if you want their data gone, or `sudo deluser <user> lab` to just revoke shared-group access.)
3. Edit this page (`docs/sysadmin/lab-roster.md` in `cemrg-lab-guides`) to reflect the new state. Bump the "Last updated" date.
4. Commit with a message like `roster: add dugurlu on ic-ada` and push.

## How to bootstrap this table for a new machine

When you set up a new box (see [lab machine setup](lab-machine-setup.html)), you'll pick `LAB_USERS` from this table. The common patterns are:

- **Everyone in the lab** → copy the full list, marking admins.
- **Just admins + a project subset** → copy your three admins and whichever lab members are working on that project.
- **Personal workstation** → only that person + the three admins.

After the kit runs, add a new column to the table above for the new machine and fill in the ✅ / ⚙️ marks.

## See also

- [Lab machine setup (end-to-end)](lab-machine-setup.html) — how `LAB_USERS` ends up provisioning accounts.
- [Create a user](create-users.html) — manual reference for adding a single user.
