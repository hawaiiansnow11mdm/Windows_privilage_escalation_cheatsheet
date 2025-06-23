# Windows Privilege Escalation Cheatsheet

This is a comprehensive cheatsheet for privilege escalation techniques on Windows, detailing common attack vectors and exploitation methods.

## 0. INITIAL ENUMERATION - ESSENTIAL ⭐⭐⭐⭐⭐
────────────────────────────────────────────
- `whoami`
- `hostname`
- `systeminfo`
- `whoami /priv`
- `whoami /groups`
- `net user`
- `net localgroup administrators`
- `tasklist`
- `netstat -ano`
- `wmic logicaldisk get caption,description,providername`
- `dir /s /b c:\*.txt`
- `icacls C:\path`

## 1️⃣ WEAK SERVICE PERMISSIONS ⭐⭐⭐⭐⭐
────────────────────────────────────────────
To find services with weak permissions:
```bash
sc qc <service>
