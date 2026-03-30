# My_Little_Cyber_Projects_11


EXPLOITING METASPLOITABLE 2
Metasploitable 2 is an intentionally vulnerable Ubuntu Linux virtual machine designed for security training and penetration testing practice. In this walkthrough, I will methodically exploit its weaknesses, demonstrating a common penetration testing lifecycle: Reconnaissance, Enumeration and Exploitation. My attacking platform will be Kali Linux.
 
CO CHCI DELAT?

1. Průzkum a skenování (Reconnaissance)
Identifikace cíle: Pomocí netdiscover nebo nmap zjistěte IP adresu Metasploitable stroje.
Port Scanning (Nmap): Spusťte důkladný sken portů pro zjištění běžících služeb (nmap -sV -p- <IP_cile>).
Vulnerability Scanning: Použijte nástroje jako Nessus nebo OpenVAS (pokud jsou instalovány v Kali) k automatickému vyhledání zranitelností. 


2. Exploatace (Získání přístupu)
Metasploit Framework: Toto je hlavní nástroj. Spusťte msfconsole a hledejte exploity pro verze služeb zjištěné nmapem.
FTP backdoor: Zneužijte zranitelnou službu vsftpd 2.3.4 na portu 21, která umožňuje získat shell.
Samba/SMB: Útoky na starou verzi Samby (port 139/445) pro získání root přístupu.
Webové aplikace: Metasploitable obsahuje zranitelné webové aplikace (DVWA, Mutillidae). Zkuste SQL Injection nebo Cross-Site Scripting (XSS).
Java RMI: Zneužijte port 1099 (Java RMI registry) pro vzdálené spuštění kódu. 


3. Post-Exploatace (Co dál po průniku)
Privilege Escalation: Pokud získáte přístup jako běžný uživatel (msfadmin), pokuste se získat root práva (např. přes chybně nastavené SUID soubory nebo sudo práva).
Získávání hesel: Prohledejte soubory /etc/shadow nebo použijte hashdump v Meterpreteru a pokuste se hesla prolomit pomocí John the Ripper.
Pivoting: Zneužijte Metasploitable jako odrazový můstek pro útok na další (virtuální) stroje ve stejné vnitřní síti.
4. Další praktiky
Sniffing sítě: Použijte Wireshark v Kali k odposlouchávání nešifrované komunikace (Telnet, FTP, HTTP) probíhající mezi stroji.
Analýza logů: Zkontrolujte, jaké stopy (logs) útok zanechal v /var/log/ na Metasploitable. 


Základní info pro začátek
Login do Metasploitable: msfadmin / msfadmin.
Zjištění IP v Metasploitable: příkaz ifconfig





Když používám Nmap, pomáhá mi zjistit, které porty jsou na cílovém zařízení otevřené a jaké služby na nich běží. 

Porty si představuji jako dveře do počítače – některé jsou zavřené, některé otevřené a za každými může být jiná služba, například SSH na portu 22 nebo FTP na portu 21. 

Přepínač -sV používám tehdy, když chci zjistit verzi služby, která na portu běží. Díky tomu místo informace „na portu 22 je SSH“ zjistím například i konkrétní verzi. 

Přepínač -sC spouští základní skripty, které umí získat další informace o službě nebo odhalit jednoduché zranitelnosti. 

Když chci prohledat všechny porty od 1 do 65535, použiji ještě -p-, protože běžný scan kontroluje jen nejčastěji používané porty.



>What does  the 3-letter acronym FTP stand for?

File Transfer Protocol

Which port does the FTP service listen on usually?

21

$ sudo nmap 10.10.14.212
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-27 08:57 CDT
Nmap scan report for 10.10.14.212
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind


sudo nmap -sV -sC -p- 10.10.14.212

-p- prohledá všechny porty 1–65535
-sV zjistí, co na nich běží
-sC spustí základní detekční skripty

>What does  the 3-letter acronym FTP stand for?

File Transfer Protocol

Which port does the FTP service listen on usually?

21

$ sudo nmap 10.10.14.212
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-27 08:57 CDT
Nmap scan report for 10.10.14.212
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind


sudo nmap -sV -sC -p- 10.10.14.212

-p- prohledá všechny porty 1–65535
-sV zjistí, co na nich běží
-sC spustí základní detekční skripty


EXERCICE1:
To build a complete profile of the target, I executed an aggressive nmap scan using the -sC -sV -Pn arguments.

1. Otevru Kali a metasploitable v me virtualni laboratori
2. Ve wireshraku v metaploitable se zeptam arp-a na IP adresy
3. v kali v terminalu provedu skan portu: nmap -sV 10.0.2.2
4. uvidim otervene porty a sluzby
5.v kali si otevru prohlize http://10.0.2.2
6. to mi nefunguje, tak zkusim zjistit jak je na tom port 80 nmap -p 80 10.0.2.2
7. kali pise, ze port 80 je TCP port, http, a je filtered

Když nmap ukáže:

80/tcp filtered http

znamená to, že se Kali Linux k portu 80 na Metasploitable 2 vůbec nedostane — paket je blokovaný nebo VM nejsou ve stejné síti.


EXERCISE2:
1. nmap -sV 10.0.2.2
	Hledám hlavně něco jako:

	21/tcp open ftp
	22/tcp open ssh
	139 nebo 445 samba
	3306 MySQL


Po nmap -sV 10.0.2.2 jsem našla několik zajímavých otevřených portů. Další krok je vybrat jednu službu a zkusit její enumeraci. Pokud je otevřený port 21, mohu zkusit připojení přes File Transfer Protocol:
Toto je muj vysledek:
135/tcp open  msrpc         Microsoft Windows RPC
445/tcp open  microsoft-ds?

