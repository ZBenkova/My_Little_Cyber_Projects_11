# My_Little_Cyber_Projects_11
Když používám Nmap, pomáhá mi zjistit, které porty jsou na cílovém zařízení otevřené a jaké služby na nich běží. 

Porty si představuji jako dveře do počítače – některé jsou zavřené, některé otevřené a za každými může být jiná služba, například SSH na portu 22 nebo FTP na portu 21. 

Přepínač -sV používám tehdy, když chci zjistit verzi služby, která na portu běží. Díky tomu místo informace „na portu 22 je SSH“ zjistím například i konkrétní verzi. 

Přepínač -sC spouští základní skripty, které umí získat další informace o službě nebo odhalit jednoduché zranitelnosti. 

Když chci prohledat všechny porty od 1 do 65535, použiji ještě -p-, protože běžný scan kontroluje jen nejčastěji používané porty.
