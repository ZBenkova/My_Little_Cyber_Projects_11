# My_Little_Cyber_Projects_11
Když používám Nmap, pomáhá mi zjistit, které porty jsou na cílovém zařízení otevřené a jaké služby na nich běží. 

Porty si představuji jako dveře do počítače – některé jsou zavřené, některé otevřené a za každými může být jiná služba, například SSH na portu 22 nebo FTP na portu 21. 

Přepínač -sV používám tehdy, když chci zjistit verzi služby, která na portu běží. Díky tomu místo informace „na portu 22 je SSH“ zjistím například i konkrétní verzi. 

Přepínač -sC spouští základní skripty, které umí získat další informace o službě nebo odhalit jednoduché zranitelnosti. 

Když chci prohledat všechny porty od 1 do 65535, použiji ještě -p-, protože běžný scan kontroluje jen nejčastěji používané porty.


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


