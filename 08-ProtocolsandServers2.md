---

# Protocols and Servers 2 - Try Hack Me room

---

## Objective
- servers implementing protocols are subject to different kinds of attacks
1. Sniffing Attack (Network Packet Capture)
2. Man-in-the-Middle (MITM) Attack
3. Password Attack (Authentication Attack)
4. Vulnerabilities

---

## Introduction 

**Security Triad (CIA)**
- Confidentiality: ensure data is accessible only to authorised parties
- Integrity: ensure data is accurate, consistent and complete during transit or storage
- Availability: ensure servies and data are accessible when needed

**Priority examples:**
- Intelligence agencies: Highest emphasis on confidentiality
- Online Banking: Highest emphasis on integrity of transactions
- Ad-driven platforms: highest emphasis on availability

**Corresponding attach types**
- Disclosure: threat to confidentiality
- Alteration: threat to integrity
- destruction: threat to availability 

<img width="1302" height="442" alt="image" src="https://github.com/user-attachments/assets/1d6c97e6-4aad-4c67-916d-f32841f1d52b" />

**Attack and Security Impact:**
- network packet capture: violates confidentiality -> leads to disclosure
- password attacks: can result in disclosure of sensitive info
- man-in-the-middle (MITM) attacks: breaks integrity -> can alter communication data

- focus in this room: disclosure, alteration and integrity-related attacks relevant to protocol design and server implementation

**Vulnerabilities**
- Broader spectrum: different vulnerabilities have different impacts
- Denial of Service (DoS): affects availability
- Remote Code Execution (RCE): can cause server damage
- vulnerabilities create risk only when exploited

## Room focus 
- protecting confidentiality and integrity of transmitted data
- protocol upgrades or replacements to prevent disclosure and alteration
- recommendation to explore other modules for additional security topics

## Tools introduced
- Hydra: used to find weak passwords

---

<details>
<summary>Sniffing Attack</summary>

**Sniffing attack:**

   - captures network packets to collect info from target
   - effective when protocols communicate in cleartext -> exposes private messages and login credentials
    
- Requirements for Sniffing:
    - Ethernet (802.3) network card
    - proper permissions: root on Linux, administrator on Windows
- Common tools:
    - Tcpdump: Free CLI tool, cross platform
    - Wireshark: Free GUI tool, cross-platform
    - Tshark: CLI alternative to wireshark
   - specialised tools exist for capturing passwords/messages; Tcpdump and wireshark can achieve similar results with effort

-  Example: POP3 Email Sniffing:
   -  Command: `sudo tcpdump port 110 -A`
       - `sudo`: required for root privileges
       - `port 110` filters traffic to POP3 server
       - `-A` displays packet contents in ASCII
 - Requires access to network traffic via wiretap, port mirroring or a MITM attack 

<img width="380" height="421" alt="image" src="https://github.com/user-attachments/assets/d6b6afb6-4de6-4ad6-9ea5-e07bfce3910f" />

Filtering Relevant Packets

- unimportant packets removed to focus on critical data
- username and password sent in seperate packets
      - Example: USER frank and PASS D2xc9CgD.

using wireshark:
- filter traffic with `pop` to isolate relevant POP3 packets
- captured traffic reveals the username and password

key insights:
- both Tcpdump and Wireshark can expose sensitive credentials when data is transmitted in cleartext

<img width="911" height="665" alt="image" src="https://github.com/user-attachments/assets/c6c4b109-7f84-457b-bc3b-3ceac7863726" />

- Cleartext Protocol Vulnerability:
     - any protocol sending data in cleartext is susceptible to sniffing attacks
     - requirement: access to a system between the 2 communicating parties
- mitigation:
     - add an encryption layer to the protocol
     - common solutions:
          - TLS for HTTP, FTP, SMTP, POP3, IMAP etc
          - SSH as a secure replacement for Telnet
      
--- 

<img width="381" height="143" alt="image" src="https://github.com/user-attachments/assets/3c2a6555-3580-4228-a7c4-3755b79f6f58" />

</details>


<details>
<summary>Man-In-The-Middle (MTM) attack</summary>

