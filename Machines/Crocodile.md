🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.233.227

PING 10.129.233.227 (10.129.233.227) 56(84) bytes of data.
64 bytes from 10.129.233.227: icmp_seq=1 ttl=63 time=165 ms

--- 10.129.233.227 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 164.643/164.643/164.643/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.233.227 -oG openPorts

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 21,80 -sCV 10.129.233.227 -oN targeted

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.193
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Smash - Bootstrap Business Template
Service Info: OS: Unix
```

| Puerto | Servicio | Versión             |
| ------ | -------- | ------------------- |
| 21     | ftp      | vsftpd 3.0.3        |
| 80     | http     | Apache httpd 2.4.41 |

🗹 Conectarse al servicio **ftp** con el usuario **anonymous** y descargar los archivos que contienen una lista de usuarios y otras de contraseñas:

```shell
ftp ftp://anonymous:@10.129.233.227

Connected to 10.129.233.227.
220 (vsFTPd 3.0.3)
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.
ftp> dir
229 Entering Extended Passive Mode (|||44863|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||43903|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
100% |*******************************************************************************************************************************************|    33      528.30 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.18 KiB/s)
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||47781|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
100% |*******************************************************************************************************************************************|    62      796.66 KiB/s    00:00 ETA
226 Transfer complete.
62 bytes received in 00:00 (0.35 KiB/s)
```

🗹 Verificar si existe una página de `login.php`.

🗹 Interceptar la petición `POST /login.php` con **Burpsuite**. Enviar la petición al intruder. Realizar un ataque de tipo **Cluster Bomb** usando las listas descargas del servidor. Verificar los resultados del ataque buscando en el status code o length para dar con los datos correctos.

🗹 Ingresar el usuario y contraseña encontrados en la página de `login.php`.
## Flag

La flag se encuentra en el dashboard.