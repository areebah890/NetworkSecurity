
---

# Active Reconnaissance - Try Hack Me room

---

## Objective 

1. Understand the difference between passive and active reconnaissance.
2. Learn that active reconnaissance involves direct engagement with the target, such as network connections or social engineering.
3. Recognize the importance of legal authorization before performing active recon.
4. Explore web browsers and developer tools as reconnaissance frameworks.
5. Use common command-line tools to gather information:

ping – check host availability and response times.

traceroute – map network paths.

telnet – test service connectivity.

nc (netcat) – interact with services and ports.

6. Understand that active reconnaissance may leave traces in logs and how to distinguish normal activity from suspicious activity.
7. Gain familiarity with both GUI-based and command-line tools for efficient active recon.


---

<details>
<summary>Web Browser</summary>

## Using a Web Browser for Active Reconnaissance

- A web browser is readily available on all systems and can be used for active recon.
- Default transport ports:

HTTP → TCP port 80

HTTPS → TCP port 443

- Custom ports can also be accessed, e.g., https://127.0.0.1:8834/ connects to port 8834.
- Developer Tools shortcuts:

Windows/Linux: Ctrl + Shift + I

Mac: Option + Command + I (⌥ + ⌘ + I)

- Capabilities of Developer Tools:

Inspect and modify JavaScript files.

View cookies set by the server.

Explore site folder structure and resources.


- Tip: Developer Tools turns your browser into a lightweight reconnaissance framework for inspecting server responses and website structure.

## Browser Add-ons for Penetration Testing

1. FoxyProxy

- Quickly switch proxy servers while browsing.
- Useful when using tools like Burp Suite or managing multiple proxies.

2. User-Agent Switcher and Manager
- Change your browser’s user-agent to mimic different devices or browsers.
- Example: Pretend to browse a site from an iPhone while using Firefox.

3. Wappalyzer
- Detects and displays technologies used by websites (e.g., CMS, frameworks, analytics tools).
- Helps gather tech information while browsing like a normal user.

**Tip:** These extensions enhance browser-based active reconnaissance by making it easier to gather detailed target information.

--- 

Browse to the following website and ensure that you have opened your Developer Tools on AttackBox Firefox, or the browser on your computer. Using the Developer Tools, figure out the total number of questions.
8

<img width="946" height="491" alt="image" src="https://github.com/user-attachments/assets/223cc9c7-eea0-43ae-8316-6f7d44fbc31b" />

</details>

<details>
<summary>Ping</summary>

**Purpose:**
- Check if a remote system is online and reachable.

**How it works:**
- Sends an **ICMP Echo Request** to the target.
- Target responds with an **ICMP Echo Reply** if reachable.

**Key Uses:**
- Verify network connectivity between two systems.
- Ensure the target system is online before performing detailed scans.

**Notes:**
- Replies may be blocked by firewalls, so no response doesn’t always mean the system is down.
- Simple yet essential tool for active reconnaissance.

---

**Command Examples:**
- Linux/macOS: `ping <target>` (use `Ctrl+C` to stop)
- Send specific number of packets: `ping -c 10 10.10.22.90`
- Windows: `ping -n 10 10.10.22.90` (send 10 packets)
  
<img width="386" height="241" alt="image" src="https://github.com/user-attachments/assets/2e10fe99-ebfa-4739-a53a-a0d601816cc3" />

- can see target is responindg
- ping output indicates that it is online and reachable 

- from penetration testing pov we try to discover more about this target
- e.g. which ports are open and which services are running 

<img width="345" height="157" alt="image" src="https://github.com/user-attachments/assets/b21b87bd-b8b2-4410-bd6e-b071ed000c46" />

- example above we have shut down the target VM and then tried to ping 10.10.22.90
- we do not recieve a reply "destination host unreachable"
- transmitted 5 packets, none were received = 100% packet loss

**Possible Reasons for No Reply:**
- Target system is off, crashed, or still booting.
- Network issues or unplugged devices along the path.
- Firewall blocks ICMP packets (software or hardware firewall).
- Windows firewall blocks ping by default.
- Your own system is disconnected from the network.


**ICMP Header Size:** 8 bytes  
- **Type:** 1 byte  
- **Code:** 1 byte  
- **Checksum:** 2 bytes  
- **Identifier:** 2 bytes  
- **Sequence Number:** 2 bytes  

- This is in addition to the **data payload**, which can be set using:
  - Linux/macOS: `-s`
  - Windows: `-l`

</details>

<details>
<summary>Traceroute</summary>

## Traceroute Command

**Purpose:**
- Trace the path packets take from your system to a target host.
- Discover IP addresses of routers (hops) along the route.
- Determine the number of hops between source and target.

**How it works:**
- Relies on **ICMP** messages to identify each hop.
- Uses **TTL (Time To Live)** in the IP header to control hop count:
  - TTL decrements by 1 at each router.
  - When TTL reaches 0, the packet is dropped and the router replies, revealing its IP.

