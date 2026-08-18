import socket
import subprocess
import platform
import concurrent.futures
import ipaddress
import sys
import re

COMMON_PORTS = {
    21: "FTP", 22: "SSH", 23: "Telnet", 25: "SMTP", 
    53: "DNS", 80: "HTTP", 443: "HTTPS", 445: "SMB", 
    3306: "MySQL", 3389: "RDP", 8080: "HTTP-Proxy"
}

def ping_host(ip):
    param = "-n 1 -w 500" if platform.system().lower() == "windows" else "-c 1 -W 1"
    cmd = f"ping {param} {ip}"
    res = subprocess.run(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
    if res.returncode == 0:
        ttl_match = re.search(r"ttl=(\d+)", res.stdout, re.IGNORECASE)
        ttl = int(ttl_match.group(1)) if ttl_match else None
        return ip, ttl
    return None

def guess_os(ttl):
    if not ttl: return "Unknown OS"
    if ttl <= 64: return "Linux/Unix/macOS"
    if ttl <= 128: return "Windows"
    return "Network Device/Solaris"

def grab_banner(ip, port):
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.settimeout(1.5)
            s.connect((ip, port))
            if port in [80, 8080]:
                s.sendall(b"HEAD / HTTP/1.1\r\nHost: target\r\n\r\n")
            banner = s.recv(1024).decode(errors='ignore').strip()
            return banner.splitlines()[0] if banner else "Banner not returned"
    except Exception:
        return "No banner available"

def scan_port(ip, port):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.settimeout(0.8)
        if s.connect_ex((ip, port)) == 0:
            service = COMMON_PORTS.get(port, "Unknown Service")
            version = grab_banner(ip, port)
            return port, service, version
    return None

def scan_target(ip, ttl):
    print(f"\n[+] Target: {ip} | Fingerprint: {guess_os(ttl)} (TTL={ttl})")
    print("    PORT     SERVICE       VERSION/BANNER")
    print("    " + "-"*50)
    with concurrent.futures.ThreadPoolExecutor(max_workers=20) as executor:
        futures = [executor.submit(scan_port, ip, port) for port in COMMON_PORTS]
        for f in concurrent.futures.as_completed(futures):
            res = f.result()
            if res:
                port, srv, ver = res
                print(f"    {str(port)+'/tcp':<8} {srv:<13} {ver[:35]}")

def main():
    if len(sys.argv) < 2:
        print("Usage: python netprobe.py <IP or Subnet CIDR>")
        print("Example: python netprobe.py 192.168.1.0/24")
        return

    target = sys.argv[1]
    net = ipaddress.ip_network(target, strict=False)
    print(f"[*] Starting NetProbe scan against: {target}")
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=50) as executor:
        results = executor.map(ping_host, [str(ip) for ip in net.hosts()])
    
    active_hosts = [r for r in results if r is not None]
    print(f"[*] Discovered {len(active_hosts)} active host(s).")
    
    for ip, ttl in active_hosts:
        scan_target(ip, ttl)

if __name__ == "__main__":
    main()