- victim A thinks they are communicating with a legitimate destination B
- attacker E intercepts and modifies the communication
- example: A requests transfer of £20 to M but E changes the amount before B receives it
- impact: B acts on altered message, compromising integrity

<img width="882" height="562" alt="image" src="https://github.com/user-attachments/assets/74755f5e-0720-4119-952a-aecb66611eb3" />

**Ease of MITM attack**
- simple if parties do not verify authenticity and integrity of messages
- some protocols lack secure authentication or integrity checks

**Vulnerability Protocols**
- HTTP browsing is highly susceptible, often undetectable by user
- other cleartext protocols at risk: FTP, SMTP. POP3

**Tools for MITM**
- anytime browse over HTTP you are susceptible to MITM attack
- you cannot recognise it many tools would aid you in carrying such an attack
- examples: Ettercup, Bettercap

**Mitigation**
- use cryptography: authentication, encryption, or signing of messages
- TLS and PKI and trusted root certifications protects against MITM attacks

---

<img width="253" height="115" alt="image" src="https://github.com/user-attachments/assets/21da8116-9eb1-48a7-89a4-dc0e8697e7ec" />

</details>


<details>
<summary>Transport Layer Security (TLS)</summary>

- Goal:
     - defend against password sniffing and Man-In-The-Middle (MITM) attacks
     - ensure confidentiality (privacy of data) and integrity (unaltered communication)
 
- SSL/TLS History:
     - SSL (Secure Sockets Layer): introduced by Netscape in 1994
        - SSL 3.0 released in 1996.
     - TLS (Transport Layer Security): introduced in 1999 as a more secure successor
     - today, TLS is standard solution for securing communications
      
- Problem with cleartext protocols:

   - Protocols like HTTP, FTP, POP3, SMTP transmit data in cleartext  
   - Makes traffic easy to capture, save, and analyze by attackers

- OSI Model Placement:
  
     - application-layer protocols are vulnerable by default
     - by adding encryption at the presentation layer, data is covered into chipertext before transmission

- Pratical impact:
  
   - SSL/TLS provides an encryption at the presentation layer on top of existing protocols
   - Examples: HTTPS, FTPS, SMTPS, IMAPS
   - Ensures authentication, integrity, and confidentiality of communications

<img width="802" height="642" alt="image" src="https://github.com/user-attachments/assets/f65f0569-2095-4b51-b4d1-c42c4bdc247b" />


**SSL vs TLS and Upgrading Cleartext Protocols**

- SSL vs TLS:
    - TLS is successor to SSL and is more secure
    - TLS has pratically replaced SSL in modern systems
    - the term SSL is still widely used, so both are often written as SSL/TLS to avoid confusion
    - expect all modern servers to be runnin TLS not SSL

 - Upgrading Cleartext Protocols:
    - SSL/TLS can add encryption to existing cleartext protocols
    - examples of upgraded protocols
        - HTTP -> HTTPS
        - FTP -> FTPS
        - SMTP -> SMTPS
        - POP3 -> POP3S
        - IMAP -> IMAPS

- each secure protocol uses a different default port than its cleartext version
- (detailed table shows cleartext vs SSL/TLS port mappings; purpose = demonstrate how encryption is applied at the protocol level)

| Protocol | Default Port | Secured Protocol | Default Port with TLS |
| -------- | ------------ | ---------------- | --------------------- |
| HTTP     | 80           | HTTPS            | 443                   |
| FTP      | 21           | FTPS             | 990                   |
| SMTP     | 25           | SMTPS            | 465                   |
| POP3     | 110          | POP3S            | 995                   |
| IMAP     | 143          | IMAPS            | 993                   |

--- 

**HTTP vs HTTPS Connection Steps**

- HTTP(cleartext):
1. establish a TCP connection with the web server
2. send HTTP requests (e.g. GET, POST)

- HTTP (Encrypted):
1. establish a TCP connection
2. establish an SSL/TLS connection (extra encryption step)
3. send HTTP requests securely (encrypted)

- Key difference:
    - HTTPS adds the SSL/TLS handshake before any HTTP data is exchanged
    - ensures confidentiality, integrity and authentication