**Commands:**
- **Linux/macOS:** `traceroute <target>`
- **Windows:** `tracert <target>`

**Notes:**
- Route may change due to dynamic routing protocols.
- TTL is the **maximum number of hops**, not time.
- Example: Packet sent with TTL 64 reaches target with TTL 60 → passed through 4 routers.

<img width="315" height="435" alt="image" src="https://github.com/user-attachments/assets/3dc85805-6a42-482d-8a3e-c0058574ca05" />

- if TTL reaches 0 it will be dropped and an ICMP Time-to-Live exceeded will be sent to original sender
- in bellow figure the system set TTL to 1 before sending it to router
- teh first router on teh path decrements the TTL by 1 resulting in a TTL of 0
- therefore this router discards the packet and sends an ICMP time exceeded in-transit error message
- some routers are configured not to send such ICMP messages when discarding a packet

<img width="488" height="434" alt="image" src="https://github.com/user-attachments/assets/27640903-62d3-4f02-94a5-4638309aba9f" />

- On Linux, `traceroute` starts by sending **UDP packets** with TTL = 1.
- The first router decrements TTL to 0, drops the packet, and sends back an **ICMP Time-to-Live Exceeded** message.
- This reveals the **IP address of the first router**.
- Traceroute then sends packets with increasing TTLs (2, 3, …), each revealing the next hop along the path.
- This process continues until the packet reaches the target host.

---
`traceroute` tryhackme.com from TryHackMe’s AttackBox

<img width="346" height="174" alt="image" src="https://github.com/user-attachments/assets/40052796-65b4-4382-a7b6-48196f820b2f" />

**Target IP:** 104.20.29.66
**Max hops:** 30
**Packet size:** 60 bytes

| Hop | IP Addresses (Responded)                      | RTT (ms)              | Description                                     |
| --- | -------------------------------------------- | -------------------- | ----------------------------------------------- |
| 1   | 244.5.0.227 / 244.5.0.221 / 244.5.0.229     | 4.983 / 6.717 / 6.617 | Likely local network or ISP edge router        |
| 2   | 240.1.92.13 / 240.1.92.33 / 240.1.92.15     | 0.283 / 0.252 / 0.203 | Second hop along the path                      |
| 3   | 242.3.170.53 / 242.3.171.183 / 242.3.171.49 | 0.731 / 1.441 / 1.185 | Intermediate router in ISP or regional network |
| 4   | 240.4.244.14 / 240.4.244.15 / 240.4.244.12  | 1.607 / 1.623 / 0.997 | ISP backbone router                             |
| 5   | 242.13.251.1 / 242.13.250.135 / 242.13.250.5| 0.845 / 0.976 / 1.556 | ISP backbone / transit network                  |
| 6   | 151.148.8.200                                | 1.710 / 1.825 / 2.053 | ISP backbone router                             |
| 7   | 151.148.8.201                                | 2.516 / 2.472 / 2.297 | ISP backbone router                             |
| 8   | 162.158.36.39 / 162.158.36.41                | 2.455 / 6.376 / 6.247 | Likely part of Cloudflare network               |
| 9   | 104.20.29.66                                 | 1.114 / 1.553 / 1.538 | Target server (tryhackme.com) reached          |

## Key Points

- TTL (Time To Live): Each hop decrements TTL by 1; routers return ICMP "Time Exceeded" until the target is reached.
- 3 probes per hop: Helps detect inconsistent routers or temporary delays.
- RTT values: Measure latency to each hop in milliseconds.
- Traceroute stops when the target host is reached or the maximum hop count is exceeded.
- Tip: Use traceroute to identify network paths, bottlenecks, or firewall/router behavior during active reconnaissance.

  </details>

<details>
<summary>Telnet</summary>

## Telnet

- **Protocol:** TELNET (Teletype Network), developed in 1969.
- **Purpose:** Communicate with a remote system via a command-line interface (CLI) for administration.
- **Default Port:** 23
- **Security Concerns:**
  - Sends all data, including usernames and passwords, in **cleartext**.
  - Vulnerable to interception and credential theft.
- **Secure Alternative:** SSH (Secure SHell) protocol.

## Telnet for Banner Grabbing and Service Interaction

- **TCP-Based:** The Telnet client relies on the TCP protocol, allowing connections to any TCP service.
- **Usage:** `telnet <IP> <PORT>` connects to the target service.
- **Banner Grabbing:** Telnet can be used to retrieve service banners unless the service uses encryption.
- **Example – HTTP Server on Port 80:**
  1. Connect: `telnet 10.10.22.90 80`
  2. Request a page using HTTP protocol: `GET / HTTP/1.1`
  3. Specify the host header: `Host: example.com` and press **Enter** twice.
- **Result:** The server responds with the requested page or resource.

<img width="288" height="215" alt="image" src="https://github.com/user-attachments/assets/c45574bc-4a56-4515-97a9-5de5544148e8" />

