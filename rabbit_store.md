# Rabbitstore – TryHackMe Write-Up

**Target:** storage.cloudsite.thm  
**Category:** Web / API / Privilege Escalation  
**Difficulty:** Medium  
**Environment:** Authorized TryHackMe Lab

---

## Enumeration

Initial analysis of the web application revealed several API endpoints exposed through client-side JavaScript:

/api/upload  
/api/store-url  
/api/uploads  
/api/logout  

These endpoints suggested authenticated functionality related to file handling and session management.

---

## Authentication & Token Handling

The application relied on token-based authentication and accepted tokens through multiple mechanisms:

- Authorization header (Bearer token)  
- Cookies  
- Custom authentication headers  

Token validation was improperly implemented. While invalid tokens were sometimes rejected, the backend failed to strictly validate token claims, opening the door to logical abuse.

---

## Logic Flaw: Subscription Manipulation

A critical logic flaw was identified in the user registration process.

**Vulnerable Endpoint:**  
POST /api/register  

The backend trusted user-supplied fields such as:

- subscription status  
- token-related claims (iat, exp)  

By modifying the registration request, it was possible to register an account with an active subscription state, bypassing intended access controls.

**Impact:**  
Restricted functionality, including file uploads, became available to a standard user.

---

## Authenticated Access & File Uploads

After authentication with the manipulated subscription status:

- Upload functionality was accessible  
- Uploaded files could be retrieved via predictable API paths  

This confirmed that authorization was enforced solely through the flawed subscription logic.

---

## SSRF via URL Upload Feature

The application included a feature that allowed users to upload files via remote URLs. This feature failed to restrict access to internal network resources.

### Findings
- Requests to localhost were permitted  
- An internal service was discovered running on port 3000  

This indicated a Server-Side Request Forgery (SSRF) vulnerability.

---

## Internal API Disclosure

Using SSRF, an internal API documentation endpoint was accessed on the local service. This documentation revealed additional internal routes, including a chatbot API that was never intended to be exposed externally.

---

## Remote Code Execution (User Shell)

The chatbot endpoint:

- Accepted JSON input  
- Was implemented using Flask  
- Passed user input unsafely to backend logic  

Due to insufficient input sanitization, the endpoint was vulnerable to server-side code execution. Exploiting this resulted in a reverse shell connection and **user-level access**.

**Flag Obtained:**  
user.txt

---

## Privilege Escalation

### RabbitMQ Misconfiguration

Post-exploitation enumeration revealed a RabbitMQ installation. A sensitive Erlang cookie was found stored with weak permissions:

/var/lib/rabbitmq/.erlang.cookie  

This cookie is used to authenticate Erlang nodes and RabbitMQ administrative actions.

### Impact

Possession of the Erlang cookie allowed authenticated interaction with the RabbitMQ service, leading to privilege escalation and full system compromise.

---

## Attack Chain Summary

1. Client-side API enumeration  
2. Authentication weakness identification  
3. Subscription logic manipulation  
4. Unauthorized access to upload functionality  
5. SSRF via URL upload feature  
6. Internal API documentation exposure  
7. Code execution through chatbot endpoint  
8. User shell access  
9. Privilege escalation via Erlang cookie  

---

## Lessons Learned

- Client-side code should not expose sensitive API endpoints  
- Token claims must always be validated server-side  
- Authorization logic should never rely on user-controlled fields  
- Internal services must be isolated from user input  
- Service credentials such as Erlang cookies must be strictly protected  

---

## Disclaimer

This write-up documents exploitation performed **only in an authorized TryHackMe lab environment**.  
All information is provided strictly for educational purposes.
