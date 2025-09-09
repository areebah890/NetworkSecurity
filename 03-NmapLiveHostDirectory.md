
---

# Nmap Live Host Directory - Try Hack Me room

---

## Objective 

| Objective | Description |
|-----------|-------------|
| Purpose of Nmap | Identify online systems and prepare for service discovery in later rooms |
| ARP Scan | Discover hosts on a local network using ARP requests |
| ICMP Scan | Detect live hosts via ICMP requests |
| TCP/UDP Ping Scan | Check host availability by sending TCP/UDP packets |
| Additional Tools | `arp-scan` for ARP-based scanning, `masscan` for high-speed host discovery |
| Nmap Overview | Free, open-source network mapper created by Gordon Lyon (Fyodor) in 1997 |
| Importance | Avoid scanning offline systems to save time and reduce network noise |

<details>
<summary>Subnetworks</summary>

## Network Segments and Subnetworks

- **Network Segment**: A group of computers connected using a shared physical medium, e.g., Ethernet switch or WiFi access point.  
- **Subnetwork (Subnet)**: One or more network segments logically grouped together and connected via a router; has its own IP address range.  
- **Key Difference**:
  - Network segment → Physical connection
  - Subnetwork → Logical connection
- **Additional Notes**:
  - A system typically connects to a single network segment/subnet.
  - Firewalls may enforce security policies between subnetworks.
 
<img width="1400" height="1400" alt="image" src="https://github.com/user-attachments/assets/ae206b57-49bd-4872-b7e3-7b94e3b03433" />

- above shows 4 network segements or subnetworks
- a system would be connected to  one of these subnetworks
- subnetwork has its own IP address range and is connected to a more extensive network via router
- subnets with /16 = the subnet mask can be written as 255.255.0.0 = subnet can have 65000 hosts
- subnets with /24 =bsubnet mas can be expressed as 255.255.255.0 = subnet can have 250 hosts

## Host Discovery Using ARP

- **Active Reconnaissance Goal**: Discover live hosts within a subnet or network.  
- **ARP (Address Resolution Protocol)**:
  - Used to obtain the hardware (MAC) address of hosts.
  - Can infer that a host is online if it responds to ARP queries.
- **Subnet Limitations**:
  - ARP can only discover hosts **within the same subnet** (e.g., 10.1.100.0/24).  
  - ARP queries **cannot cross routers**, so devices on other subnets require routed packets (via default gateway) to be reached.
- **Key Takeaway**: ARP is a link-layer protocol; its discovery capabilities are limited to the local subnet.

---

<img width="947" height="490" alt="image" src="https://github.com/user-attachments/assets/bf1a2aee-9485-4035-97a3-ce67e2fdb3b3" />

</details>

<details>
<summary>Enumerating Targets</summary>

## Nmap Target Specification and Host Discovery

- **Target Types:**
  - **List:** Scan specific hosts, e.g., `MACHINE_IP scanme.nmap.org example.com` → 3 IPs.
  - **Range:** Scan consecutive IPs, e.g., `10.11.12.15-20` → 6 IPs (`10.11.12.15` to `10.11.12.20`).
  - **Subnet:** Scan subnet, e.g., `MACHINE_IP/30` → 4 IPs.
  - **File Input:** Scan a list of hosts from a file using `-iL list_of_hosts.txt`.

- **Checking Targets without Scanning:**
  - Use `nmap -sL TARGETS` to list IPs without scanning.
  - Add `-n` to skip reverse-DNS lookups if desired.

- **Example Questions:**
  - First IP Nmap would scan for `10.10.12.13/29`: `10.10.12.8`
  - Number of IPs scanned for range `10.10.0-255.101-125`

- **Key Notes:**
  - Nmap allows flexible target specification (list, range, subnet, file).
  - Reverse-DNS resolution may reveal additional host information.

</details>

<details>
<summary>Discovering Live Hosts</summary>

## TCP/IP layers

- ARP from Link Layer
- ICMP from Network Layer
- TCP from Transport Layer
- UDP from Transport Layer

<img width="984" height="664" alt="image" src="https://github.com/user-attachments/assets/e68e2971-205c-4a39-87cc-12425941c989" />

## Brief Review of Protocols for Host Discovery

- **ARP (Address Resolution Protocol)**
  - Purpose: Broadcast a request on the local network asking the device with a specific IP to respond with its MAC address.
  - Useful for discovering hosts on the same subnet.

