
---

# Nmap Advanced Port Scans - Try Hack Me room

---

## Objectives

- explain advanced types of scans and scan options
- cover following types of port scans:
1. Null Scan
2. FIN Scan
3. Xmas Scan
4. Maimon Scan
5. ACK Scan
6. Window Scan
7. Custom Scan

- Also cover:
   - Spoofing IP
   - Spoofing MAC
   - Decoy Scan
   - Fragmented Packets
   - Idle/Zombie Scan
   - discuss options and techniques to evade firewalls and IDS systems
   - Cover options to get more verbose details from Nmap

---

<details>
<summary>TCP Null Scan, FIN Scan, Xmas Scan</summary>

## Null Scan

- does not set any flag
- all 6 flag bits set to 0
- can choose null scan using `-sN` option
- TCP packet w/ no flags set trigger no response when it reaches open port
- so from Nmap perspective: a lack of reply in a null scan = port is open ot firewall is blocking packet

<img width="862" height="220" alt="image" src="https://github.com/user-attachments/assets/bd7d7737-1bc8-4658-9015-a2c93d4db942" />

- but we expect target to respond w/ RST packet if port closed
- can use lack of RST response to figure out if ports open or filtered

<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/a7992ec2-83c6-4029-b64a-269d439ca9c3" />

- example of null scan against Linux server:

<img width="368" height="226" alt="image" src="https://github.com/user-attachments/assets/79e99ea5-577a-478d-831e-86845ddffc00" />

- null scan above has identified 6 open ports on target system
- not certain as the lack of response could be due to firewall
- Nmap options require root privileges or sudo

## FIN Scan 

- Sends a TCP packet w/ FIN flag set
- `-sF` option
- no response sent if TCP port open
- Nmap cannot be sure if open or blocked by firewall

<img width="862" height="220" alt="image" src="https://github.com/user-attachments/assets/90281ede-312c-474b-9a64-200fa20531a0" />

- target system should respond w/ RST if port closed
- some firewalls will 'silently' drop the traffic w/o sending an RST


<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/1afa80fa-3967-438c-afde-b77e72d03619" />

- example of FIN Scan against Linux server 

<img width="354" height="238" alt="image" src="https://github.com/user-attachments/assets/af9aec21-e36c-446c-8ad5-2b7236ffc3e8" />

## Xmas Scan

- sets the FIN, PSH and URG flags simultaneously
- use option `-sX`
- if RST packet received port is closed
- otherwise reported open|filtered

<img width="862" height="220" alt="image" src="https://github.com/user-attachments/assets/bb3de8e3-1969-4135-9a10-3c08ebbb11cd" />


<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/e07f4323-e782-4bcb-ac0d-7621fe03d437" />

- example of Xmas scan against Linux

<img width="386" height="239" alt="image" src="https://github.com/user-attachments/assets/283453d2-d4f4-4ea8-a9ad-ef89941b3adf" />


- one scenario where these 3 scan types useful = scanning a target behind a stateless (non-stateful) firewall
- stateless firewall checks if incoming packet has SYN flag set to detect a connection attempt
- using flag combo that doesnt match SYN packet = possible to deceive firewall and reach system behind it
- but a stateful firewall will practically block all such crafted packets = this kind of scan useless

---

<img width="835" height="269" alt="image" src="https://github.com/user-attachments/assets/77218ba8-edf3-47bb-8c17-ef8f23029085" />

</details>


<details>
<summary>TCP Maimon Scan</summary>

- FIN ACK bits are set
- target should send RST packet in response
- but certain BSD-derived systems drop the packet if it's open port = exposing open port
- scan doesn't work on most targets encountered in modern networks
- option `-sM`

- most target systems respond w/ RST packet regardless if TCP port open

<img width="862" height="280" alt="image" src="https://github.com/user-attachments/assets/ce018d22-e2cf-407d-b36c-e632ec175a0f" />

- example of TCP Maimon scan against Linux

<img width="389" height="113" alt="image" src="https://github.com/user-attachments/assets/ab6dd6fd-9f40-4a0e-928a-d17d5adb50a7" />

