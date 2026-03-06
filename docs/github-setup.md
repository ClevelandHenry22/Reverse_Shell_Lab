## GitHub Setup — Quick Guide

1. Create the Repository

Go to github.com → New repository

Name: reverse-shell-lab

Description: Hands-on cybersecurity home lab: Reverse shells, Metasploit & Wireshark traffic analysis | Kali Linux + Metasploitable 2

Public  | Add no default files (we have our own)

Click Create repository

2. Push Your Files (run on Kali)
   
```
cd ~/reverse-shell-lab
git init
git add .
git commit -m " Initial commit: Complete reverse shell lab

- Manual Netcat reverse shell
- Metasploit VSFTPd CVE-2011-2523 exploitation  
- Meterpreter post-exploitation
- Wireshark traffic analysis
- Full penetration test report"

git remote add origin https://github.com/YOUR_USERNAME/reverse-shell-lab.git
git branch -M main
git push -u origin main
```
