# Node Hardening - Lab 9 (K-B302-23-06)
 
| | |
|---|---|
| Host | `K-B302-23-06` |
| User | `opilane` |
| OS | Ubuntu 24.04.4 LTS |
| Kernel | 6.6.87.2-microsoft-standard-WSL2 |
| Environment | WSL |
| Run on | 2026-10-23 09:32 EEST |
 
## Notes
- WSL has no router VM and no desktop environment. ProxyJump was demonstrated through localhost (`router` and `lan-b` both resolve to 127.0.0.1) and nmap scanned localhost.
- `find` used `-xdev` so it does not crawl the Windows drives under /mnt, a small deviation from the lab command.

## Submission checklist
- [x] captured `before.txt`
- [x] captured `after.txt`
- [x] recorded the diff
- [ ] documented each removal with justification
- [x] hardened SSH (drop-in `00-hardening.conf`)
- [x] validated `sshd -t`
- [x] reloaded SSH without restarting (sshd MainPID 5338 -> 5338)
- [x] disabled password authentication (effective `sshd -T` value)
- [x] configured `ProxyJump`
- [x] enabled unattended updates
- [x] proved automatic updates ran
- [x] verified port exposure and password refusal
 
## 9.A - Measure and reduce
 
| Metric | Before | After | Change |
|---|---|---|---|
| Listening TCP sockets (unique port/process) | 4 | 4 | 0 |
| Running services | 15 | 16 | 1 |
| Installed packages | 611 | 611 | 0 |
| SUID binaries | 14 | 14 | 0 |
 
Files: [before.txt](evidence/before.txt), [after.txt](evidence/after.txt), [diff.txt](evidence/diff.txt)
 
**Listening ports closed:**
- none
 
**Listening ports newly opened:**
- none
 
**Services no longer running:**
- none
 
0 packages were removed in total (targets plus dependencies cleaned up by autoremove); 0 SUID binaries disappeared.
 
### Justification table
 
| Change | What did this expose? | What would break without it? |
|---|---|---|
| (nothing removed) | No target packages were installed on this machine | n/a |

### Account audit
Full output: [accounts-audit.txt](evidence/accounts-audit.txt)
- Accounts (UID >= 1000, login shell) other than `opilane` needing review: none
- Accounts with an empty password field: none
- Accounts deleted by this run: none
 
## 9.B - SSH and patching
 
### sshd hardening
- Key login to localhost worked before changing anything: yes
- Drop-in: [00-hardening.conf](evidence/00-hardening.conf) (named `00-` so it is read first and wins over other drop-ins); main file [sshd_config](evidence/sshd_config)
- Effective values: [sshd-effective.txt](evidence/sshd-effective.txt)
- `sshd -t` passed: yes; applied with `systemctl reload ssh` (existing sessions stay alive)
- Password login attempt result ([password-refused.txt](evidence/password-refused.txt)): opilane@localhost: Permission denied (publickey).
 
### ProxyJump
Config: [ssh_config_demo](evidence/ssh_config_demo). Test output: [proxyjump-test.txt](evidence/proxyjump-test.txt) (`K-B302-23-06`)
 
### Unattended upgrades
[unattended-upgrades.txt](evidence/unattended-upgrades.txt) - enabled in `20auto-upgrades`: yes. Last run result: No packages found that can be upgraded unattended and no pending auto-removals
 
### Verification with nmap
[nmap.txt](evidence/nmap.txt) - open TCP ports: 22/tcp (ssh), 80/tcp (http)

 
## Reproduce
```bash
bash harden.sh        # BLOAT=1, STALE_USERS, KEEP_PW=1, TARGET, REPO options are documented in the script header
```