---

1. ClientHello
     - Client proposes:
        - supported protocol version (TLS version)
        - supported cipher suites (encryption algorithms)
        - random number (for key generation)

2. ServerHello
      - server responds with:
          - chosen protocol version
          - selected cipher suite
          - server random number
      - sends digital certificate (includes public key, signed by a trusted CA)
  
3. Authentication and Key Exchange
- client verifies the server's certificate using PJU and trusted root CAs
- key exchange happens (depending on chosen method):
     - RSA, Diffie-Hellman, ECDHE etc.
- both client and server derive a shared session key

4. Session key Establishment
- client and server generate symmetric keys (using exchanged random numbers + key exchange data)
- future communication will use fast symmetric encryption

5. handshake completion
- both sides send finished message encrypted with session key
- secure channel is established - HTTPS traffic (HTTP requests/responses) can now begin

<img width="882" height="516" alt="image" src="https://github.com/user-attachments/assets/29b7e9e7-6884-4a30-aa74-1c8a5fdaa726" />

**Simplified SSL/TLS handshake steps**

1. ClientHello
- client indicates capabilities (supported algorithms, TLS version etc.)

2. ServerHello
   - server selects connection paramaters
   - provides its digital certificate (signed by a trusted CA) for authentication
   - may send ServerKeyExchange (extra key info)
   - ends negotiation with ServerHelloDone

3. ClientKeyExchanged
    - client sends info needed to generate the master key
    - switches to encryption and signals this using ChangeCipherSpec

4. Server ChangeCipherSpec
   - server also switches to encryption
   - both sides now share a securely generated secret session key
  
--- 

**Key outcomes**
- the client and server successfully agree on a shared secret key
- the key is generated securely, so even an attacker monitoring the channel cannot derive it
- all further communication (e.g. HTTP requests and data) is encrypted

  ---

**Importance of Certificates and PKI** 

- SSL/TLS relies on public certificates signed by trusted Certificate Authorities (CAs).
- Example: When visiting a site (e.g., TryHackMe over HTTPS), the browser checks the site’s certificate against trusted CAs.
- This ensures:
     - The client is talking to the legitimate server.
     - Prevents Man-in-the-Middle (MITM) attacks.
 
---

<img width="677" height="835" alt="image" src="https://github.com/user-attachments/assets/f7813867-9758-4fb1-974d-6aa474aaafe6" />

**SSL/TLS Certificate Information:**

- Key Details in a Certificate:
     - Issued to: Name of the company/organization using the certificate.
     - Issued by: Certificate Authority (CA) that signed and issued it.
     - Validity period: Start and end dates; expired certificates are invalid.

- Browser’s Role:
     - Browsers automatically check certificates for us.
     - They verify we are communicating with the correct server.
     - Certificates ensure the connection is secure and trusted.

--- 

<img width="375" height="61" alt="image" src="https://github.com/user-attachments/assets/f70337f7-1cd8-4e95-8bd5-7895820e43d5" />

- Purpose: Encrypts DNS queries to protect confidentiality and integrity.
- Benefit: Prevents eavesdropping and MITM attacks on DNS traffic

</details>


<details>
<summary>Secure Shell (SSH) Overview</summary>

- Purpose

  - provides a secure way to remotely administer systems
  - allows executing commands on a remote system over the network
 
- Security Guarantees (the “S” in SSH):
1. server authentication: confirm the identity of the remote server
2. confidentiality: messages are encrypted and only decryptable by the intended recipient
3. integrity: both sides can detect any message modification

- How security is achieved:

    - using cryptography, combining different encryption algorithms
    - ensures confidentiality and integrity
 
- requirements to use SSH:
    
    - SSH server: listens on port 22 by default
    - SSH client: used to connect and authenticate
 
- authentication methods:
1. username and password
2. public/ private key pair (server must recognise the public key)

- example SSH connection:
    - `ssh username@MACHINE_IP`
    - connects to the SSH server at MACHINE_IP with login username
    - prompts for password if using password authentication
    - once authenticated, grants access to the remote terminal
 
