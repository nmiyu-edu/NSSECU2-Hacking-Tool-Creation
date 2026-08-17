# NSSECU2-Hacking-Tool-Creation

NetProbe is a lightweight, multi-threaded CLI network reconnaissance and vulnerability scanning utility written in Python. Patterned after core NMAP scanning principles, it performs rapid host discovery, TCP port scanning, passive OS fingerprinting, and service/version banner grabbing using only Python standard libraries.

---

## Key Features

* **Host Discovery (Ping Sweep):** Rapidly discovers alive systems across CIDR target subnets (`/24`) using concurrent ICMP echo requests.
* **Passive OS Fingerprinting:** Inspects Time-To-Live (TTL) return values from echo responses to infer remote operating systems without invasive probes:
  * $\text{TTL} \le 64$: Linux / Unix / macOS
  * $\text{TTL} \le 128$: Windows OS
  * $\text{TTL} \le 255$: Network Appliances / Cisco IOS / Solaris
* **TCP Port Scanning:** Executes asynchronous TCP Connect handshakes (`socket.connect_ex`) against common service ports.
* **Service & Version Banner Grabbing:** Interrogates open ports with initial payload requests (e.g., HTTP HEAD, SSH banners) to identify service versions.
* **Zero External Dependencies:** Built entirely with native Python modules (`socket`, `subprocess`, `concurrent.futures`, `ipaddress`, `re`).
* **Standalone Executable:** Fully compatible with PyInstaller for one-click compilation into a portable `.exe` binary.

---

## Project Structure

```text
netprobe/
├── netprobe.py          # Main Python source script
├── netprobe.exe         # Pre-compiled standalone Windows executable
├── README.md            # Documentation and setup guide
├── Manual.pdf           # Detailed User's Manual and lab testing report
└── Presentation.pptx    # Project presentation slide deck
```

---

## Prerequisites & Installation

### Option 1: Running from Python Source
* Python 3.8 or higher installed.
* Standard library modules only (no `pip install` required for runtime).

### Option 2: Building Standalone Executable (`.exe`)
To build the binary yourself using PyInstaller:
```bash
pip install pyinstaller
pyinstaller --onefile netprobe.py
```
The output binary will be generated inside the `dist/` directory as `netprobe.exe`.

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

---

## Disclaimer

This tool is designed and developed strictly for authorized educational research, network defense evaluation, and simulated penetration testing labs. Unauthorized scanning of networks without prior written permission is illegal.
