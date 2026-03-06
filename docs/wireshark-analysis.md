# Wireshark Traffic Analysis

## Setup

Start Wireshark on Kali before running any attacks:

`sudo wireshark `

# Select interface: eth1 (host-only adapter) → Start capture


---

## Key Filters

| Filter | Purpose |
|--------|---------|
| `tcp.port == 4444` | Netcat reverse shell traffic |
| `tcp.port == 21 \|\| tcp.port == 6200` | VSFTPd backdoor traffic |
| `tcp.port == 4433` | Meterpreter session traffic |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | All new connection attempts |
| `ip.src == 192.168.56.5` | Traffic from target only |
| `frame contains "bash"` | Packets with shell strings |

---

## Analysis 1 — Netcat Reverse Shell (Port 4444)

Apply filter → right-click any **PSH, ACK** packet → **Follow → TCP Stream**

**What you'll see (two colors):**
- Red = commands typed on Kali (attacker)
- Blue = responses from Metasploitable (target)

```
bash: no job control in this shell
msfadmin@metasploitable:~$ whoami
whoami
msfadmin

```

**Key findings:**
- Target initiated the connection (reverse shell signature)
- All data completely unencrypted — every command visible
- `/etc/passwd` contents fully readable in the stream

---

## Analysis 2 — VSFTPd Backdoor (Ports 21 & 6200)

**Filter:** `tcp.port == 21 || tcp.port == 6200`

Follow the FTP stream — you'll see the `:)` backdoor trigger in the username field, followed by the root shell opening on port 6200.

---

## Analysis 3 — Meterpreter (Port 4433)

**Filter:** `tcp.port == 4433`

Follow TCP stream — unlike Netcat, you'll see encrypted binary data (unreadable). This shows why Meterpreter is harder to detect via content inspection, though the connection metadata is still visible.

---

## IOCs Captured

| IOC | Value | Confidence |
|-----|-------|------------|
| Non-standard outbound port | 4444, 6200, 4433 | High |
| Reverse connection pattern | Target initiates outbound TCP | High |
| Shell strings in traffic | `"bash: no job control"`, `"whoami"` | High |
| FTP backdoor trigger | USER containing `:)` | Critical |
| Shadow file in stream | `/etc/passwd` visible plaintext | Critical |
