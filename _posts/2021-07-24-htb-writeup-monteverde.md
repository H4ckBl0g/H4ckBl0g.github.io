---
layout: single
title: Monteverde - Hack The Box
excerpt: "Monteverde se centró en Azure Active Directory. Primero, miraré RPC para obtener una lista de usuarios y luego verificaré si alguno usó su nombre de usuario como contraseña. Con los créditos de SABatchJobs, obtendré acceso a SMB para encontrar un archivo de configuración XML con una contraseña para uno de los usuarios en el cuadro que tiene permisos de WinRM. Desde allí, puedo abusar de la base de datos del directorio activo de Azure para filtrar la contraseña de administrador."
date: 2021-07-16
classes: wide
header:
  teaser: /assets/images/Monteverde/Untitled.png
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

# Monteverde 

Hola muy buenas a todos! Hoy voy a estar resolviendo la maquina Monteverde de  HTB, es una maquina Windows de dificultad media, en la que vamos a aprender cosas nuevas de Directorio Activo, en mi opinion es muy importante este tema, ya que el 90% de las empresas lo usan, para el que no sepa que es aqui les dejo una deficion:

Active Directory (AD) o Directorio Activo (DA) son los términos que utiliza Microsoft para referirse a su implementación de servicio de directorio en una red distribuida de computadoras. Un Active Directory almacena información de una organización en una base de datos central, organizada y accesible.

![/assets/images/Monteverde/Untitled.png](/assets/images/Monteverde/Untitled.png)

![/assets/images/Monteverde/Untitled%201.png](/assets/images/Monteverde/Untitled%201.png)

# **ENUMERACIÓN CON NMAP**

Lo primero de todo como siempre será enumerar que puertos tiene abiertos la maquina, para ellos vamos a utilizar la herramienta "NMAP" con la que haremos un escaneo exhaustivo de puertos.

Para hacer un "Fast Scan" de puertos, siempre suelo utilizar esta sintaxis:

`nmap -sS —min-rate 5000 -p- —open -n -Pn -vvv <IPMACHINE> -oN <FILENAME>` 

-sS : 

![/assets/images/Monteverde/Untitled%202.png](/assets/images/Monteverde/Untitled%202.png)

—min-rate :

![/assets/images/Monteverde/Untitled%203.png](/assets/images/Monteverde/Untitled%203.png)

-p- : Escaneo de todos los puertos

![/assets/images/Monteverde/Untitled%204.png](/assets/images/Monteverde/Untitled%204.png)

—open : Mostrar unicamente puertos abiertos

-n : 

![/assets/images/Monteverde/Untitled%205.png](/assets/images/Monteverde/Untitled%205.png)

-Pn : Para que no haga descubrimientos de hosts

![/assets/images/Monteverde/Untitled%206.png](/assets/images/Monteverde/Untitled%206.png)

-vvv : 

![/assets/images/Monteverde/Untitled%207.png](/assets/images/Monteverde/Untitled%207.png)

-oN :

![/assets/images/Monteverde/Untitled%208.png](/assets/images/Monteverde/Untitled%208.png)

![/assets/images/Monteverde/Untitled%209.png](/assets/images/Monteverde/Untitled%209.png)

Yo lo exporto en formato Grep para extraer los puertos con la utilidad "extractPorts" del Youtuber/Streamer "S4vitaar".

Esta utilidad me copia los puertos en la clipboard. Os comparto el Script por aquí pero recordad dejar una estrellita en el Github de S4vitar: (solo tenéis que tener instalado xclip y pegar este código en la .bashrc o .zshrc)

[https://pastebin.com/tYpwpauW](https://pastebin.com/tYpwpauW) 

![/assets/images/Monteverde/Untitled%2010.png](/assets/images/Monteverde/Untitled%2010.png)

Ahora vamos a enumerar versiones y servicios de todos los puertos con Nmap:

`nmap -sC -sV -p<PUERTOS> <IPMACHINE> -oN <FILENAME>`

-sC : Lanzar una serie de scripts basicos de enumeración

![/assets/images/Monteverde/Untitled%2011.png](/assets/images/Monteverde/Untitled%2011.png)

-sV : 

![/assets/images/Monteverde/Untitled%2012.png](/assets/images/Monteverde/Untitled%2012.png)

![/assets/images/Monteverde/Untitled%2013.png](/assets/images/Monteverde/Untitled%2013.png)

Vemos el nombre del dominio, lo agregamos al /etc/hosts y procedemos a enumerar puertos:

Con RpcEnum vemos una lista de usuarios, lo guardamos por si nos hiciera falta:

![/assets/images/Monteverde/Untitled%2014.png](/assets/images/Monteverde/Untitled%2014.png)

![/assets/images/Monteverde/Untitled%2015.png](/assets/images/Monteverde/Untitled%2015.png)

No encuentro nada mas interesante con rpcenum, asi que voy a tratar de hacer un ataque de fuerza bruta contra el servicio samba, después de un buen rato con el rockyou sin encontrar nada, pruebo la reutilización de usuarios como contraseñas :

![/assets/images/Monteverde/Untitled%2016.png](/assets/images/Monteverde/Untitled%2016.png)

Bingo! SABatcJobs:SABatcJobs

![/assets/images/Monteverde/Untitled%2017.png](/assets/images/Monteverde/Untitled%2017.png)

![/assets/images/Monteverde/Untitled%2018.png](/assets/images/Monteverde/Untitled%2018.png)

Enumerando el puerto samba encontramos un archivo .xml

Nos lo pasamos a nuestra pagina y bingo! tiene una contraseña:

![/assets/images/Monteverde/Untitled%2019.png](/assets/images/Monteverde/Untitled%2019.png)

Nos conectamos con evil-winrm a la maquina:

![/assets/images/Monteverde/Untitled%2020.png](/assets/images/Monteverde/Untitled%2020.png)

Leemos la flag:

![/assets/images/Monteverde/Untitled%2021.png](/assets/images/Monteverde/Untitled%2021.png)

# **ESC. PRIV. ROOT**

Enumerando el usuario vemos que pertenece al grupo Azure Admins:

![/assets/images/Monteverde/Untitled%2022.png](/assets/images/Monteverde/Untitled%2022.png)

Buscando por google encuentro este [post](https://blog.xpnsec.com/azuread-connect-for-redteam/) que nos explica como podemos extraer la contraseña de la base de datos:

creamos un script en powershell tal y como indica la pagina solo cambiando esta linea:

`$client = new-object System.Data.SqlClient.SqlConnection -ArgumentList "Server=127.0.0.1;Database=ADSync;Integrated Security=True"`

lo cargamos en la maquina:

![/assets/images/Monteverde/Untitled%2023.png](/assets/images/Monteverde/Untitled%2023.png)

![/assets/images/Monteverde/Untitled%2024.png](/assets/images/Monteverde/Untitled%2024.png)

Y hemos conseguido sacar las credenciales de administrador:

![/assets/images/Monteverde/Untitled%2025.png](/assets/images/Monteverde/Untitled%2025.png)

Espero que os haya gustado y servidor, para ser una maquina de dificultad media, me ha parecido bastante fácil.