---
layout: single
title: Bastion - Hack The Box
excerpt: "Bastion era una caja sólida y fácil con algunos desafíos simples como montar un VHD desde un recurso compartido de archivos y recuperar contraseñas de un programa de bóveda de contraseñas. Comienza, de manera algo inusual, sin un sitio web, sino más bien con imágenes vhd en un recurso compartido SMB, que, una vez montadas, brindan acceso a la colmena del registro necesaria para extraer las credenciales. Estas credenciales brindan la capacidad de ingresar al host como usuario. Para obtener acceso de administrador, aprovecharé la instalación de mRemoteNG, extraeré los datos del perfil y los datos cifrados, y mostraré varias formas de descifrarlos. Una vez que separe la contraseña de administrador, puedo ingresar como administrador."
date: 2021-06-24
classes: wide
header:
  teaser: /assets/images/Bastion/Bastion/Untitled.png
  teaser_home_page: true
  icon: /assets/images/Bastion/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - SMB  
  - Active Direcotry
---



# Bastion HTB

Hola muy buenas a todos! Hoy voy a estar resolviendo la maquina Bastión de HTB, es una maquina Windows de dificultad fácil. Es una de las máquinas Windows de HTB que recomiendan para prepararse el OSCP:

![/assets/images/Bastion/Untitled.png](/assets/images/Bastion/Untitled.png)

![/assets/images/Bastion/Untitled%201.png](/assets/images/Bastion/Untitled%201.png)

![/assets/images/Bastion/Untitled%202.png](/assets/images/Bastion/Untitled%202.png)

# **ENUMERACIÓN CON NMAP**

Lo primero de todo como siempre será enumerar que puertos tiene abiertos la maquina, para ellos vamos a utilizar la herramienta "NMAP" con la que haremos un escaneo exhaustivo de puertos.

Para hacer un "Fast Scan" de puertos, siempre suelo utilizar esta sintaxis:

`nmap -sS —min-rate 5000 -p- —open -n -Pn -vvv <IPMACHINE> -oN <FILENAME>` 

-sS : 

![/assets/images/Bastion/Untitled%203.png](/assets/images/Bastion/Untitled%203.png)

—min-rate :

![/assets/images/Bastion/Untitled%204.png](/assets/images/Bastion/Untitled%204.png)

-p- : Escaneo de todos los puertos

![/assets/images/Bastion/Untitled%205.png](/assets/images/Bastion/Untitled%205.png)

—open : Mostrar unicamente puertos abiertos

-n : 

![/assets/images/Bastion/Untitled%206.png](/assets/images/Bastion/Untitled%206.png)

-Pn : Para que no haga descubrimientos de hosts

![/assets/images/Bastion/Untitled%207.png](/assets/images/Bastion/Untitled%207.png)

-vvv : 

![/assets/images/Bastion/Untitled%208.png](/assets/images/Bastion/Untitled%208.png)

-oN :

![/assets/images/Bastion/Untitled%209.png](/assets/images/Bastion/Untitled%209.png)

![/assets/images/Bastion/Untitled%2010.png](/assets/images/Bastion/Untitled%2010.png)

Yo lo exporto en formato Grep para extraer los puertos con la utilidad "extractPorts" del Youtuber/Streamer "S4vitaar".

Esta utilidad me copia los puertos en la clipboard. Os comparto el Script por aquí pero recordad dejar una estrellita en el Github de S4vitar: (solo tenéis que tener instalado xclip y pegar este código en la .bashrc o .zshrc)