- as open and closed ports behave the same way Maimon scan discovers no open ports
- still scan may come in handy

--- 

<img width="401" height="68" alt="image" src="https://github.com/user-attachments/assets/92fb0675-aca8-4a56-a62b-e801c284addb" />

</details>


<details>
<summary>TCP ACK, Window and Custom Scan</summary>

## TCP ACK Scan 

- ACK scan will send TCP packet w/ ACK flag set
- option `-sA`
- target responds to ACK w/ RST regardless of state of port
- happens b/c TCP paket w/ ACK flag set should be sent only in response to received TCP packet to acknowledge the receipt of some data (unlike our case)
- so this scan won't tell if a port is open in a simple setup

<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/9a03068d-3105-4dc1-85ff-b414b6279328" />

- in below example we scan target VM before installing firewall on it
- as expected we did not learn which ports are open

<img width="375" height="125" alt="image" src="https://github.com/user-attachments/assets/09c9c21f-6ccb-47ac-a30c-ad6bbf039deb" />

- this scan helpful = if firewall infront of target
- based on which ACK packets resulted in responses = learn which ports are not blocked by firewall
- a.k.a scan is more suitable to discover firewall rules set and configuration 

<img width="364" height="191" alt="image" src="https://github.com/user-attachments/assets/1c606ee4-f40f-49a9-b227-7ff773ead88f" />

- above output is repeated ACK scan but this time target VM 10.10.173.34 has firewall
- shows 3 ports aren't blocked by firewall
- results indicate firewall is blocking all other ports except for these 3


## Window Scan 

- TCP Window Scan similar to ACK scan but examines the TCP window field of the RST packets returned 
- on certain systems this can reveal that the port is open
- option `-sW`
- we expect RST packet to reply to our 'uninvited' ACK packets regardless port is open or closed

<img width="862" height="260" alt="image" src="https://github.com/user-attachments/assets/961d290e-f73c-4e90-97b7-ba3ff8fd576c" />

- example of TCP window scan against Linux system with no firewall = provide not much info

<img width="341" height="128" alt="image" src="https://github.com/user-attachments/assets/f9d511a0-33d4-435a-ba91-e64f07e36ba9" />

- if scan repeated against server behind firewall = better results

<img width="326" height="200" alt="image" src="https://github.com/user-attachments/assets/70891abd-be04-485d-b9ef-e2c8f29cc823" />

- TCP window scan output 3 ports detected closed  (in contrast ACK scan labelled same three as unfiltered)
- we know these 3 ports are closed but realise they responded differently = firewall doesn't block them

## Custom Scan 

- experiment w/ new TCP flag combo beyond built in TCP scan types
- option `--scanflags`
- e.g. set SYN, RST and FIN simultaneously = `--scanflags RSTSYNFIN`
- if you develop custom scan must kbow how diff ports will behave to interpret results in scenarios correctly

<img width="862" height="262" alt="image" src="https://github.com/user-attachments/assets/b86ff0fb-1998-4026-8eb8-f57b1819eba8" />

---
Note: 

ACK scan and window scan were efficient in helping us map out firewall rules but remember that just b/c firewall is not blocking specific port doesn't mean a service is listening on that report

example:

possibility that the firewall rules need updating to reflect recent service changes = ACK and window scans are exposing the firewall rules not the services 

---

<img width="745" height="346" alt="image" src="https://github.com/user-attachments/assets/63ed2caa-9af5-434d-9395-5d98484466b1" />

</details>


<details>
<summary>Spoofing and Decoys</summary>

- in some network setups you can scan a target system using a spoofed IP address and even a spoofed MAC address
- scan is beneficial = in situation where you can guarantee to capture the response
- if we try to scan a target from random network using spoofed IP address = likely won't have any response routed to us and scan results can be unreliable


- following figure shows attacker launching comman 'nmap -S SPOOFED_IP 10.10.34.98`

<img width="882" height="722" alt="image" src="https://github.com/user-attachments/assets/41853b9c-3c93-4ff7-ab90-546b10104bf9" />

- Nmap will craft all the packets uing provided source IP address SPOOFED_IP
- target machine responds to incoming packets sending the replies to destination IP address SPOOFED_IP
- for scan to work and give accurate results attacker needs to monitor the network traffic to analyse replies