## Identifying Service Type and Version

- **Goal:** Discover the type and version of the target service.
- **Example – Web Server:**
  - Response: `Server: nginx/1.6.2`
  - Shows the server software and version.
- **Protocol-Specific Commands:**
  - For web servers → HTTP commands (e.g., GET / HTTP/1.1)
  - For mail servers → SMTP or POP3 commands
- **Purpose:** Helps determine potential vulnerabilities associated with the service version.

--- 
Start the attached VM from Task 3 if it is not already started. On the AttackBox, open the terminal and use the telnet client to connect to the VM on port 80. What is the name of the running server? 
Apache 

<img width="215" height="74" alt="image" src="https://github.com/user-attachments/assets/ffd91369-0e20-4cb7-b032-3632dfe88ede" />

What is the version of the running server (on port 80 of the VM)? 2.4.61

</details>

<details>
<summary>Netcat</summary>

- Netcat (`nc`) is a versatile networking tool for penetration testers.
- Supports both **TCP** and **UDP** protocols.
- Can function as:
  - **Client**: Connects to a listening port.
  - **Server**: Listens on a port of your choice.
- Useful for collecting banners, testing connectivity, or transferring data.

## Example Usage

- Connect to a server to collect a banner (similar to Telnet):

```bash
nc 10.10.22.90 PORT
```
- When sending HTTP requests, you might need to press SHIFT+ENTER after typing the GET line.
- Acts as a simple client or server over TCP/UDP.
- Tip: Netcat is often called the "Swiss Army knife" of networking due to its versatility in pentesting.
  
<img width="203" height="167" alt="image" src="https://github.com/user-attachments/assets/00875740-c7f3-462d-bcc7-daf48e667ab8" />

- in example above, used netcat to connect to 10.10.22.90 port 80 using `nc 10.10.22.90 80`
- then issues a get for teh default page using `GET / HTTP/1.1` (we are specifying to target server that our client supports HTTP version 1.1)
- finally we give the name to our host so add host: netcat
- based on output Server: `nginx/1.6.2` can tell that on port 80 we have version 1.6.2 listening for incoming connections

- can use netcat on a TCP port and connect to a listening port on another system

- on the server system where we want to open a port and listen to it can issue nc -lp 1234 or nc -vnlp 1234 (equivalent to nc -v -1 -n -p 1234)

| Option | Meaning |
|--------|---------|
| -l     | Listen mode |
| -p     | Specify the port number |
| -n     | Numeric only; no resolution of hostnames via DNS |
| -v     | Verbose output (optional, useful for debugging) |
| -vv    | Very verbose output (optional) |
| -k     | Keep listening after client disconnects |

- The option `-p` should appear just before the port number you want to listen on.
- The option `-n` avoids DNS lookups and related warnings.
- Port numbers less than 1024 require root privileges to listen on.
- On the client-side, connect to the server using:

`nc 10.10.22.90 PORT_NUMBER`

- Example: Using Netcat to echo messages

1. **Server-side**: Listen on port 1234
`nc -vnlp 1234`
*(equivalent to `nc -lvnp 1234`)*

2. **Client-side**: Connect to the server
`nc 10.10.22.90 1234`

- Once connected, whatever you type on the client-side will be echoed on the server-side, and vice versa, creating a simple TCP tunnel for communication.

---

</details>

## Active Reconnaissance Summary

In this room, we covered several fundamental tools for active reconnaissance. These tools can be combined in scripts to build a basic network and system scanner. They allow you to:

- Map the path to the target (traceroute)
- Check if the target system responds (ping)
- Identify open and reachable ports (telnet and netcat)
- Advanced scanners like nmap automate these tasks at a much more sophisticated level.

| Tool / Purpose   | Example Command (Linux/macOS)    | Example Command (Windows)        |
| ---------------- | -------------------------------- | -------------------------------- |
| Ping             | `ping -c 10 10.10.22.90`         | `ping -n 10 10.10.22.90`         |
| Traceroute       | `traceroute 10.10.22.90`         | `tracert 10.10.22.90`            |
| Telnet           | `telnet 10.10.22.90 PORT_NUMBER` | `telnet 10.10.22.90 PORT_NUMBER` |
| Netcat as client | `nc 10.10.22.90 PORT_NUMBER`     | `nc 10.10.22.90 PORT_NUMBER`     |
| Netcat as server | `nc -lvnp PORT_NUMBER`           | `nc -lvnp PORT_NUMBER`           |

## Web Browser Developer Tools

A web browser is a powerful tool for active reconnaissance because it is installed on almost every system. Using Developer Tools, you can inspect website resources, view cookies, and explore web page structures without raising alarms.

| Operating System | Developer Tools Shortcut           |
| ---------------- | ---------------------------------- |
| Linux / Windows  | `Ctrl + Shift + I`                 |
| macOS            | `Option + Command + I (⌥ + ⌘ + I)` |