Port 135 je Microsoft RPC Endpoint Mapper a port 445 je Microsoft SMB. To znamená, že na cíli běží sdílení souborů a síťové služby typické pro Windows.

Další krok je zkusit zjistit více informací o SMB na portu 445.

Jeste si poznamenam: 10.0.2.15 je IP KALI!

EXERCISE 3:
Pracuji s Host-only Adapter Network settings.
IP Metasploitable je 192.168.56.102

1. Provedu mapování: nmap -F IP
                                                                  
┌──(kali㉿kali)-[~]
└─$ nmap -F  192.168.56.102
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-30 16:20 EDT
Nmap scan report for 192.168.56.102
Host is up (0.00069s latency).
Not shown: 84 closed tcp ports (reset)
PORT      STATE SERVICE
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
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
MAC Address: 08:00:27:B9:D4:C2 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 13.27 seconds

2. Zjistim si vice informaci o portech, o kterych vim, ze jsou otevrene:
nmap -sV -sC -p 21,22,80,135,139,445,3306,3389,8009,8080 192.168.56.102


                                                                                                                        
┌──(kali㉿kali)-[~]
└─$ nmap -sV -sC -p 21,22,80,135,139,445,3306,3389,8009,8080 192.168.56.102
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-30 16:27 EDT
Nmap scan report for 192.168.56.102
Host is up (0.0011s latency).

PORT     STATE SERVICE        VERSION
21/tcp   open  ftp            Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp   open  ssh            OpenSSH 7.1 (protocol 2.0)
| ssh-hostkey: 
|   2048 fd:08:98:ca:3c:e8:c1:3c:ea:dd:09:1a:2e:89:a5:1f (RSA)
|_  521 7e:57:81:8e:f6:3c:1d:cf:eb:7d:ba:d1:12:31:b5:a8 (ECDSA)
80/tcp   open  http           Microsoft IIS httpd 7.5
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
135/tcp  open  msrpc          Microsoft Windows RPC
139/tcp  open  netbios-ssn    Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds   Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds
3306/tcp open  mysql          MySQL 5.5.20-log
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.20-log
|   Thread ID: 7
|   Capabilities flags: 63487
|   Some Capabilities: Support41Auth, DontAllowDatabaseTableColumn, IgnoreSpaceBeforeParenthesis, LongColumnFlag, IgnoreSigpipes, Speaks41ProtocolOld, SupportsTransactions, FoundRows, InteractiveClient, ConnectWithDatabase, SupportsLoadDataLocal, Speaks41ProtocolNew, ODBCClient, SupportsCompression, LongPassword, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: -x8JM%o#Z|%=CyX-v/"]
|_  Auth Plugin Name: mysql_native_password
3389/tcp open  ms-wbt-server?
| ssl-cert: Subject: commonName=vagrant-2008R2
| Not valid before: 2026-01-28T13:46:15
|_Not valid after:  2026-07-30T13:46:15
| rdp-ntlm-info: 
|   Target_Name: VAGRANT-2008R2
|   NetBIOS_Domain_Name: VAGRANT-2008R2
|   NetBIOS_Computer_Name: VAGRANT-2008R2
|   DNS_Domain_Name: vagrant-2008R2
|   DNS_Computer_Name: vagrant-2008R2
|   Product_Version: 6.1.7601
|_  System_Time: 2026-03-30T20:28:35+00:00
|_ssl-date: 2026-03-30T20:28:38+00:00; -2s from scanner time.
8009/tcp open  ajp13          Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8080/tcp open  http           Oracle GlassFish 4.0 (Servlet 3.1; JSP 2.3; Java 1.8)
| http-methods: 
|_  Potentially risky methods: PUT DELETE TRACE
|_http-title: GlassFish Server - Server Running
|_http-server-header: GlassFish Server Open Source Edition  4.0 
MAC Address: 08:00:27:B9:D4:C2 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h23m59s, deviation: 3h07m50s, median: 0s
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Standard 7601 Service Pack 1 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: vagrant-2008R2
|   NetBIOS computer name: VAGRANT-2008R2\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-03-30T13:28:35-07:00
|_nbstat: NetBIOS name: VAGRANT-2008R2, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:b9:d4:c2 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2026-03-30T20:28:35
|_  start_date: 2026-03-30T20:14:29


Nejzajímavější věci, které jsem našla:

FTP na portu 21: Microsoft FTP Service
Web na portu 80: Microsoft IIS 7.5 s povoleným TRACE
SMB na 445: podpis není vyžadován
Web na 8080: GlassFish Server 4.0
AJP na 8009: často bývá spojený s Apache Tomcat nebo Java serverem

Co dal?
1. Web (80, 8080)
2. FTP anonymous
3. SMB shares
4. Potom teprve hledat konkrétní zranitelnosti verzí, například pro:
GlassFish Server 4.0


EXERCISE 4:
1.msfconsole
2.search http scanner
3.use auxiliary/scanner/http/http_version
set RHOSTS 192.168.56.102
set RPORT 80
run
4.



>What does  the 3-letter acronym FTP stand for?

File Transfer Protocol

Which port does the FTP service listen on usually?

21

$ sudo nmap 10.10.14.212
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-27 08:57 CDT
Nmap scan report for 10.10.14.212
Host is up (0.0000040s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind


sudo nmap -sV -sC -p- 10.10.14.212

-p- prohledá všechny porty 1–65535
-sV zjistí, co na nich běží
-sC spustí základní detekční skripty


