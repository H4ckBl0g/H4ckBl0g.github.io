---
layout: single
title: TheNoteBook - Hack The Box
excerpt: "En TheNoteBook vamos a aprender a como usar las cookies a nuestro favor, gracias a ellas vamos a editar un JWT a nuestro antojo para convertirnos en admin de la web, subiremos una shell.php a la web para ganar acceso y una vez dentro obtendremos una clave ssh, en la escalada de privilegios nos toparemos con un CVE-2019-5736"
date: 2021-07-14
classes: wide
header:
  teaser: /assets/images/TheNoteBook/Untitled.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  -   Web
  -   Cookie
  -   JWT
  -   Docker
---

<div>
<p style = 'text-align:center;'>
<img src="http://www.hackthebox.eu/badge/image/497437" alt="" width="200px">
</p>
</div>

# TheNoteBook


Buenas! Hoy os voy a enseñar a como pwnear la maquina de HTB que acaban de retirar llamada TheNoteBook, es una maquina de dificultad media y de sistema operativo linux, os dejo aquí unas estadísticas de la maquina y la guia que vamos a seguir:

![/assets/images/TheNoteBook/excalidraw.png](/assets/images/TheNoteBook/excalidraw.png)
=======

![/assets/images/TheNoteBook/Untitled.png](/assets/images/TheNoteBook/Untitled.png)

![/assets/images/TheNoteBook/Untitled%201.png](/assets/images/TheNoteBook/Untitled%201.png)

# **ESCANEO**

# **ENUMERACIÓN CON NMAP**

Lo primero de todo como siempre será enumerar que puertos tiene abiertos la maquina, para ellos vamos a utilizar la herramienta "NMAP" con la que haremos un escaneo exhaustivo de puertos.

Para hacer un "Fast Scan" de puertos, siempre suelo utilizar esta sintaxis:

`nmap -sS —min-rate 5000 -p- —open -n -Pn -vvv <IPMACHINE> -oN <FILENAME>` 

-sS : 

![/assets/images/TheNoteBook/Untitled%202.png](/assets/images/TheNoteBook/Untitled%202.png)

—min-rate :

![/assets/images/TheNoteBook/Untitled%203.png](/assets/images/TheNoteBook/Untitled%203.png)

-p- : Escaneo de todos los puertos

![/assets/images/TheNoteBook/Untitled%204.png](/assets/images/TheNoteBook/Untitled%204.png)

—open : Mostrar unicamente puertos abiertos

-n : 

![/assets/images/TheNoteBook/Untitled%205.png](/assets/images/TheNoteBook/Untitled%205.png)

-Pn : Para que no haga descubrimientos de hosts

![/assets/images/TheNoteBook/Untitled%206.png](/assets/images/TheNoteBook/Untitled%206.png)

-vvv : 

![/assets/images/TheNoteBook/Untitled%207.png](/assets/images/TheNoteBook/Untitled%207.png)

-oN :

![/assets/images/TheNoteBook/Untitled%208.png](/assets/images/TheNoteBook/Untitled%208.png)

![/assets/images/TheNoteBook/Untitled%209.png](/assets/images/TheNoteBook/Untitled%209.png)

Yo lo exporto en formato Grep para extraer los puertos con la utilidad "extractPorts" del Youtuber/Streamer "S4vitaar".

Esta utilidad me copia los puertos en la clipboard. Os comparto el Script por aquí pero recordad dejar una estrellita en el Github de S4vitar: (solo tenéis que tener instalado xclip y pegar este código en la .bashrc o .zshrc)