## In brief, scanning with spoofed IP address is 3 steps 

1. attacker sends a packet w/ a spoofed source IP address to target machine
2. target machine replies to the spoofed IP address as the destination
3. attacker captures the replies to figure put open ports

---

- specify the network w/ `-e`
- explicitly disable ping scan `-Pn`
- so instead of `nmap -S SPOOFED_IP 10.10.34.98`  need to issue `nmap -e NET_INTERFACE -Pn -S SPOOFED_IP 10.10.34.98` to tell Nmap explicitly which network interface to use and not to expect to receive a ping reply
- scan is useless if attacker system cannot monitor network for responses

---

- when on the same subnet as target machine = would be able to spoof our MAC address too
- can specify source MAC `--spoof-mac SPOOFED_MAC`
- this address spoofing only possible if attacker and target on same ethernet (802.3) network or WiFi (802.11)

---

- spoofing only works in minimal cases where certain conditions are met
- so attacker might resort to using decoys to make it more challenging to be pinpointed
- concept is simple = make scan appear to be coming from many IP addresses so the attacker's IP address is lost among them

<img width="882" height="822" alt="image" src="https://github.com/user-attachments/assets/6a13c938-64a6-4b14-bf7a-75e420132bfb" />

- figure above =  scan appear to be coming from 3 IP addresses, the replies will go to the decoy as well

---

<img width="493" height="132" alt="image" src="https://github.com/user-attachments/assets/a005b7f2-4640-4415-8cdc-00aa9c58e9d9" />

</details>


<details>
<summary>Fragmented Packets</summary>

## Firewall

- a piece of software or hardware that permits to packets to pass through or blocks them
- functions based on firewall rules
- summarised as blocking all traffic with exceptions or allowing in all traffic with exceptions
- example:
    - might block all traffic to our server except those coming to our web server
- traditional firewalls inspects at least IP header and the transport layer header
- more sophisticated firewall also try to examine the data carried by transport layer

  ## IDS

  - intrusion detection system
  - inspects network packets for select behavioural patterns or specified content signatures
  - raises an alert whenebr malicious rule is met
  - in addition to IP header and transport layer IDS inspect data contents in transport layer and check it is matches malicious patterns
  
  **how can we make it less likely for traditional firewall/IDS to detect Nmap activity?**
  not easy to answer, but depending on type of firewall/IDS we might benefit from dividing packet into smaller packets

## Fragmented Packets 

- nmap provide `-f` option to fragment packets
- once chosen IP data is divided into 8 bytes or less
- adding another `-f` (`-f -f` or `-ff`) splits data into 16 bytes
- can change default using `--mtu` = should always choose a multiple of 8

<img width="1282" height="928" alt="image" src="https://github.com/user-attachments/assets/fd6cb9a1-3ea4-424f-838e-31cfbcc6a4a7" />

- look at IP header above to properly understand fragmentation
- source address taking 32 bits (4 bytes) on the fourth row
- destination address takes another 4 bytes on fifth row
- data we will fragment across multiple packets highlighted red
- to aid in reassembly on recipient side IP uses identification (ID) and fragment offset shown on second row

---

**compare running `sudo nmap -sS -p80 10.20.30.144` and `sudo nmap -sS -p80 -f 10.20.30.144`**

- this will use stealth TCP SYN scan on port 80 but in second command we are requesting Nmap fragment the IP packets

- in first 2 lines we see ARP query and response
- Nmap issued an ARP query b/c target is on same ethernet
- second 2 lines show TCP SYN ping and reply
- fifth line is beginning of port scan: Nmao sends a TCP SYN packet to port 80
- this case the IP header is 20 bytes and TCP header is 24 bytes
- the minimum size of TCP header is 20 byte

  <img width="915" height="177" alt="image" src="https://github.com/user-attachments/assets/e6371c9f-36f9-4cf7-830f-65a02cf8c79f" />

