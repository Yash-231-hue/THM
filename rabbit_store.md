CloudSite — TryHackMe Style Write-Up

Room Type: Web / API / Privilege Escalation
Target: storage.cloudsite.thm
Difficulty: Medium
Status: Completed

Enumeration

Initial reconnaissance revealed a JavaScript-heavy web application. Client-side source inspection exposed several internal API endpoints:

/api/upload
/api/store-url
/api/uploads
/api/logout


These endpoints suggested authenticated functionality related to file handling and user sessions.

Authentication Analysis

The application relied on token-based authentication. Multiple token delivery mechanisms were accepted by the backend:

Authorization header

Cookies

Custom authentication headers

Despite this, token validation was weak. Invalid or manipulated tokens were rejected inconsistently, indicating improper verification of token integrity and claims.

Logic Flaw: Subscription Manipulation

While reviewing the registration workflow, a critical logic flaw was identified.

Vulnerable Endpoint
POST /api/register

Issue

User-controlled fields such as:

subscription

token claim values (iat, exp)

were trusted directly by the backend.

Impact

By registering with a modified request, a normal user could obtain an active subscription state, unlocking restricted functionality without proper authorization checks.

Authenticated Functionality Access

Once authenticated with elevated privileges:

File upload functionality became available

Uploaded files were accessible through predictable API paths

This confirmed that subscription status directly controlled access to sensitive features.

SSRF via URL Upload Feature

The application provided a URL-based upload mechanism. This functionality failed to restrict internal network access.

Result

Internal services on localhost were reachable

Port enumeration identified an internal service running on port 3000

Internal Service Disclosure

Accessing the internal service revealed an API documentation endpoint. This endpoint exposed additional internal routes, including a chatbot service not intended to be user-accessible.

This represented a classic case of internal API exposure via SSRF.

Remote Code Execution (User Access)

The chatbot endpoint:

Accepted JSON input

Was built using the Flask framework

Passed user input unsafely to backend logic

Due to insufficient input sanitization, this endpoint was vulnerable to server-side code execution. Exploiting this resulted in a reverse connection to the attacker, providing user-level shell access.

Flag Obtained

user.txt

Privilege Escalation
RabbitMQ Enumeration

Post-exploitation enumeration revealed a RabbitMQ installation. A sensitive Erlang cookie was found stored with weak file permissions:

/var/lib/rabbitmq/.erlang.cookie

Finding

The Erlang cookie allows authenticated interaction with the RabbitMQ node. Possession of this cookie effectively grants administrative-level access to the message broker service.

Impact

Abusing this misconfiguration enabled privilege escalation from the user account to a higher-privileged context, completing the room.

Attack Chain Summary

Client-side API discovery

Authentication and logic flaw abuse

Unauthorized subscription activation

SSRF through URL upload feature

Internal API documentation exposure

Command execution via chatbot endpoint

User shell access

Privilege escalation via Erlang cookie misuse

Lessons Learned

Client-side code should never expose sensitive API logic

Token claims must be validated server-side

Internal services should not be reachable from user input

Sensitive service credentials (Erlang cookies) must be strictly protected

Defense-in-depth is critical for API-driven applications

Disclaimer

This write-up documents exploitation performed only in an authorized TryHackMe lab environment.
All techniques are presented for educational purposes and must not be used against systems without explicit permission.
