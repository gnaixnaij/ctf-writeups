# [Challenge Name]

| Field | Value |
|-------|-------|
| **Category** | Web / PWN / Rev / Crypto / Forensics / OSINT |
| **Difficulty** | Easy / Medium / Hard / Insane |
| **Platform** | HTB / THM / PicoCTF / CTFtime |
| **Date** | YYYY-MM-DD |

## Summary

Brief description of the challenge and the vulnerability/technique used.

## Enumeration

What tools and techniques were used to discover the attack surface.

```
# Commands used
nmap -sC -sV target.com
gobuster dir -u http://target.com -w /usr/share/wordlists/common.txt
```

## Exploitation

Step-by-step breakdown of how the vulnerability was exploited.

```python
# Payload or exploit script
import requests
r = requests.get("http://target.com/flag")
print(r.text)
```

## Flag

```
flag{...}
```

## Key Takeaways

- What was learned
- Common pitfalls
- Related techniques