<img width="368" height="188" alt="image" src="https://github.com/user-attachments/assets/9cc67805-a0b7-42b7-8e6f-4c8e78fcac93" />

--- 

**SSH: Connection example and security considerations**

- example connection
  
  - command: `ssh mark@MACHINE_IP`
  - enter password -> gain access to the remote system's terminal
  - all commands and credentials are sent over an encrypted channel, ensuring confidentiality

- First-Time Connection Security:

  - Must confirm the SSH server’s public key fingerprint
  - Prevents Man-in-the-Middle (MITM) attacks.

- MITM Recap:

   - attacker (E) sits between client (A) and server (B)
   - E intercepts and modifies communication while both A and B think they are talking dierctly
   - in SSH since there is no third-party validation by default manual key verification is necessary for the first connection
 
- Key point:

   - SSH remains secure and reliable once the server's key is verified
 

<img width="882" height="562" alt="image" src="https://github.com/user-attachments/assets/d91f62e6-b910-44a9-aa19-78f8271a052a" />

---

**File Transfer with SSH: SCP (Secure Copy Protocol)**

- purpose:
  
    -  Transfer files securely over the network using SSH encryption.
 
- examples:
  
1. copy from remote to local:
`scp mark@MACHINE_IP:/home/mark/archive.tar.gz ~`
- Copies `archive.tar.gz` from remote `/home/mark/` to the local home directory (`~`)

2. copy from local to remote:
`scp backup.tar.bz2 mark@MACHINE_IP:/home/mark/`
- Copies `backup.tar.bz2` from local machine to remote directory `/home/mark/`

- key point

   - SCP leverages SSH encryption, ensuring confidentiality and integrity during file transfer


 <img width="358" height="64" alt="image" src="https://github.com/user-attachments/assets/236cdc95-96bb-4ad3-bafc-b63695b68141" />


---

<img width="458" height="109" alt="image" src="https://github.com/user-attachments/assets/bb282a1e-9c27-4362-b676-d168f0ce317e" />


<img width="472" height="160" alt="image" src="https://github.com/user-attachments/assets/ebc27e6e-2c91-4981-85a7-bcbc569ed7a2" />


<img width="464" height="102" alt="image" src="https://github.com/user-attachments/assets/ac1eab03-343b-4a58-9a7e-efbb1890a0cc" />


</details>


<details>
<summary>Password Attack</summary>

**Password Attacks & Authentication**

- Context:

   - Third type of attack after network packet captures and MITM attacks.
   - Focus: Password attacks on protocols that require authentication
 
- Authentication:

    - The process of proving your identity to a system.
    - Required before gaining access (e.g., POP3 mailbox)
 

- POP3 Example:

     - User frank provides a password.
     - Server verifies the password and authenticates the user.
     - Password acts as a means of authentication

- Key Point:

     - Protocols relying on passwords are vulnerable to attacks if the credentials are weak or intercepted

 
 <img width="362" height="352" alt="image" src="https://github.com/user-attachments/assets/66ff1344-f5ac-4cab-9855-169289886075" />

---

**Authentication Methods**

- Authentication = proving your identity.
- Can be based on:
1. Something you know: Passwords, PIN codes (focus of this task).
2. Something you have: SIM card, RFID card, USB dongle.
3. Something you are: Biometric data like fingerprint or iris.

- Focus for This Task:

     - Password attacks (something the target knows).
     - Protocols like Telnet, SSH, POP3, IMAP require passwords to gain access.

example:

<img width="336" height="359" alt="image" src="https://github.com/user-attachments/assets/df00f42f-3ad7-4087-92d0-aa231c10fd76" />



- Common Weak Passwords (Adobe 2013 breach):
  
1. 123456
2. 123456789
3. password
4. adobe123
5. 12345678
6. qwerty
7. 1234567
8. 111111
9. photoshop
10. 123123

- Key Insight:
  
     - Weak passwords are highly predictable and easy targets for attackers
     - most weak passwords are generic not related to specific products
     - users often underestimate predictability of simple sequences or keyboard patterns

---

1. Password Guessing:

     - Attacker guesses based on knowledge of the target (e.g., pet’s name, birth year).

