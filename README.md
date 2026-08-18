# NSSECU2-Hacking-Tool-Creation

NetProbe is a lightweight network reconnaissance and scanning tool built to be a custom security tool. It is an automated NMAP network probing tool that is built on many of the fundamental NMAP network probing principles. NetProbe is a fully system library-based Python application, and does not require third-party modules and can be compiled to run directly as a portable binary (.exe) for cross-platform lab demonstrations. Its key functions are identifying remote operating systems, TCP port scanning and service version banner reading.

---

## Prerequisites

**System Requirements**
* **Interpreter:** Python 3.8 or higher installed (Linux / macOS / Windows).
* **Standard Libraries:** socket, subprocess, concurrent.futures, ipaddress, platform, re, sys (all included instandard Python distributions).
* **Standalone Binary Compilation** PyInstaller (optional, for standalone .exe generation).

**Network Testlab Configuration**
* **Virtualization Platform:** VMware Workstation
* **Network Mode:** Host-Only (VMnet1)
* **Target Systems:** Windows Host (Target 1) and Kali Linux (Target 2).

---

## Key Features

* **Host Discovery (Ping Sweep):** Sends ICMP echo requests simultaneously to different CIDR subnets, based on the user defined CIDR range. Multi-threaded execution pools to quickly map alive machines, (/24).
* **Passive OS Fingerprinting:** Recovers and analyzes the return values for the ICMP echo Time-To-Live (TTL) from IP packets. The architecture and the ability to classify target operating systems with accuracy, without sending intrusive probe traffic.
* **TCP Port Scanning:** Performs multi-threaded TCP handshakes (socket.connect) across standard enterprise service ports
* **Service & Version Banner Grabbing:** Starts low overhead banner requests (e.g., HTTP HEAD, SSH
Retrieval of server versions directly from open listening sockets (handshakes).

---

## Project Structure

```text
netprobe/
├── netprobe.py          # Main Python source script
├── netprobe.exe         # Pre-compiled standalone Windows executable
├── README.md            # Documentation and setup guide
├── Manual.pdf           # Detailed User's Manual
└── Presentation.pptx    # Project presentation slide deck

---

## Usage Instructions

### Basic Syntax
```bash
python netprobe.py <Target-IP-or-CIDR>
```
Or with the compiled binary:
```bash
netprobe.exe <Target-IP-or-CIDR>
```

### Examples

* **Single Host Scan:**
  ```bash
  python netprobe.py 192.168.56.101
  ```

* **Subnet Ping Sweep & Port Scan:**
  ```bash
  python netprobe.py 192.168.56.0/24
  ```

---

## Sample Terminal Output

```text
[*] Starting NetProbe scan against: 192.168.56.0/24
[*] Discovered 2 active host(s).

[+] Target: 192.168.56.101 | Fingerprint: Linux/Unix/macOS (TTL=64)
    PORT     SERVICE       VERSION/BANNER
    --------------------------------------------------
    22/tcp   SSH           SSH-2.0-OpenSSH_8.9p1 Ubuntu-3
    80/tcp   HTTP          Apache/2.4.52 (Ubuntu)

[+] Target: 192.168.56.102 | Fingerprint: Windows (TTL=128)
    PORT     SERVICE       VERSION/BANNER
    --------------------------------------------------
    445/tcp  SMB           Banner not returned
    3389/tcp RDP           Banner not returned
```

---

## Lab Simulation & Testbed Environment

NetProbe was verified in a host-only virtualized test network:
* **Attacker Machine:** Kali Linux / Windows 11 host executing `netprobe.py`.
* **Target VM 1 (Linux):** Ubuntu Server running Apache2 (`80`) and OpenSSH (`22`).
* **Target VM 2 (Windows):** Windows Server with SMB (`445`) and RDP (`3389`) enabled.
* **Verification:** Results cross-checked against baseline scans using `nmap -sV -O <target>`.

---

## Limitations & Future Roadmap

* **Firewall / NAT Evasion:** TTL-based OS detection can be skewed by intermediate routers decrementing hop counts or security devices mangling packet headers.
* **Scan Stealth:** Standard TCP Connect scans complete the full 3-way handshake, generating connection logs on target systems.
* **Roadmap:**
  * Implement raw TCP SYN (half-open) stealth scanning using raw sockets or Scapy.
  * Integrate UDP port scanning and comprehensive probe matching databases.
  * Add configurable port ranges and JSON/XML export capabilities.