- **ICMP (Internet Control Message Protocol)**
  - Many types exist; for ping:
    - Type 8 → Echo Request
    - Type 0 → Echo Reply
  - ARP queries often precede ICMP Echo on the same subnet.

- **TCP & UDP**
  - Transport layer protocols.
  - Scanners can send specially-crafted packets to common ports to check for responses.
  - Useful when ICMP Echo is blocked, providing an efficient alternative for host discovery.

---

## Example Network Log
ARP REQUEST: Who has computer3 tell computer1
ARP RESPONSE: Hey computer1, I am computer3
PING: Sending Ping Request packet from computer1 to computer3
PING: computer3 received ping request from computer1, sending ping response to computer1
PING: Sending Ping Response packet from computer3 to computer1
PING: computer1 received ping response from computer3

<img width="955" height="466" alt="image" src="https://github.com/user-attachments/assets/1eba5929-4b23-4b57-bf52-0e45f0b2e78d" />

</details>

<details>
<summary>Nmap Host Discovery Using ARP</summary>

## How Nmap Discovers Live Hosts

Nmap uses different methods depending on user privileges and target location:

- **Privileged user on local network (Ethernet)**  
  - Uses **ARP requests** to discover live hosts.  

- **Privileged user on external network**  
  - Uses a combination of:  
    - ICMP Echo requests  
    - TCP ACK to port 80  
    - TCP SYN to port 443  
    - ICMP Timestamp requests  

- **Unprivileged user on external network**  
  - Falls back to a **TCP 3-way handshake** by sending SYN packets to ports 80 and 443.
 
## Nmap ARP Scan (Host Discovery Only)

- **Default behavior**:  
  - Nmap uses a ping scan to find live hosts, then scans ports on those hosts.  
  - To discover hosts without port-scanning, use:  
    ```bash
    nmap -sn TARGETS
    ```

- **ARP Scan**:  
  - Works **only if you’re on the same subnet** as the targets (Ethernet/WiFi).  
  - Communication requires knowing the **MAC address**, obtained via an **ARP query**.  
  - A host that replies to ARP is **online**.  
  - Expect many ARP queries when scanning a local network.  

- **Command for ARP scan only (no port scan)**:  
  ```bash
  nmap -PR -sn TARGETS
  ```
- Example
 ```bash
  nmap -PR -sn MACHINE_IP/24
  ```

<img width="362" height="145" alt="image" src="https://github.com/user-attachments/assets/2f20b2d5-c6b0-4f91-86d9-47f96cbd5da2" />



<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/1dbd927a-8ee0-409f-9f0d-9df55d562a6f" />

## `arp-scan` vs `nmap -PR -sn`

- **`arp-scan`**:  
  - A dedicated tool for ARP-based host discovery.  
  - Generates **ARP queries** to find live hosts on the same subnet.  

- **`nmap -PR -sn`**:  
  - Uses ARP for host discovery without port-scanning.  
  - Behavior and traffic are similar to `arp-scan`.  

- **Packet Capture**:  
  - When monitored with `tcpdump`, Wireshark, etc., both `arp-scan` and  
    `nmap -PR -sn` produce **similar ARP query patterns**.  
  - Each ARP request asks: *“Who has IP X? Tell me your MAC address.”*  
  - Hosts that reply indicate they are **up and reachable**.

- wireshark output

<img width="949" height="671" alt="image" src="https://github.com/user-attachments/assets/74278361-d2d1-4d4a-aa1d-f218d6375349" />

--- 

<img width="958" height="362" alt="image" src="https://github.com/user-attachments/assets/130b9bbd-da5e-439a-99a8-cd579e64a384" />


## ICMP Echo Host Discovery

- **Method**:  
  - Sends **ICMP Echo Request (Type 8)**.  
  - Expects an **ICMP Echo Reply (Type 0)** if the host is online.  

- **Nmap Command**:  
  - `nmap -PE -sn TARGETS`  
  - `-PE` → ICMP Echo Request  
  - `-sn` → Host discovery only (no port scan).  

- **Notes**:  
  - On the **same subnet**, an ARP query will always precede ICMP.  
  - **Limitations**:  
    - Firewalls often block ICMP Echo.  
    - Modern Windows systems block ICMP Echo by default.  
  - Therefore, ICMP discovery is simple but not always reliable.


<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/8464b32a-3ba8-4e34-a0da-9951033f9798" />

## ICMP Echo Scan Example

- **Command**:  
  ```bash
  sudo nmap -PE -sn MACHINE_IP/24
  ```
  