2. Dictionary Attack:

     - Uses a precompiled list of words or passwords.
     - Expands on guessing by trying many valid words systematically.

3. Brute Force Attack:

     - Tries all possible character combinations.
     - Time-consuming, grows exponentially with password length and        complexity


- key insights: Weak and predictable passwords are highly vulnerable to all three attack types

--- 
## Dictionary Attacks:
- use precompiled wordlists from data breaches (e.g. RockYou)
- file location on attackbox: /usr/share/wordlists/rockyou.txt
- wordlist choice can be target-specific (like french words for a frech user)

**THC Hydra**
- tool for automated password attacks on many protocols: FTP, POP3, IMAP, SMTP, SSH, HTTP
- Basic syntax: `hydra -l username -P wordlist.txt server service`
- Options:

   - `-l username` -> target login name
   - `-P wordlist.txt` -> file with passwords to try
   - `server` -> hostname or IP of target
   - `service` -> service to attack (ftp,ssh, etc)
 
- example commands:

  - FTP Attack:
  - `hydra -l mark -P /usr/share/wordlists/rockyou.txt MACHINE_IP ftp
hydra -l mark -P /usr/share/wordlists/rockyou.txt ftp://MACHINE_IP`

      - SSH attack:
      - `hydra -l frank -P /usr/share/wordlists/rockyou.txt MACHINE_IP ssh`
   
- optional arguments:

     - `-s PORT` -> specify non-default port
     - `-V` or `-vV` -> verbose output, shows attempted username/password combos
     - `-t n` -> number of parallel connections (e.g. `-t 16`)
     - `-d` -> debuggin output, useful for troubleshooting connection issues
     - use `CTRL-C` to stop once password is found
 
  - Mitigation against password attack:
1. Password policy: human verification for GUI login pages
2. Public Certificates: authentication via certificates (e.g. SSH)
3. Two-factor Authentication (2FA): secondary code via email, app or SMS
4. Other sophisticated methods: IP-based geolocation, behavioural monitoring
5. Best practice: combine multiple mitigation approaches for strong protection

--- 

<img width="931" height="220" alt="image" src="https://github.com/user-attachments/assets/996a6064-65be-4367-9320-27f6df0aa526" />

</details>

---

## Summary 

3 common attacks are:
1. Sniffing Attack
2. MITM Attack
3. Password Attack

- for each above we focused on attack details and the mitigation steps
- it is good to remember the default port number for common protocols

---

| Protocol | TCP Port | Application(s)                 | Data Security         |
|----------|----------|--------------------------------|---------------------|
| FTP      | 21       | File Transfer                  | Cleartext           |
| FTPS     | 990      | File Transfer                  | **Encrypted**       |
| HTTP     | 80       | Worldwide Web                  | Cleartext           |
| HTTPS    | 443      | Worldwide Web                  | **Encrypted**       |
| IMAP     | 143      | Email (MDA)                    | Cleartext           |
| IMAPS    | 993      | Email (MDA)                    | **Encrypted**       |
| POP3     | 110      | Email (MDA)                    | Cleartext           |
| POP3S    | 995      | Email (MDA)                    | **Encrypted**       |
| SFTP     | 22       | File Transfer                  | **Encrypted**       |
| SSH      | 22       | Remote Access and File Transfer| **Encrypted**       |
| SMTP     | 25       | Email (MTA)                    | Cleartext           |
| SMTPS    | 465      | Email (MTA)                    | **Encrypted**       |
| Telnet   | 23       | Remote Access                  | Cleartext           |

---

- cleartext protocols -> unsafe, vulnerable to sniffing
- encrypted protocols -> safe, use TLS/SSL or SSH for confidentiality and integrity

---

- Hydra remains efficient tool

| Option       | Explanation                                             |
|--------------|---------------------------------------------------------|
| -l username  | Provide the login name                                  |
| -P WordList.txt | Specify the password list to use                      |
| server service | Set the server address and service to attack         |
| -s PORT      | Use in case of non-default service port number         |
| -V or -vV    | Show the username and password combinations being tried|
| -d           | Display debugging output if the verbose output is not helping |
