---
title: THM - Easy Peasy
date: 2022-12-14
categories: [Hacking Ético, Máquinas]
toc: true
img_path: /assets/lib/THM-EasyPeasy
---

Write up de la máquina de TryHackMe - Easy Peasy, donde utilizaremos las herramientas Nmap, GoBuster, JohnTheRipper y diferentes métodos de desencriptado.

## Escaneo de puertos

A continuación realizaremos un mapeo de puertos, con **Nmap** para obtener información de los puertos que estan abiertos y que servicios se encuentran en cada uno de ellos.

```
$ sudo nmap -sC -sV -p- 10.10.189.205
    PORT      STATE SERVICE VERSION
    80/tcp    open  http    nginx 1.16.1
    | http-robots.txt: 1 disallowed entry 
    |_/
    |_http-title: Welcome to nginx!
    |_http-server-header: nginx/1.16.1
    6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
    |   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
    |_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)
    65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
    |_http-title: Apache2 Debian Default Page: It works
    | http-robots.txt: 1 disallowed entry 
    |_/
    |_http-server-header: Apache/2.4.43 (Ubuntu)
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Fuerza bruta a los directorios

Tras identificar los servicios en los diferentes puertos comenzaremos a utilizar la herramienta **GoBuster** en cada uno de ellos para investigar los posibles directorios ocultos y posteriormente acceder a ellos.
### Primer directorio oculto

```
$ sudo gobuster dir -u http://sudo gobuster dir -u http://10.10.189.205 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    ===============================================================
    Gobuster v3.3
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.189.205
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.3
    [+] Timeout:                 10s
    ===============================================================
    2022/12/15 03:50:23 Starting gobuster in directory enumeration mode
    ===============================================================
    /hidden               (Status: 301) [Size: 169] [--> http://10.10.189.205/hidden/]
```
Podemos comprobar que se ha encontrado un nuevo directorio llamado **hidden** por lo que accederemos a el para seguir investigando.

![Captura de pantalla de 2022-12-14 10-12-24.png](Captura_de_pantalla_de_2022-12-14_10-12-24.png)

Inspeccionaremos el código fuente del directorio por si encontramos alguna pista mientras volvemos a ejecutar **GoBuster** en este directorio por si a su vez posee un nuevo directorio oculto dentro de él.
### Segundo directorio oculto
```
$ sudo gobuster dir -u http://sudo gobuster dir -u http://10.10.189.205/hidden -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    ===============================================================
    Gobuster v3.3
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.189.205/hidden
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.3
    [+] Timeout:                 10s
    ===============================================================
    2022/12/15 03:51:13 Starting gobuster in directory enumeration mode
    ===============================================================
    /whatever             (Status: 301) [Size: 169] [--> http://10.10.189.205/hidden/whatever/]
```

Tras no encontrar ninguna pista en el directorio **hidden** accederemos al nuevo directorio encontrado **whatever**.

![Captura de pantalla de 2022-12-14 10-13-41.png](Captura_de_pantalla_de_2022-12-14_10-13-41.png)

Inspeccionamos el código fuente en busca de alguna pista y nos encontramos con lo siguiente.

![Captura de pantalla de 2022-12-14 10-15-01.png](Captura_de_pantalla_de_2022-12-14_10-15-01.png)

Podemos observar que existe una etiqueta HTML que esta oculta con lo que podría ser un hash a descifrar.
El final de este hash nos indica que podría encontrarse encriptado en Base64 por lo que realizaremos la siguiente operación.

```
echo "ZmxhZ3tmMXJzN19mbDRnfQ==" | base64 -d
flag{f1rs7_fl4g} 
```
### Primera flag
Aqui tenemos nuestra primera flag.
> flag{f1rs7_fl4g}
{: .prompt-tip }

Llegados a este punto comenzaremos a investigar cada uno de los puertos y servicios comenzando así por donde identificamos un servidor Apache, que se encuentra en el puerto **:65524**.
Como vimos anteriormente, tenemos un fichero **robots.txt** que podría ser interesante de investigar y una vez accedemos a él nos encontramos con lo siguiente.

```
User-Agent:*
Disallow:/
Robots Not Allowed
User-Agent:a18672860d0510e5ab6699730763b250
Allow:/
This Flag Can Enter But Only This Flag No More Exceptions
```
A continuación vamos a intentar averiguar de que tipo de hash se trata de la siguiente manera.
```
$ hash-identifier "a18672860d0510e5ab6699730763b250"                  
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
   ---------------------------------------------------

    Possible Hashs:
    [+] MD5
    [+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```
Por lo tanto vamos a utilizar alguna de las herramientas online que existen para descifrar este hash.

![Captura de pantalla de 2022-12-14 10-30-40.png](Captura_de_pantalla_de_2022-12-14_10-30-40.png)
### Segunda flag
Aqui tenemos nuestra segunda flag.
> flag{1m_s3c0nd_fl4g}
{: .prompt-tip }
## Servicio Apache
En primer lugar accederemos al directorio principal del puerto donde tenemos el servicio Apache e inspeccionaremos el código fuente de la misma realizando una búsqueda en él por la palabra clave **flag** y obtendremos el siguiente resultado.
```
<li>
    They are activated by symlinking available
    configuration files from their respective
    Fl4g 3 : flag{9fdafbd64c47471a8f54cd3fc64cd312}
	available/ counterparts. These should be managed
    by using our helpers
```
### Tercera flag
Aqui tenemos nuestra tercera flag.
> flag{9fdafbd64c47471a8f54cd3fc64cd312}
{: .prompt-tip }

A continuación accederemos al directorio principal del puerto donde tenemos el servicio Apache e inspeccionaremos el código fuente de la misma realizando una búsqueda en él por la palabra clave **hidden** y obtendremos el siguiente resultado.

```
<span class="floating_element">
    Apache 2 It Works For Me
	<p hidden>its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu</p>
</span>
```

Tras investigar un poco comprobamos que este hash se encuentra en cifrado de **Base62** por lo que vamos a descifrarlo con alguna herramienta online.

![Captura de pantalla de 2022-12-14 09-32-17.png](Captura_de_pantalla_de_2022-12-14_09-32-17.png)

Nos damos cuenta de que no se trata de ninguna flag si no que es un nuevo directorio.
### Nuevo directorio descifrado
> /n0th1ng3ls3m4tt3r
{: .prompt-info }
 
Una vez accedemos al directorio nos encontramos lo siguiente.

![Captura de pantalla de 2022-12-14 10-07-48.png](Captura_de_pantalla_de_2022-12-14_10-07-48.png)

Si inspeccionamos el código fuente del directorio nos encontramos con **dos potenciales pistas**, una imagen y un posible hash.

```
<center>
<img src="binarycodepixabay.jpg" width="140px" height="140px"/>
<p>940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81</p>
</center>
```
Comenzaremos investigando el posible hash, por lo que primero intentaremos averiguar el tipo de codificado que posee.
```
$ hash-identifier "940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81"
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
   --------------------------------------------------

   Possible Hashs:
   [+] SHA-256
   [+] Haval-256
```
Vemos que se trata de un codificación en **SHA-256**, por lo que haremos uso de un diccionario de palabras que se nos proporcionado llamado **easypeasy.txt**

Para averiguar esto utilizaremos la herramienta **JohnTheRipper** de la siguiente manera.
```
$ echo "940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81" > hash.txt
                                                              
$ john --w=easypeasy.txt --format=GOST hash.txt                                     
    Using default input encoding: UTF-8
    Loaded 1 password hash (gost, GOST R 34.11-94 [64/64])
    Will run 4 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    mypasswordforthatjob (?)
```
Por lo tanto obtendremos nuestra hash descodificado.
> mypasswordforthatjob
{: .prompt-info }

Para el siguiente paso utilizaremos la herramienta **Steghide** para así investigar la imagen que teniamos anteriormente a ver si posee algún tipo de información oculta en ella.
```
$ steghide --extract -sf matrix-3109795_960_720.jpg                   
Enter passphrase:
```
Nos pide una passphrase, por lo que introduciremos el hash descifrado que obtuvimos previamente, **mypasswordforthatjob**.
```
$ steghide --extract -sf matrix-3109795_960_720.jpg
Enter passphrase: 
wrote extracted data to "secrettext.txt".
```
Accederemos a nuestro fichero **secrettext.txt**, que contiene lo siguiente.
```
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
```
Tras ver el documentos, entendemos que debemos convertir estos datos binarios a texto por lo que realizaremos esto mediante una herramienta online.

![Captura de pantalla de 2022-12-14 13-17-20.png](Captura_de_pantalla_de_2022-12-14_13-17-20.png)
### Credenciales SSH
En este punto ya disponemos de nuestras credenciales para acceder por SSH.
> Username: boring
{: .prompt-info }
> Password: iconvertedmypasswordtobinary
{: .prompt-info }

Accedemos a través de SSH de la siguiente manera con nuestras credenciales obtenidas.
```
$ ssh -p 6498 boring@10.10.189.205
    boring@10.10.189.205's password:
    You Have 1 Minute Before AC-130 Starts Firing
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    !!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
    boring@kral4-PC:~$
```
Una vez en este punto, tendremos que listar los ficheros existentes en el directorio y visualizar su contenido.
```
boring@kral4-PC:~$ ls
user.txt
boring@kral4-PC:~$ cat user.txt
User Flag But It Seems Wrong Like It's Rotated Or Something
synt{a0jvgf33zfa0ez4y}
```
Tras investigar que formato de codificación podría tener, llegamos a la conclusión de que se trata de **ROT13**.

Por lo tanto, utilizaremos alguna herramienta online para descifrarlo.

![Captura de pantalla de 2022-12-14 13-27-39.png](Captura_de_pantalla_de_2022-12-14_13-27-39.png)
### Flag de usuario
Finalmente tendremo nuestra penúltima flag.
> flag{n0wits33msn0rm4l}
{: .prompt-info }

Aún quedaría la última flag, pero esto lo resolveremos más adelante.