[https://pastebin.com/tYpwpauW](https://pastebin.com/tYpwpauW) 

![/assets/images/TheNoteBook/Untitled%2010.png](/assets/images/TheNoteBook/Untitled%2010.png)

Ahora vamos a enumerar versiones y servicios de todos los puertos con Nmap:

`nmap -sC -sV -p<PUERTOS> <IPMACHINE> -oN <FILENAME>`

-sC : Lanzar una serie de scripts basicos de enumeración

![/assets/images/TheNoteBook/Untitled%2011.png](/assets/images/TheNoteBook/Untitled%2011.png)

-sV : 

![/assets/images/TheNoteBook/Untitled%2012.png](/assets/images/TheNoteBook/Untitled%2012.png)

![/assets/images/TheNoteBook/Untitled%2013.png](/assets/images/TheNoteBook/Untitled%2013.png)

Con la herramienta `whatweb`podemos sacar algo de información de la pagina web:

![/assets/images/TheNoteBook/Untitled%2014.png](/assets/images/TheNoteBook/Untitled%2014.png)

En la pagina web podemos crearnos un usuario, nos creamos nuestro propio usuario:

tukutu:test123:test@test.com

![/assets/images/TheNoteBook/Untitled%2015.png](/assets/images/TheNoteBook/Untitled%2015.png)

Enumerando un poco la pagina vemos una cookie un tanto extraña:

![/assets/images/TheNoteBook/Untitled%2016.png](/assets/images/TheNoteBook/Untitled%2016.png)

# **EXPLOTACION DE LA VULNERABILIDAD**

Vamos a ver que podemos hacer con ella desde `Burpsuite`

Tiene un formato que se parece a un JSON de JWT, copiamos la cookie y la analizamos en [jwt.io](http://jwt.io) 

Por google encuentro como aprovecharme de esto en este [link](https://blog.pentesteracademy.com/hacking-jwt-tokens-kid-claim-misuse-key-leak-e7fce9a10a9c)

Cambiamos el admin_cap a 1 y nos creamos una key en nuestra maquina local:

![/assets/images/TheNoteBook/Untitled%2017.png](/assets/images/TheNoteBook/Untitled%2017.png)

vamos a sustituir el admin_cap a 1:

![/assets/images/TheNoteBook/Untitled%2018.png](/assets/images/TheNoteBook/Untitled%2018.png)

Importante agregar la privkey.key en el jwt.io:

![/assets/images/TheNoteBook/Untitled%2019.png](/assets/images/TheNoteBook/Untitled%2019.png)

Nos quedaría una cookie de esta forma:

![/assets/images/TheNoteBook/Untitled%2020.png](/assets/images/TheNoteBook/Untitled%2020.png)

La copiamos y cambiamos la cookie por la creada, y hemos ganado acceso al panel de admin:

![/assets/images/TheNoteBook/Untitled%2021.png](/assets/images/TheNoteBook/Untitled%2021.png)

# **GANANDO ACCESO A LA MAQUINA**

Podemos subir un archivo; vamos a subir una rev shell:

![/assets/images/TheNoteBook/Untitled%2022.png](/assets/images/TheNoteBook/Untitled%2022.png)

![/assets/images/TheNoteBook/Untitled%2023.png](/assets/images/TheNoteBook/Untitled%2023.png)

Editamos el php y lo subimos:

![/assets/images/TheNoteBook/Untitled%2024.png](/assets/images/TheNoteBook/Untitled%2024.png)

![/assets/images/TheNoteBook/Untitled%2025.png](/assets/images/TheNoteBook/Untitled%2025.png)

Hemos obtenido una Rev. Shell:

![/assets/images/TheNoteBook/Untitled%2026.png](/assets/images/TheNoteBook/Untitled%2026.png)

Hacemos el tratamiento de la TTY:

`script /dev/null -c bash`

`CTRL + Z`

`stty raw -echo;fg`

`reset`

`xterm`

`export TERM=xterm`

`export SHELL=bash`

`stty rows [] columns []`

# **PIVOTANDO A NOAH**

Ya tenemos una consola interactiva como el usuario www-data, toca pivotar al usuario noah para luego escalar privilegios:

Enumerando el sistema encuentro en la carpeta backups, un archivo llamado home.tar.gz, me lo voy a traspasar a mi equipo para extraerlo:

![/assets/images/TheNoteBook/Untitled%2027.png](/assets/images/TheNoteBook/Untitled%2027.png)

![/assets/images/TheNoteBook/Untitled%2028.png](/assets/images/TheNoteBook/Untitled%2028.png)

Lo descomprimimos dos veces con la herramienta `7z`

`7z x home.tar.gz`

`7z x home.tar`

Una vez descomprimido tenemos acceso a la carpeta home de la maquina, por lo que podemos usar la id_rsa de noah para conectarnos por ssh: (`chmod 600 id_rsa`)

![/assets/images/TheNoteBook/Untitled%2029.png](/assets/images/TheNoteBook/Untitled%2029.png)

![/assets/images/TheNoteBook/Untitled%2030.png](/assets/images/TheNoteBook/Untitled%2030.png)

# **ESCALADA DE PRIVILEGIOS**

Toca escalar privilegios a root, con sudo -l nos muestra los programas o comandos que podemos ejecutar con permisos temporales:

![/assets/images/TheNoteBook/Untitled%2031.png](/assets/images/TheNoteBook/Untitled%2031.png)

Examinando que podemos hacer con el comando:

![/assets/images/TheNoteBook/Untitled%2032.png](/assets/images/TheNoteBook/Untitled%2032.png)

Podemos ejecutar comandos en un contenedor, vamos a spawnear una bash:

![/assets/images/TheNoteBook/Untitled%2033.png](/assets/images/TheNoteBook/Untitled%2033.png)

Busco la version del docker para ver si existe algun exploit:

![/assets/images/TheNoteBook/Untitled%2034.png](/assets/images/TheNoteBook/Untitled%2034.png)

[https://github.com/Frichetten/CVE-2019-5736-PoC](https://github.com/Frichetten/CVE-2019-5736-PoC)

Nos descargamos el main.go y modificamos a nuestro gusto:

![/assets/images/TheNoteBook/Untitled%2035.png](/assets/images/TheNoteBook/Untitled%2035.png)

Lo compilamos con `go build main.go` , y nos pasamos el archivo main al docker:

![/assets/images/TheNoteBook/Untitled%2036.png](/assets/images/TheNoteBook/Untitled%2036.png)

Y cuando veamos el Overwritten ejecutamos en otra ventana :

![/assets/images/TheNoteBook/Untitled%2037.png](/assets/images/TheNoteBook/Untitled%2037.png)

Ahora /bin/bash es SUID:

![/assets/images/TheNoteBook/Untitled%2038.png](/assets/images/TheNoteBook/Untitled%2038.png)
