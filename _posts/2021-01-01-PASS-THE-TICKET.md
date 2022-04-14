# PASS-THE-TICKET

Antes que nada os recomiendo mucho leer este [post](https://www.tarlogic.com/blog/como-funciona-kerberos/) de Tarlogic donde explican muy bien como funciona kerberos en Active Directory.

El entorno donde se van a realizar las pruebas es el siguiente:

![Untitled Diagram.drawio (19).png](/assets/images/ASPROAST-ATTACK/Untitled_Diagram.drawio_(19).png)

El primer ***ticket*** que trataremos es el ***TGT*** o ***Ticket Granting Ticket***. Lo presentaremos ante el ***KDC*** para obtener el ***TGS*** o ***Ticket Granting Service***. El ***TGT*** será cifrado con la clave del ***KDC***, es decir, la clave derivada de la cuenta del ***krbtgt.***

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%201.png)

Haremos una suplantación de usuario, en vez de utilizando el ***hash ntlm*** de la cuenta del usuario, utilizaremos un ***ticket Kerberos*** con validez y que pertenezca a dicho usuario.

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%202.png)

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%203.png)

Nos traemos el ticket a nuestra maquina de atacante:

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%204.png)

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%205.png)

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%206.png)

Nos conectamos a PC-ADRI

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%207.png)

Si intentamos ver los recursos compartidos en el DC, nos da permiso denegado

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%208.png)

Nos pasamos el mimikatz y el golden.kirbi a la maquina

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%209.png)

Ejecutamos mimikatz seguido de los comandos que se ven en la imagen para cargar el .kirbi, y de esta forma poder tener acceso al Controlador de Dominio.

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%2010.png)

Otra forma posible seria la siguiente, con la ayuda del script en python ticketer, podemos crear un archivo llamado Administrador.ccache.

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%2011.png)

Lo exportamos a la variable KRB5CCNAME

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%2012.png)

Podemos conectarnos al DC sin proporcionar contraseña.

![Untitled](/assets/images/PASS-THE-TICKET/Untitled%2013.png)

De esta forma tenemos persistencia absoluta en el DC aunque cambien la contraseña del administrador.
