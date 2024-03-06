🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.163.99

PING 10.129.163.99 (10.129.163.99) 56(84) bytes of data.
64 bytes from 10.129.163.99: icmp_seq=1 ttl=63 time=169 ms

--- 10.129.163.99 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 168.735/168.735/168.735/0.000 ms
```

El sistema operativo identificado es **Linux**.

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.163.99 -oG openPorts

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -sCV -p 21 10.129.163.99 -oN targeted

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.71
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
Service Info: OS: Unix
```

| Puertos | Servicios | Versión      |
| ------- | --------- | ------------ |
| 21      | ftp       | vsftpd 3.0.3 |
¿Qué es FTP?
[RFC](https://www.rfc-es.org/rfc/rfc0959-es.txt)

🗹 Conectarse al servidor mediante el protocolo FTP con el usuario **anonymous** y sin ingresar contraseña:

```shell
ftp ftp://anonymous:@10.129.163.99

Connected to 10.129.163.99.
220 (vsFTPd 3.0.3)
331 Please specify the password.
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
200 Switching to Binary mode.
ftp>
```

### Root flag

La **flag** se encuentra en el directorio principal:

`more flag.txt`
## Cositas aprendidas

✅ Configurar el servicio FTP:
- Instalar el servidor FTP **vsftpd** (Very Secure FTP Daemon): `sudo apt install vsftpd`.
- Configurar el servidor FTP modificando el archivo `/etc/vsftpd.conf`:

```shell
anonymous_enable=YES
anon_root=/
```

- Habilitar el servicio `vsftpd` con el comando `systemctl start vsftpd`.
- Si hacemos una actualización a la configuración del archivo `/etc/vsftpd` reiniciar el servicio con el comando `systemctl restart vsftpd`.