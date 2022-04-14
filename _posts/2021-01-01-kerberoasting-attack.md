# kerberoasting-attack

¿Cómo se aplica todo esto a Kerberoasting? Bueno, en lo que respecta a la intención, comprometer el dominio de Active Directory comienza cuando el atacante escanea el entorno en busca de cuentas de dominio cuyos valores de SPN indiquen cualquier asociación relacionada con el servicio.

Con esta información en la mano, el siguiente paso es solicitar tickets de servicio TGS para cualquier SPN de alto privilegio utilizando su TGT de prueba de identidad mientras se enfoca en SPN específicamente vinculados a cuentas de usuario de dominio y no basados en host.

Las cuentas de servicio son cuentas no humanas que se utilizan para ejecutar servicios o aplicaciones. Las cuentas de servicio a menudo tienen privilegios elevados, sus contraseñas rara vez se cambian y los equipos de seguridad rara vez las supervisan.

Cada instancia de servicio tiene un identificador único llamado Nombre principal del servicio (SPN), que también incluye información sobre para qué se usa la cuenta y su ubicación. Cualquier usuario autenticado puede iniciar sesión en un dominio de Active Directory y enviar una solicitud de un ticket de servicio de concesión de tickets (TGS) para cualquier cuenta de servicio especificando su valor de SPN.

Luego, pueden extraer el hash de la contraseña de la cuenta de servicio e intentar un ataque de fuerza bruta para obtener la contraseña de texto sin formato, con poco riesgo de ser detectados o bloqueados de su cuenta.

Para la realización de este ataque vamos a utilizar este entorno ya configurado:

![Untitled](/assets/images/kerberoasting-attack/Untitled.png)

Si ejecutamos la herramienta de impacket GetUserSPNs, podemos ver que nuestro dominio no cuenta con ninguna cuenta configurada para realizar este ataque.

![Untitled](/assets/images/kerberoasting-attack/Untitled%201.png)

Configuramos para que el usuario hackblog tenga el atributo SPN.

![Untitled](/assets/images/kerberoasting-attack/Untitled%202.png)

Con cualquier credencial que tengamos podemos realizar un kerberoasting-attack

![Untitled](/assets/images/kerberoasting-attack/Untitled%203.png)

Podemos obtener el TGS de este usuario e intentar crackearlo por fuerza bruta

![Untitled](/assets/images/kerberoasting-attack/Untitled%204.png)

Y mediante un ataque de diccionario offline podemos obtener la contraseña del usuario.

![Untitled](/assets/images/kerberoasting-attack/Untitled%205.png)