- with fragmented request using `-f` the 24 bytes of TCP header will be divided into multiples of 8
- the lsat fragment containing 8 bytes or less of TCP header
- since 24 is divisible by 8 we have 3 IP fragments each w/ 20 bytes of IP header and 8 bytes of TCP header
- can see the 3 fragments between 5th and 7th line

<img width="915" height="149" alt="image" src="https://github.com/user-attachments/assets/3817fedc-d751-4d7c-b61d-a7298b38d01b" />

- if we added `-ff` the fragmentation of data will be multiples of 16 a.k.a the 24 bytes of TCP header would be divided over 2 IP fragments
- first containing 16 bytes and second containing 8 bytes of the TCP header
- if prefer to increase size of packets to make them look innocuous = `--data-length NUM` Num = number of bytes we want to append to our packets

 </details>


<details>
<summary>Idle/Zombie Scan</summary>

- spoofing IP address =  can be great approach for scanning stealthily
- but spoofing only work in specific network setups = requires us to be in position where you can monitor traffic
- can upgrade w/ idle scan

- idle/zombie scan requires an idle system connected to the network that you can communicate with
- Nmap will make each probe appear as if it's coming from idle (zombie) host
- then it checks for indicators whether the idle host received any response to the spoofed probe
- accomplished by checking the IP identification (IP ID) value in the IP header
- can run idle scan w/ `nmap -sI ZOMBIE_MACHINE MACHINE_IP` where `ZOMBIE_ID` is IP addess of idle host

**Idle scan requires following 3 steps to discover whether port is open**
1. Trigger idle host to respond so you can record current IP ID on idle host
2. Send SYN packet to a TCP port on the target, packet should be spoofed to appear as if it's coming from the idle host IP address
3. trigger the idle machine again to respond so we can compare the new IP ID  w/ one received earlier

- figure below we have attacker system porbing idle machine, a multi function printer
- by sending a SYN/ACK it responds w/ RST pscket containing its newly incremented IP ID

<img width="882" height="546" alt="image" src="https://github.com/user-attachments/assets/5118afad-b252-41b4-bc4a-1a07d4568f77" />

- attacker sends a SYN packet to TCP port they want top check on the target machine in the next step
- but this packet will use idle host IP address as the source
- 3 scenarios would arise

**Scenario 1**
- shown in figure below TCP port is closed s
- so the target machine responds to the idle host w/ RST packet
- idle host does not respond so it's IP ID is not incremented

<img width="882" height="562" alt="image" src="https://github.com/user-attachments/assets/da74b775-0822-456e-a2a7-376ed4ffc8e2" />

**Scenario 2**
- TCP port is open so the target machine responds w/ a SYN/ACK to the idle host
- idle host responds to this unexpected packet w/ an RST packet
- thus incrementing it's IP ID

<img width="882" height="562" alt="image" src="https://github.com/user-attachments/assets/254c7a64-49a1-459f-928e-77201f506b39" />

**Scenario 3**
- target machine does not respond at all due to firewall rules
- lack of repsonse leads to same result as closed response = idle host won't increase the IP ID
- the attacker sends another SYN/ACK to the idle host
- idle host responds w/ RST packet, incrementing the IP ID by one again
- attacker needs to compare the IP ID of the RST packet received in the first step w/ the IP ID of the RST packet received in this third step
- if the difference is 1 it means the port on the target machine was closed or filtered
- but the difference is 2 = means the port scan on the target was open
- worth repeating that this scan is called an idle scan b/c choosing an idle host is indispensable for the accuracy of the scan
- if the 'idle host' is busy all the returned IP IDs would be useless

--- 

<img width="541" height="73" alt="image" src="https://github.com/user-attachments/assets/b057aab5-854d-4005-a495-e3a587fd3129" />

</details>


<details>
<summary>Getting More Details</summary>

- may consider adding `--reason` if want Nmap to provide more details regarding its reasoning and conclusions

1. without '--reason'

<img width="342" height="196" alt="image" src="https://github.com/user-attachments/assets/c746bbe2-6f9e-4460-92eb-b0d61b4cb258" />

2. with `--reason`nmap

<img width="332" height="191" alt="image" src="https://github.com/user-attachments/assets/3757dc6f-5146-404e-8423-010f60793a3e" />

