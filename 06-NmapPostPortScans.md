---

# Nmap Post Port Scans- Try Hack Me room

---

## Objective 
- focus on these steps and how to execute them after port scan
5. Detect versions
6. Detect OS
7. Traceroute
8. Scripts
9. Write output

---
<details>
<summary>Service Detection</summary>

- once Nmap discovers open ports we can probe the available port to detect the running service
- further investigations of open ports is an essential piece of info as pentester cab use it to learn if there are any known vulnerabilities of the service


- adding `-sV` to Nmap comand collects and determine service and version info for the open ports
- can control intensity w/ `--version-intensity LEVEL` where level ranges between 0 (lightest) and 9 (most complete)
- `-sV --version-light` has intensity of 2
- `-sV --version-all` has intensity of 9

- important to note using `-sV` will force Nmap to proceed w/ the TCP 3-way handshake and establish the connection
- connection establishment is necessary b/c Nmap cannot discover the version w/o establishing a connection fully and communicating w/ the listening service
- a.k.a stealth STN scan `-sS` is not possible when `-sV` option is chosen

- the console output below shows simple Nmap stealth SYN scan w/ `-sV` option
- adding the `-sV` option leads to a new column in the output showing the version for each detected service
- in the case of TCP port 22 being open instead of `22/tcp open ssh` we obtain `22/tcp open ssh OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)`
- the SSH protocol guessed asthe service b/c TCP port 22 is open
- Nmap did not need to connect to port 22 to confirm
- but `-sV` required connecting to this open port to grab the service banner and any version info it can get like `nginx 1.6.2`
- so unlike service column, version column is not a guess

<img width="365" height="182" alt="image" src="https://github.com/user-attachments/assets/df6eb6ad-96e2-4247-af0b-c7d451e7d5b2" />

<img width="340" height="224" alt="image" src="https://github.com/user-attachments/assets/4f2703d7-d915-4f0d-b57a-d4341b104ef1" />

- many Nmao options require root priviledges
- unless you are running Nmap as root you need sudo like in examples above

---

<img width="866" height="251" alt="image" src="https://github.com/user-attachments/assets/b962b830-330a-4323-8068-32b65841b8cc" />


</details>


<details>
<summary>OS Detection and Traceroute</summary>

## OS Detection 
- Nmap can detect Operating Systems (OS) based on its behaviour and any telltale signs in its responses
- OS detection `-O` option

- run `nmap -sS -O 10.10.212.130`
  
<img width="380" height="268" alt="image" src="https://github.com/user-attachments/assets/d9ea2dd9-ce8b-420f-bf89-2fa7031f094a" />


<img width="344" height="224" alt="image" src="https://github.com/user-attachments/assets/d300f94a-8a6d-4a69-804b-cb4f2ee971f7" />

- Nmap detected the OS as Linux 3.X then guessed it's further that it was running kernel 3.13
- in another case we scanned Fedora Linux system w/ kernel 5.13.14
- but Nmap detected it as Linux 2.6.X
- good news is Nmap detected the OS correctly
- not-so-good news kernel version was wrong
- OS detection is very convenient but many factors might affect its accuracy

- first Nmap needs to find at least one open and one closed port on the target to make a reliable guess
- but the guest OS fingerprints might get distorted due to the rising use of virtualisation and similar technologies
- so always take OS version with a grain of salt 

## Traceroute 

-  want Nmap to find the routers between us and the target add `--traceroute`
-  executed `nmap -sS --traceroute 10.10.212.130`

<img width="386" height="245" alt="image" src="https://github.com/user-attachments/assets/497fa1a7-c7f0-4836-8893-284c9c0d8b6a" />


- above example Nmap appended a traceroute to its scan results
- Nmap's traceroute works slightly different than the `traceroute` command found on Linux and macOS or `tracert` found on MS Windows
- standard `traceroute` starts w/ a packet of low TTL (time to live) and keeps increasing until it reaches the target
- Nmap's traceroute starts w/ a packet of high TTL and keeps decreasing it

- many routers are configured not to send ICMP Time-to-Live exceeded which would prevent us from discovering their IP addresses 

---

<img width="485" height="70" alt="image" src="https://github.com/user-attachments/assets/642f0f4d-59fa-4f0d-adf9-51180b5303ce" />

</details>


<details>
<summary>Nmap Scripting Engine (NSE)</summary>

- script = piece of code that does not need to be compiled
- a.k.a it remains in its original human-readable form and does not need to be converted to machine language
- many prgrams provide additional functionality via scripts; moreover, scripts make it possible to add custom functionality that fif not exist via the built-in commands
- Nmap provides support for scripts using Lua language
- Nmap Scripting Engine (NSE) has Lua interpreter that allows Nmap to execute Nmap scripts written in Lua Language
- but don't need to learn Lua to make use of Nmap scripts

---
- our Nmap default installation can easily contain close to 600 scripts
- on attackbox check files '/usr/share/nmap/scripts`
- notice that there are hundreds of scripts conveniently named starting w/ protocol they target
- listed all the scripts starting w/ HTTP on attackbox in the console output below
- found around 130 scripts starting with http
- w/ future updates can only expect the number of installed scripts to increase

<img width="345" height="410" alt="image" src="https://github.com/user-attachments/assets/f4c6d5a6-ea88-463d-992b-50f96ec8732d" />

- can specify to use nay pt a grpup of these installed scripts
- can install other user's scripts and use them for our scans
- can tun scripts in default category `--script=default` or just add `-sC`
- in addition to defualt categories include:

| Script Category | Description                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| auth            | Authentication related scripts                                              |
| broadcast       | Discover hosts by sending broadcast messages                                |
| brute           | Performs brute-force password auditing against logins                       |
| default         | Default scripts, same as -sC                                                |
| discovery       | Retrieve accessible information, such as database tables and DNS names      |
| dos             | Detects servers vulnerable to Denial of Service (DoS)                       |
| exploit         | Attempts to exploit various vulnerable services                             |
| external        | Checks using a third-party service, such as Geoplugin and Virustotal        |
| fuzzer          | Launch fuzzing attacks                                                      |
| intrusive       | Intrusive scripts such as brute-force attacks and exploitation              |
| malware         | Scans for backdoors                                                         |
| safe            | Safe scripts that won’t crash the target                                    |
| version         | Retrieve service versions                                                   |
| vuln            | Checks for vulnerabilities or exploit vulnerable services                  |

- Some scripts belong to more than one category
- some scripts launch brute-force attacks against services
- others launch DoS attacks and exploit systems
- so crucial to be careful selecting scripts to run - don’t want to crash services or exploit them

- `sudo nmap -sS -sC 10.10.111.99`
- `-sC` ensure that Nmap will execute the default scripts following the SYN scan
- SSH service at port 22; Nmap recovered all four public keys related to the running server
- HTTP service at port 80; Nmap retrieved the default page title, page has been left as default

<img width="345" height="371" alt="image" src="https://github.com/user-attachments/assets/7e3c0c17-b4de-4f13-a471-5bf8b907e095" />

-  specify the script `--script "SCRIPT-NAME"`
-  or a pattern `--script "ftp*"` whic includes `ftp-brute` - “Performs brute force password auditing against FTP servers.”
-  be careful some scripts are pretty intrusive
- some scripts might be for a specific server, chosen at random = no benefit
- ensure you are authorised to launch such tests on the target server

- consider `http-date`
- would retrieve the http server date and time
- “Gets the date from HTTP-like services. Also, it prints how much the date differs from local time…”
- execute `sudo nmap -sS -n --script "http-date" 10.10.111.99`

<img width="313" height="213" alt="image" src="https://github.com/user-attachments/assets/efe914ff-615d-4562-9736-c4bbf22a522f" />

- might expand the functionality of Nmap beyond the official Nmap scripts
- can write your script or download Nmap scripts from the Internet
- Downloading and using a Nmap script from the Internet = certain level of risk
- good idea not to run a script from an author you don’t trust

---

<img width="611" height="272" alt="image" src="https://github.com/user-attachments/assets/fcdf77ea-5915-420c-9832-360ec7166880" />

</details>


<details>
<summary>Saving the Output(NSE)</summary>

- whenever running an Nmap scan reasonable to save results in a file
- selecting and adopting a good naming convention is crucial
- number of files can grow and hinder ability to find previous scan reuslt
- three main formats are:
1. normal
2. grepable
3. XML

**Normal**
- similar to the output we get on screen when scanning target
- `-oN FILENAME`
- N stands for normal
- example result:

<img width="338" height="242" alt="image" src="https://github.com/user-attachments/assets/7ab8b344-259f-439f-8104-399ebb43929a" />

 **Grepable**
- `grep`
- Global Regular Expression Printer
- makes filtering the scan output for specific keywords or terms efficient
- can have scan result in grepable format `-oG FILENAME`
- normal output = 21 lines but grepable = 4
- each line meaningful and complete when the user applies grep
- lines are long not conveninient to read compared to normal

<img width="487" height="92" alt="image" src="https://github.com/user-attachments/assets/f0faefac-2978-4bfd-9dfb-b582096cfe43" />

- grep filters Nmap output for a specific keyword.
- Using grep on normal output:
      - Shows service info (e.g., 80/tcp open http nginx 1.6.2)
      - Does not include host IP
      - Less convenient for scanning multiple hosts
- Using grep on grepable output:
     - Each line includes the host IP
     - Easier to process and analyze results
 
<img width="487" height="128" alt="image" src="https://github.com/user-attachments/assets/dd91940f-821d-4534-8d71-bd209648cccb" />

**XML Format**
- XML format:
    - Save Nmap scan results using -oX FILENAME
    - Convenient for processing in other programs
- All formats at once:
    - Use -oA FILENAME
    - Combines:
    - -oN → Normal output
    - -oG → Grepable output
    - -oX → XML output

**Script Kiddie format**
- Not useful for searching keywords or saving results for later
- Command example: nmap -sS 127.0.0.1 -oS FILENAME
- Displays output in a “leet” style, e.g., 31337, mainly for fun or showing off to non-technical people

<img width="406" height="209" alt="image" src="https://github.com/user-attachments/assets/0c59a792-03bc-4ba4-967f-e5b0d6e8915a" />

--- 

<img width="843" height="193" alt="image" src="https://github.com/user-attachments/assets/1db27ba9-cad1-4596-acda-cadca4ffc749" />

</details>

---

## Summary
- learned how to detect running services and their versions along w/ host operating system
- learned how to enable traceroute
- covered selecting one or more scripts to aid in pen testing
- covered the different formats to save the scan results for future reference 

--- 

| Option                     | Meaning                                         |
|-----------------------------|------------------------------------------------|
| -sV                         | Determine service/version info on open ports  |
| -sV --version-light         | Try the most likely probes (2)               |
| -sV --version-all           | Try all available probes (9)                 |
| -O                          | Detect OS                                     |
| --traceroute                | Run traceroute to target                      |
| --script=SCRIPTS            | Nmap scripts to run                           |
| -sC or --script=default     | Run default scripts                           |
| -A                          | Equivalent to -sV -O -sC --traceroute        |
| -oN                         | Save output in normal format                  |
| -oG                         | Save output in grepable format                |
| -oX                         | Save output in XML format                     |
| -oA                         | Save output in normal, XML, and grepable formats |
