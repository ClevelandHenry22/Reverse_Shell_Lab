# Reverse Shell Lab -- Offensive Security Home Lab

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

### Table of Contents

1. Project Overview
2. Objectives
3. Lab Structure
4. Tools and Technologies
5. Environment Setup

     - VirtualBox Installation
     - Kali Linux VM Setup
     - Metasploitable 2 VM setup
     - Network Configuration
  
6. Attack Phase 1 -- Manual Reverse Shell with Netcat
7. Attack Phase 2 -- Metasploit Exploitation
8. Traffic Analysis with Wireshark
9. Detection and Blue Team perspectives
10. Key Findings
11. Lessons Learned
12. Skills Demonstrated
13. Further Research and Next Steps

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

     ATTACKER VM                      TARGET VM

     Kali Linux
     IP: 192.168.56.101

     Tools:
      - Metasploit
      - Netcat
      - Wireshark
      - Nmap

     TARGET VM
     Metasploitable 2
     IP: 192.168.56.102

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

`ping 192.168.56.102` - *ping Metasploitable*

*If the ping responds, the lab is ready*

                                         
                                         
