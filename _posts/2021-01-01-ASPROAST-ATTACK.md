# ASPROAST-ATTACK

Para realizar este ataque es necesario tener una lista con nombres de usuarios que existan en el directorio activo, hay muchas formas de sacar usuarios validos del directorio activo:

- **Ataque de fuerza bruta con `kerbrute`:**
    - La herramienta `kerbrute` te permite realizar mucha enumeración de un DC, entre ellos una enumeración de usuarios con el siguiente comando: `./kerbrute userenum -d <domain> usernames.txt` , os recomiendo usar esta lista de usuarios: [link](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/xato-net-10-million-usernames.txt)
- **Enumeración de RPC con herramientas como `rpcclient` :**
    - Si la maquina victima tiene habilitada el inicio de sesión nulo o sin necesidad de autenticarse, puedes enumerar información bastante importante del dominio, como usuarios, grupos, etc. Para comprobar si el sistema victima tiene habilitado el NULL session lo podéis comprobar de esta forma: `rpcclient -U "" -N <IP>` , si tienen deshabilitado el NULL session pero tenéis credenciales del dominio podéis usarlas para enumerar el dominio con el siguiente comando: `rpcclient -U ”<USER>%<PASSWORD>” <IP>` , una vez consigáis acceder al RPC podéis obtener los usuarios del dominio ejecutando el comando: `enumdomusers`
- **OSINT de la organización o empresa:**
    - Otra opción para conseguir usuarios del dominio es realizar una recolección de información previa sobre la empresa u organización de la que queréis conseguir los usuarios, muchas de ellas utilizan los mismos usuarios del correo electrónico como usuarios del dominio, si la empresa tiene una pagina web podéis sacar información de dicha pagina en el apartado: “Conócenos”, “Quienes somos”, “ponte en contacto con nosotros”, etc...
- **Sacando información con Esteganografía:**
    - Otra opción seria sacar información de archivos que hemos obtenido de alguna forma que pertenezcan a la organización, por ejemplo un archivo PDF, el cual con la herramienta `exiftool`, podemos sacar información relevante como el nombre del usuario que ha creado el archivo.

### ASREPROAST-ATTACK

Una vez tenemos una lista de usuarios del dominio, podemos intentar realizar un ataque ASREPROAST, pero **¿Qué es un ataque ASREPROAST? ¿Qué requisitos tienen que existir para que tenga efecto?**

El ataque `ASREPRoast` busca usuarios sin el atributo requerido de autenticación previa de Kerberos (DONT_REQ_PREAUTH). Eso significa que cualquiera puede enviar una solicitud AS_REQ al DC en nombre de cualquiera de esos usuarios y recibir un mensaje AS_REP. Este último tipo de mensaje contiene una parte de los datos cifrados con la clave de usuario original, derivados de su contraseña y se podría realizar un ataque de fuerza bruta local para intentar de romper ese hash obtenido.

Yo estaré utilizando el mismo escenario que cree para el ataque de SMB-Relay.

![Untitled Diagram.drawio (19).png](ASPROAST-ATTACK/Untitled_Diagram.drawio_(19).png)

En este punto no esta configurado para que sea vulnerable a este ataque, si intentamos realizarlo nos dirá que ningún usuario tiene el atributo “No pedir la autenticación previa de kerberos” activada.

![Untitled](ASPROAST-ATTACK/Untitled.png)

![Untitled](ASPROAST-ATTACK/Untitled%201.png)

Ninguna cuenta lo tiene activado, para activarlo tenemos que ir al DC y configurar una cuenta activando esta casilla:

![Untitled](ASPROAST-ATTACK/Untitled%202.png)

Se ha activado para el usuario adrianhack, por lo que si ahora realizamos este ataque podremos obtener el **TGT** (Ticket Granting Ticket) del usuario. El TGT (Ticket Granting Ticket) es el ticket
que se presenta ante el KDC para obtener los TGS. El **TGS** (Ticket Granting Service) es el ticket
que se presenta ante un servicio para poder acceder a sus recursos. Se cifra con la clave del servicio correspondiente.

Hay varias formas de ejecutar este ataque:

- Con la herramienta `Get-NPUsers` de `impacket`
- Con el modulo PowerView.ps1 (`Get-DomainUser -PreauthNotRequired -verbose`), se necesita estar dentro de una maquina del directorio activo.
- Con la herramienta Rubeus.exe (`.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.asreproast`), también se necesita acceso a una maquina del AD.

Yo lo realizare desde mi maquina kali.

`impacket-GetNPUsers adricorp.local/ -no-pass -usersfile users`

![Untitled](ASPROAST-ATTACK/Untitled%203.png)

Obtenemos el TGT, el cual podemos romper por fuerza bruta si la contraseña es débil.

![Untitled](ASPROAST-ATTACK/Untitled%204.png)

### ¿Cómo prevenir este ataque?

- **Todos los usuarios del dominio tienen que pedir la autenticación previa de kerberos.**