- **What it does:**
- Sends ICMP Echo Requests to all IPs in the /24 subnet.
- Expects Echo Replies from live hosts.

- **Key Point:**
- Simple way to check which hosts are up.
- Limitation: Many firewalls block ICMP Echo, so some live hosts may not reply.

<img width="376" height="332" alt="image" src="https://github.com/user-attachments/assets/411bff80-438d-4efb-928a-0e17ffed614c" />

- scan output shows that 8 hosts are up
- shows their MAC addresses
- we don't expect to learn the MAC addresses of targets unless they are on the same subnet as our system
- output above indicates Nmap did not need to send ICMP packets as it is confirmed that the hosts are up based on ARP response received

- repeat the scan above but scan from a system that belongs to a different subnet
- results are similar but without MAC addresses

<img width="307" height="254" alt="image" src="https://github.com/user-attachments/assets/29b79d4a-53d4-4a39-be75-4ffdc0364a69" />

- look at network packets with a tool like wireshark

<img width="979" height="671" alt="image" src="https://github.com/user-attachments/assets/f641c36c-919c-4b0b-b803-7b1f6cefcff8" />

- can see we have one source IP address
- on a different subnet that that of the destination subnet
- sends ICMP echo requests to all teh IP addresses in target subnet to see which will reply

<img width="979" height="671" alt="image" src="https://github.com/user-attachments/assets/7ade8bab-8063-4fb8-8829-0728fc1d0e0a" />

---

- as ICMP requests tend to be blocked may consider ICMP timestamp or ICMP address mark requests to tell if a system is online
- Nmap uses timestamp request (ICMP type 13) and check if it gets a timestamp reply (ICMP type 14)
- adding `-PP` option tells Nmap to use ICMP timestamp requests
- expect live hosts to reply

<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/a4a2142c-ee13-449a-a979-6b32d282822c" />

---

- run `nmap -PP -sn MACHINE_IP/24` to discover the online computers on the target machine subnet

<img width="313" height="253" alt="image" src="https://github.com/user-attachments/assets/27ad641b-254a-4c0a-b896-4321f9b31fae" />

- similar to previous ICMP scan this sends many ICMP timestamp requests to every valid IP address in target subnet

- wireshark below can see one source IP address sending ICMP packets to every possible IP address to discover online hosts

<img width="979" height="671" alt="image" src="https://github.com/user-attachments/assets/b26197e3-5116-444d-941e-52e2fab57884" />

--- 

- Nmap uses address mask queries (ICMP Type 17) and checks if it gets an address mask reply (ICMP type 18)
- scan enabled with `-PM`

<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/1c4dfdf2-3b3a-4f55-be90-a5fb9b9ae056" />

- attempt to discover live hosts using ICMP address mask queries run command `nmap -PM -sn MACHINE_IP/24`

<img width="296" height="80" alt="image" src="https://github.com/user-attachments/assets/7d058550-2c94-4e39-8cfd-61b345c93b4d" />



- based on previous scan we know at least 8 hosts are up, this scan returned none
- reason = the target system or firewall on the route is blocking this type of ICMP packet
- so it's essential to learn multiple approaches to achieve same result
- if one packet is being blocked we can choose another to discover the target network and services

- we did not get a reply and could not figure out which hosts are online it's essential to note that this scan sent ICMP address mask requests to every valid IP address and waited for a reply
- each ICMP request sent twice seen in screenshot below

<img width="979" height="671" alt="image" src="https://github.com/user-attachments/assets/6ef65705-2685-444a-a4ed-4ea936de2ec4" />

---

<img width="485" height="160" alt="image" src="https://github.com/user-attachments/assets/e2076c6e-2872-4e0c-bfca-ca891185b985" />

</details>

<details>
<summary>Nmap host discovery using TCP and UDP</summary>

## TCP SYN Ping

- can send a packet with SYN (synchronise) flag sent to a TCP port (80 by default) and wait for response
- an open report should reply with SYN/ACK (Acknowledge)
- closed port reult in RST (reset) = we only check if we will get any response to infer if the host is up
- specific state of port is not significant here

## TCP 3-way hanshake:

<img width="862" height="320" alt="image" src="https://github.com/user-attachments/assets/05ac0095-d750-44f6-9b0c-f128334dabc6" />

---