[https://pastebin.com/tYpwpauW](https://pastebin.com/tYpwpauW) 

![/assets/images/Bastion/Untitled%2011.png](/assets/images/Bastion/Untitled%2011.png)

Ahora vamos a enumerar versiones y servicios de todos los puertos con Nmap:

`nmap -sC -sV -p<PUERTOS> <IPMACHINE> -oN <FILENAME>`

-sC : Lanzar una serie de scripts basicos de enumeración

![/assets/images/Bastion/Untitled%2012.png](/assets/images/Bastion/Untitled%2012.png)

-sV : 

![/assets/images/Bastion/Untitled%2013.png](/assets/images/Bastion/Untitled%2013.png)

![/assets/images/Bastion/Untitled%2014.png](/assets/images/Bastion/Untitled%2014.png)

Podemos ver el nombre de la maquina: BASTION

Como siempre me gusta echarle un vistazo primero al puerto 445 (SAMBA), y tenemos permisos de listar el contenido con el uso de un Null Session:

![/assets/images/Bastion/Untitled%2015.png](/assets/images/Bastion/Untitled%2015.png)

Enumerando el sistema por samba encontramos una nota.

![/assets/images/Bastion/Untitled%2016.png](/assets/images/Bastion/Untitled%2016.png)

y también encontramos dos .vhd :

![/assets/images/Bastion/Untitled%2017.png](/assets/images/Bastion/Untitled%2017.png)

estos archivos son muy pesados y tardariamos mucho en descargarnoslos, asi que vamos a hacer una montura para poder visualizar los archivos de la maquina desde nuestra montura (esto no significa que lo tengamos descargados):

vamos a realizar la montura:

`mkdir /mnt/L4mpje-PC`

`mkdir /mnt/vhd`

`modprobe nbd`

`mount -t cifs //10.10.10.134/Backups/WindowsImageBackup/L4mpje-PC /mnt/L4mpje-PC/ -o user=anonymous`

`qemu-nbd -r -c /dev/nbd0 "/mnt/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd"`

 `mount -r /dev/nbd0p1 /mnt/vhd`

si el comando qemu-nbd no os funciona teneis que instalarlo:

`sudo apt-get install qemu-kvm qemu virt-manager virt-viewer`

# **ESC. PRIV. ROOT**

Enumeramos la montura y en el directorio Users no vemos nada relevante, asi que enumeramos el directorio : C:\Windows\System32\config

![/assets/images/Bastion/Untitled%2018.png](/assets/images/Bastion/Untitled%2018.png)

Podemos dumpear la sam con la herramienta samdump2:

![/assets/images/Bastion/Untitled%2019.png](/assets/images/Bastion/Untitled%2019.png)

y podemos crackear el hash NTLM del usuario l4mpje:

![/assets/images/Bastion/Untitled%2020.png](/assets/images/Bastion/Untitled%2020.png)

Podemos usar la contraseña para conectarnos por ssh a la maquina:

![/assets/images/Bastion/Untitled%2021.png](/assets/images/Bastion/Untitled%2021.png)

Enumerando los programas que tiene la maquina instalado encontramos que tiene mRemoteNG:

![/assets/images/Bastion/Untitled%2022.png](/assets/images/Bastion/Untitled%2022.png)

Por internet encuentro este [post](https://ethicalhackingguru.com/how-to-exploit-remote-connection-managers/) que explica como escalar privilegios abusando de este programa:

Seguimos los pasos del post y encontramos este archivo:

![/assets/images/Bastion/Untitled%2023.png](/assets/images/Bastion/Untitled%2023.png)

Y dentro del archivo encontramos las credenciales encriptadas del administrador:

![/assets/images/Bastion/Untitled%2024.png](/assets/images/Bastion/Untitled%2024.png)

Tenemos que desencriptarlas con esta herramienta [mremoteng_decrypt.py](https://github.com/haseebT/mRemoteNG-Decrypt/blob/master/mremoteng_decrypt.py) 

![/assets/images/Bastion/Untitled%2025.png](/assets/images/Bastion/Untitled%2025.png)

![/assets/images/Bastion/Untitled%2026.png](/assets/images/Bastion/Untitled%2026.png)

![/assets/images/Bastion/Untitled%2027.png](/assets/images/Bastion/Untitled%2027.png)

![/assets/images/Bastion/Untitled%2028.png](/assets/images/Bastion/Untitled%2028.png)

Espero que les haya gustado y sido de ayuda, nos vemos en la siguiente maquina!