- providing `--reason` flag gives us the explicit reason why Nmap concluded that the system is up or a particular port is open
- we can see this system is considered online b/c Nmap 'received arp-response'
- on the other hand we know the SSH port is deemed to be open b/c Nmap received a 'syn-ack' packet back

- for more detailed output can use `-v` for verbose output or `-vv` for even more verbosity

<img width="341" height="238" alt="image" src="https://github.com/user-attachments/assets/c5a516cf-cf2f-4145-b73f-a5e6e4a9a04e" />

<img width="339" height="237" alt="image" src="https://github.com/user-attachments/assets/d1efaa7d-37bb-4bd9-a4d1-b25573ecbe46" />

- if `-vv` does not satisfy your curiosity
- `-d` for debugging details
- `-dd` for even more details
- `-d` will create an output that extends beyond a single screen

---

<img width="855" height="252" alt="image" src="https://github.com/user-attachments/assets/d9054ae2-79a2-4c7d-aee2-2f7238660596" />

 </details>

---

## cheat sheet 

| Port Scan Type            | Example Command                                           | Purpose                                                                 |
| ------------------------- | --------------------------------------------------------- | ----------------------------------------------------------------------- |
| TCP Null Scan             | `sudo nmap -sN 10.10.37.190`                             | Sends TCP packets with no flags set; used for stealth scanning.         |
| TCP FIN Scan              | `sudo nmap -sF 10.10.37.190`                             | Sends TCP FIN packets; bypasses some firewall rules.                    |
| TCP Xmas Scan             | `sudo nmap -sX 10.10.37.190`                             | Sends FIN, PSH, URG flags; detects open ports stealthily.              |
| TCP Maimon Scan           | `sudo nmap -sM 10.10.37.190`                             | Uses FIN/ACK combination; less common stealth scan.                     |
| TCP ACK Scan              | `sudo nmap -sA 10.10.37.190`                             | Determines firewall rules and port states.                              |
| TCP Window Scan           | `sudo nmap -sW 10.10.37.190`                             | Checks TCP window size to detect open ports.                            |
| Custom TCP Scan           | `sudo nmap --scanflags URGACKPSHRSTSYNFIN 10.10.37.190`  | Sends custom TCP flags for advanced scanning techniques.                |
| Spoofed Source IP         | `sudo nmap -S SPOOFED_IP 10.10.37.190`                  | Sends packets using a fake source IP to hide your identity.            |
| Spoofed MAC Address       | `--spoof-mac SPOOFED_MAC`                                | Changes the MAC address for stealth or bypassing MAC-based filters.     |
| Decoy Scan                | `nmap -D DECOY_IP,ME 10.10.37.190`                       | Uses decoy IPs to confuse intrusion detection systems.                  |
| Idle (Zombie) Scan        | `sudo nmap -sI ZOMBIE_IP 10.10.37.190`                  | Uses a third-party host (zombie) to scan targets stealthily.           |
| Fragment IP data into 8B  | `-f`                                                      | Splits packets into 8-byte fragments to evade detection.                |
| Fragment IP data into 16B | `-ff`                                                     | Splits packets into 16-byte fragments for stealth scanning.             |


| Option             | Purpose                                 |
| ------------------ | --------------------------------------- |
| --source-port PORT_NUM | Specify source port number             |
| --data-length NUM     | Append random data to reach given length |

- these scan types rely on setting TCP flags in unexpected ways to prompt ports for a reply
- Null, FIN and Xmas scan provoke a response from closed ports
- Maimon, ACK, and window scans provoke a response from open and closed ports

| Option  | Purpose                                 |
| ------- | --------------------------------------- |
| --reason | Explains how Nmap made its conclusion  |
| -v       | Verbose                                |
| -vv      | Very verbose                            |
| -d       | Debugging                               |
| -dd      | More details for debugging              |

## Reflection
This room has significantly strengthened my network reconnaissance skills. I now understand the variety of scanning methods Nmap provides, when to use them, and how to interpret the results. I also appreciate the importance of stealth, timing, and proper target selection to avoid unnecessary noise or detection. These skills are foundational for ethical penetration testing and network security assessments.
