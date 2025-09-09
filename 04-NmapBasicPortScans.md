---

# Nmap Basic Port Scans - Try Hack Me room

---

## Objective 

1. check which ports are open and listening and which are closed
2. learn the different types of port scans used by Nmap
- TCP connect port scan
- TCP SYN port scan
- UDP port scan

---

<details>
<summary>TCP and UDP ports</summary>

- in the way IP address specifies a host on a network among many others, a TCP port or UDP port are used to identify a network service running on that host
- server provides the network service and adheres to specific network protocol
- examples:
     - providing time
     - responding to DNS queries
     - serving web pages
- a port is usually linked to a service using that specific port number
     - a HTTP server would bind to TCP port 80 by default
     - if the HTTP supports SSL/TLS it listens to TCP port 443
     - these are defualt ports for HTTP and HTTPS but webserver administrators can choose other port numbers if necessary
 
## Port states
- put very simpily we can classify ports in 2 states:
1. open port = there is some service listening on that port
2. closed port = there is no service listening on that port
- in practical solutions we need to consider the impact of firewalls (a port can be open but firewall is blocking packets

**Nmap considers the following 6 states:**

1. **Open:** service is listening on the specified port
2. **Closed:** no service is listening on the specified but the port is accessible (it is reachable and not blocked by a firewall or other security appliances/programs)
3. **Filtered:** Nmap cannot determine if port is open or closed b/c port is not accessible - usually due to firewall preventing Nmap from reaching port. Nmap's packets  may be blocked from reaching the port or responses are blocked from reaching Nmap's host
4. **Unfiltered:** Nmaap cannot determine if port open or closed but the port is accessible (state encountered when using an ACK scan `-sA`)
5. **Open|Filtered:** Nmap cannot determine if port is open or filtered
6. **Closed|Filtered:** Nmap cannot decide if port is closed or filtered

--- 

<img width="490" height="188" alt="image" src="https://github.com/user-attachments/assets/0c0ffc20-9932-4eb1-baf9-de4fa06df8e5" />

- an open port indicates service is actively listening on that port, so it may be accessible for interaction, exploitation, or further enumeration

</details>


<details>
<summary>TCP Flags</summary>

- Nmap supports diff types of TCP port scans
- TCP header is the first 24 bytes of a TCP segment

<img width="586" height="386" alt="image" src="https://github.com/user-attachments/assets/6c23f813-93c8-42c2-a23e-87ed1f9207ed" />

- figure shows TCP header defined as RFC793
- first row is the source TCP port number and destination port number (can see is allocated 16 bits= 2 bytes)
- second and third rows have sequence number and acknowledgement number (each row has 32 bits = 4 bytes allocated, w/ 6 rows total = 24 bytes)

- particularly focus on flags Nmap can set or unset
- TCP flags in figure are highlighted in red
- setting flag bit = setting value 1

From left to right TCP header flags are:

1. **URG:** urgent flag = urgent pointer filed is significant. urgent pointer = incoming data is urgent and TCP segment w/ URG flag set is processed immediately (doesn't consider previously sent TCP segments)
2. **ACK:** acknowledgement flag = the ack number is significant, used to acknowledge the receipt of a TCP segment
3. **PSH:** push flag = asking TCP to pass the data to application promptly
4. **RST:** reset flag = reset the connection. another device (like firewall) might send it to tear a TCP connection. also used when data is sent to a host and there is no service on receiving end to answer
5. **SYN:** synchronize flag = used to iniciate TCP 3-way handshake and synchronize sequence numbers w/ other host. sequence number should be set randomly during TCP connection establishment.
6. **FIN:** sender has no more data to send 

</details>

<details>
<summary>TCP Connect Scan</summary>

- works by completing TCP 3-way handshake
- in standard TCP connection establishment client sends TCP packet with SYN flag set and server responds with SYN/ACK if port open the client responds w/ 3-way handshake by sending ACK

<img width="862" height="320" alt="image" src="https://github.com/user-attachments/assets/11356a5a-2cf5-4463-ba1b-c7e459a876b0" />

- we are interested in learning if TCP port is opne not establishing TCP connection
- so connection is torn as soon as state is confirmed by sending RST/ACL
- can choose to run TCP connect scan w/ `-sT`

<img width="862" height="360" alt="image" src="https://github.com/user-attachments/assets/40605892-b946-4f54-a596-f5643772300a" />

---

- if not a priviledged user (root or sudoer) a TCP connect scan is only possible option to discover open TCP ports
- figure below shows wireshark packet capture
- see Nmap sending TCP packets w/ SYN flag set to ports 256, 443, 143 etc.
- default Nmap attempts to connect to the 1000 most common ports
- a closed TCP port responds to SYN packet w/ RST/SYN indicating it's not open
- this pattern repeats for all closed ports as we attempt to initiate TCP 3-way handshake w/ them

<img width="1011" height="671" alt="image" src="https://github.com/user-attachments/assets/27c415d9-6ebe-4d4a-9293-7c515b845c62" />

- notice that port 143 so replies w/ SYN/ACK and Nmap completes 3-way handshake by sending ACK
- figure below shows packets exchanged between out Nmap host and target system's port 143
- first 3 packets are TCP 3-way handshake being completed
- fourth tears it down w/ RST/ACK packet

<img width="1011" height="336" alt="image" src="https://github.com/user-attachments/assets/20fb81b6-8ba3-476d-a934-8e46b6ddaf1d" />

---

- TCP connect scan `-sT` command example returns detailed list of open ports

<img width="338" height="243" alt="image" src="https://github.com/user-attachments/assets/b18c921f-6501-410d-a545-9a0dabc227de" />


- `-F` = enable fast mode decrease number of ports scanned from 1000 to 100 most common ports
- `-r` = added to scan the ports in consecutive order instead of random (useful when testing whether ports open in consistent manner like when a target boots up)

---

<img width="862" height="482" alt="image" src="https://github.com/user-attachments/assets/3ef4a646-f843-4311-a24b-a8c013e50b94" />

</details>

<details>
<summary>TCP SYN Scan</summary>

- unpriviledged users = limited to connect scan
- but default scan mode is SYN scan (it requires a priviledged (root or sudoer) user to use it)
- SYN scan doesn't need to complete TCP 3-way handshake instead tears down the the connection once it receives response from server
- b/c no TCP connection established = decreases chances of scan being logged
- can select this scan type using `-sS` option

<img width="862" height="320" alt="image" src="https://github.com/user-attachments/assets/6c1931ca-7765-40b3-a390-35946136219c" />

- shows how TCP SYN scan works w/o completing TCP 3-way handshake

<img width="1011" height="671" alt="image" src="https://github.com/user-attachments/assets/214a9a0d-a23a-4f32-babc-28cc756efc4f" />

- above figure from wireshark shows TCP SYN scan behaviour when TCP port closed is simialr to TCP connect scan

--- 

## difference between the 2 scans 

<img width="1011" height="660" alt="image" src="https://github.com/user-attachments/assets/75b7f3bd-2fdc-4e22-955c-2d1965ccd30d" />

- upper half we see TCP connect scan `-sT` traffic
- any open TCP port require Nmap to complete TCP 3-way handshake before closing connection

  - lower half can see SYN scan `-sS` doesn't need to complete 3-way handshake, instead Nmap sends RST packet once a SYN/ACK packet is received
 
---

- TCP SYN scan is default scan mode when running Nmap as priviledged user (root or sudo) and is very relaible choice
- it has successfully discovered the open ports i found earlier w/ TCP connect scan yet no TCP connection was fully established w/ the target 

<img width="331" height="228" alt="image" src="https://github.com/user-attachments/assets/5797a895-0a2d-4442-99c0-7938594889ed" />

---

<img width="918" height="336" alt="image" src="https://github.com/user-attachments/assets/3d815a40-6794-40bb-9bbf-3bd247e0e525" />

</details>

<details>
<summary>UDP Scan</summary>

- UDP = connectionless protocol = no handshake required for connection establishment
- cannot guarantee a service listening on UDP port would respond to our packets
- but if UDP packet sent to closed port an ICMP port unreachable error (type 3, code 3) is returned
- can select UDP scan w/ `-sU` option
- you can combine it w/ another TCP scan

---

<img width="862" height="220" alt="image" src="https://github.com/user-attachments/assets/6e49afcc-b791-42fa-b06c-d689fc6f6172" />

- shows that if we send UDP packet to open port we cannot expect reply
- so sending UDP packet to open port won't tell us anything

---

<img width="862" height="280" alt="image" src="https://github.com/user-attachments/assets/27af0899-75ad-43f7-8e89-abee9a328403" />

- we expect to get ICMP packet of type 3 destination unreachable and code 3 port unreachable
- aka UDP ports that do not generate any response are ones Nmap state as open

---

<img width="1011" height="671" alt="image" src="https://github.com/user-attachments/assets/10b8c10d-38a5-4e45-81d1-048fb4fa593a" />

- wireshark capture shows every closed port will generate an ICMP packet destination unreachable (port unreachable)

---

- launching UDP scan against Linux proved valuable
- learn that port 111 is open
- but Nmap cannot determine if UDP port 68 is open or filterd 

---

<img width="914" height="356" alt="image" src="https://github.com/user-attachments/assets/ab12b0e3-7140-4344-9355-4c082a331509" />

</details>

<details>
<summary>Fine-Tuning Scope and Performance</summary>

- you can specify ports to be scanned instead of default 1000
- port list: `-p22,80,22` scans ports 22, 80, 443
- port range: `-p1-1023` scan all ports 1-1023 inclusive, `-p20-25` 20-25 inclusive

- can request scan all ports using `-p-` (scans all 65535 ports)
- scan the most common 100 ports add `-F`
- using `--top--ports 10` = 10 most common ports

- control scan timing `-T<0-5>`
- `-T0` = slowest (paranoid)
- `-T5` = fastest 

### Timing Templates (`-T0` to `-T5`)

| Template    | Value | Description                                                                 |
|------------|-------|-----------------------------------------------------------------------------|
| Paranoid   | 0     | Slowest scan; one port at a time with long waits (stealthy)                 |
| Sneaky     | 1     | Very slow scan to avoid detection                                            |
| Polite     | 2     | Reduces load on the network                                                 |
| Normal     | 3     | Default speed (used if no `-T` specified)                                   |
| Aggressive | 4     | Faster scan; often used in CTFs or practice environments                    |
| Insane     | 5     | Fastest scan; can cause packet loss and inaccurate results                  |

> **Tip:** Use `-T0` or `-T1` to avoid IDS/IPS detection in real engagements.

### Packet Rate Control

- `--min-rate <number>` : Ensure at least this many packets/sec are sent.  
- `--max-rate <number>` : Ensure no more than this many packets/sec are sent.  

Example:  
```bash
nmap --max-rate 10 TARGET_IP
```
- limits Nmap to sending at most 10 packets per second

## Probing Parallelisation 
- `--min-parallelism <numprobes>` : Minimum number of probes to run in parallel.
- `--max-parallelism <numprobes>` : Maximum number of probes to run in parallel.

example: `nmap --min-parallelism 512 TARGET_IP`

- maintains at least 512 probes in parallel for host discovery and port scanning

<img width="353" height="164" alt="image" src="https://github.com/user-attachments/assets/32d25559-879f-4df9-bfc8-336c42844bff" />

</details>

---

## cheat sheet

| Port Scan Type   | Example Command              |
| ---------------- | ---------------------------- |
| TCP Connect Scan | `nmap -sT 10.10.117.58`      |
| TCP SYN Scan     | `sudo nmap -sS 10.10.117.58` |
| UDP Scan         | `sudo nmap -sU 10.10.117.58` |

- these scan types get us started discovering tunning TCP and UDP services on our target host

| Option                  | Purpose                                       |
| ----------------------- | --------------------------------------------- |
| `-p-`                   | Scan all ports                                |
| `-p1-1023`              | Scan ports 1 to 1023                          |
| `-F`                    | Scan 100 most common ports                    |
| `-r`                    | Scan ports in consecutive order               |
| `-T<0-5>`               | Timing template; `-T0` slowest, `-T5` fastest |
| `--max-rate 50`         | Limit rate to ≤ 50 packets/sec                |
| `--min-rate 15`         | Ensure rate ≥ 15 packets/sec                  |
| `--min-parallelism 100` | At least 100 probes in parallel               |
