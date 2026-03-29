# My_Little_Cyber_Projects_11

EXPLOITING METASPLOITABLE 2
Metasploitable 2 is an intentionally vulnerable Ubuntu Linux virtual machine designed for security training and penetration testing practice. In this walkthrough, I will methodically exploit its weaknesses, demonstrating a common penetration testing lifecycle: Reconnaissance, Enumeration and Exploitation. My attacking platform will be Kali Linux.

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

Další krok je zkusit zjistit více informací o SMB na portu 445:




EXERCISES

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