- if we want Nmap to use TCP SYN ping can do via `-PS` followed by port number, range, list or combinarion of them
- example:
  - `-PS21` targets port 21
  - `-PS21-25` target ports 21,22,23,24,24
  - `-PS80,443,8080` target the three ports 80,443,8080
 
  - priviledged users (root and sudoers) can send TCP SYN packets not needing to complete TCP 3-way handshake even if port is open
  - unpriviledged users have to use TCP 3-way handshake even if port is open

 <img width="862" height="320" alt="image" src="https://github.com/user-attachments/assets/56255146-c29a-4c35-9091-5428375978e1" />

- run `nmap -PS -sn MACHINE_IP/24` to scan target VM subnet
- output = discovered 5 hosts

<img width="328" height="181" alt="image" src="https://github.com/user-attachments/assets/31378ef2-07b6-414a-b1bb-afc80030b94b" />

- figure below shows closer look at what happened behind the scenes by looking at network traffic on Wireshark
- since we did not specify any TCP posts to use in TCP ping scan, Nmap used common ports (here TCP port 80)
- any service listening on port 80 is expected to reply = indirectly saying host is online

<img width="979" height="671" alt="image" src="https://github.com/user-attachments/assets/2cd68978-ce4b-4eb8-a94d-205b5eabd914" />


---

## TCP ACK Ping 

- this sends a packet with an ACK flag set
- must be running Nmap as a priviledged user to do this
- if done as unpriviledged Nmap will attemp 3-way handshake
- by default port 80 used
- `-PA` should be followed by port number, range, list or combo of them

- example:
    - `-PA21`, `-PA21-25` and `-PA80,443,8080`, if no port specified port 80 used

- following figure shows any TCP packet with an ACK flag should get TCP packet with an RSP flag set
- target respondss with RST flag set b/c TCP packet with ACK flag is not part of an ongoing connection
- expected response is used to detect if target host is up

<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/7a2d887a-b7c4-441c-8237-a1db33e06d08" />

---

- `sudo nmap -PA -sn MACHINE_IP/24` to discover online hosts on target's subnet
- TCP ACK ping scan detected 5 hosts as up


<img width="314" height="176" alt="image" src="https://github.com/user-attachments/assets/ead3d43a-0f99-4544-aad6-f4dfef6c2dba" />

- network traffic shows we discover many packets with ACK flag set and sent to port 80 of target systems
- Nmap sends each packet twice
- systems w/ no response are offline or inaccessible

<img width="979" height="671" alt="image" src="https://github.com/user-attachments/assets/4cbb7715-44a8-4f67-b00a-f38b7fa5cde0" />

---

## UDP Ping 

- following figure we see UDP packet sent to open UDP port and not triggering a response
  
<img width="862" height="240" alt="image" src="https://github.com/user-attachments/assets/29eb631e-054f-4332-bede-730a8a4b988e" />

- sending a UDP packet to any closed UDP port triggers a response (ICMP port unreachable packet) indirectly indicating the target is online

<img width="862" height="280" alt="image" src="https://github.com/user-attachments/assets/e78ffcf2-dceb-4f16-ad40-fb1a338d2632" />

---

- syntax to specify the ports  Nmap uses `-PU` for UDP ping

<img width="296" height="182" alt="image" src="https://github.com/user-attachments/assets/e8d45f2a-c52c-4d80-98fa-cfec85e926b1" />

- UDP scan discovers 5 live hosts

- in wireshark we notice Nmap sending UDP packets to UDP ports that are likely closed
- Nmap uses an uncommon UDP port to trigger an ICMP destination unreachable (port unreachable) error

<img width="979" height="671" alt="image" src="https://github.com/user-attachments/assets/41a16d36-221a-47d2-b76c-b0318bf17ec9" />

---

## Masscan 

- uses similar approach to discover available systems
- but to finish its network scan quickly it is aggressive with the rate of packets it generates
- `-p` can be followed by port number, list, or range
    - `masscan MACHINE_IP/24 -p443`
    - `masscan MACHINE_IP/24 -p80,443`
    - `masscan MACHINE_IP/24 -p22-25`
    - `masscan MACHINE_IP/24 ‐‐top-ports 100`
 
--- 

<img width="492" height="160" alt="image" src="https://github.com/user-attachments/assets/491b1553-eff6-450d-9f4d-941c5ca2dfb4" />

</details>

<details>
<summary>Using Reverse-DNS Lookup</summary>

- Nmap default behaviour is using reverse-DNS online hosts 
- b/c hostnames can reveal a lot this can be a helpful step 
- but if we don't want to send such DNS queries can use `-n` to skip step

- by default Nmap looks up online hosts
- `-R` to query the DNS server even for offline hosts
- if we want to use specific DNS server can add `--dns-servers DNS_SERVER` 

</details>

---

## Reflection

- learned how ARP, ICMP, TCP, UDP can detect live hosts
- any response from host is an indication that it is online

## command line options for Nmap 

| Scan Type              | Example Command                               | What It Does                                                                 |
| ---------------------- | --------------------------------------------- | ----------------------------------------------------------------------------- |
| ARP Scan               | `sudo nmap -PR -sn MACHINE_IP/24`             | Uses ARP requests to find live hosts on the same subnet (works only in LANs). |
| ICMP Echo Scan         | `sudo nmap -PE -sn MACHINE_IP/24`             | Sends ICMP Echo (ping) requests; replies indicate live hosts. May be blocked. |
| ICMP Timestamp Scan    | `sudo nmap -PP -sn MACHINE_IP/24`             | Sends ICMP Timestamp requests; replies confirm host is alive.                 |
| ICMP Address Mask Scan | `sudo nmap -PM -sn MACHINE_IP/24`             | Sends ICMP Address Mask requests to discover live hosts.                      |
| TCP SYN Ping Scan      | `sudo nmap -PS22,80,443 -sn MACHINE_IP/30`    | Sends TCP SYN packets to specified ports; responses show active hosts.        |
| TCP ACK Ping Scan      | `sudo nmap -PA22,80,443 -sn MACHINE_IP/30`    | Sends TCP ACK packets; replies indicate host is up (even if SYN is filtered). |
| UDP Ping Scan          | `sudo nmap -PU53,161,162 -sn MACHINE_IP/30`   | Sends UDP packets to specified ports; replies (or errors) show live hosts.    |

---

- add `-sn` if only interested in host discovery without port scanning
- omitting `-sn` let's Nmap default to port-scanning the live hosts

| Option | Purpose                              |
| ------ | ------------------------------------ |
| `-n`   | No DNS lookup                        |
| `-R`   | Perform reverse-DNS lookup for hosts |
| `-sn`  | Host discovery only (no port scan)   |

---

## Host Discovery
| Option | Purpose                                      |
| ------ | -------------------------------------------- |
| `-sn`  | Host discovery only (no port scan)           |
| `-PR`  | ARP discovery scan (local network only)      |
| `-PE`  | ICMP Echo request                            |
| `-PP`  | ICMP Timestamp request                       |
| `-PM`  | ICMP Address Mask request                    |
| `-PS`  | TCP SYN ping (to given ports, e.g., 80,443)  |
| `-PA`  | TCP ACK ping (to given ports)                |
| `-PU`  | UDP ping (to given ports, e.g., 53,161)      |
| `-n`   | Disable DNS resolution                       |
| `-R`   | Force reverse-DNS lookup for all hosts       |

---

## Port Scanning
| Option | Purpose                                      |
| ------ | -------------------------------------------- |
| `-p`   | Specify port(s) (e.g., `-p 22,80,443` or `-p-` for all) |
| `-F`   | Fast scan (top 100 ports)                   |
| `-r`   | Scan ports sequentially (no randomization)  |
| `-sS`  | SYN (stealth) scan                          |
| `-sT`  | TCP connect scan                            |
| `-sU`  | UDP scan                                    |
| `-sA`  | TCP ACK scan                                |
| `-sW`  | TCP Window scan                             |
| `-sM`  | TCP Maimon scan                             |

---

## Timing & Performance
| Option | Purpose                                      |
| ------ | -------------------------------------------- |
| `-T0`  | Paranoid (very slow, IDS evasion)            |
| `-T1`  | Sneaky (slow)                                |
| `-T2`  | Polite (reduced load)                        |
| `-T3`  | Normal (default)                             |
| `-T4`  | Aggressive (faster, noisier)                 |
| `-T5`  | Insane (fastest, very noisy)                 |

---

## Output
| Option     | Purpose                                  |
| ---------- | ---------------------------------------- |
| `-oN file` | Normal output to file                    |
| `-oG file` | Grepable output                          |
| `-oX file` | XML output                               |
| `-oA file` | Output in all formats (file.nmap, .xml, .gnmap) |

---

## Other Useful Options
| Option | Purpose                                      |
| ------ | -------------------------------------------- |
| `-A`   | Aggressive scan (OS detection, versioning, scripts, traceroute) |
| `-O`   | OS detection                                 |
| `-sV`  | Service/version detection                    |
| `--script` | Run NSE scripts (e.g., `--script=vuln`)  |
