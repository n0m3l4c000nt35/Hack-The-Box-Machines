🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.174.61

PING 10.129.174.61 (10.129.174.61) 56(84) bytes of data.
64 bytes from 10.129.174.61: icmp_seq=1 ttl=63 time=185 ms

--- 10.129.174.61 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 185.418/185.418/185.418/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.174.61 -oG openPorts

PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 80 -sCV 10.129.174.61 -oN targeted

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to nginx!
```

| Puerto | Servicio | Versión      |
| ------ | -------- | ------------ |
| 80     | http     | nginx 1.14.2 |
🗹 Escanear la web en busca de directorios o archivos con la extensión `php`:

```shell
wfuzz -c -t 20 --hc 404 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt http://10.129.147.6/FUZZ.php

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
=====================================================================

000000259:   200        31 L     66 W       999 Ch      "admin"
```

🗹 Ingresar a la web encontrada `http://10.129.147.6/admin.php`.

Probar las credenciales `admin` para el campo `username` y `admin` para el campo `password`.
## Flag

La flag se muestra en pantalla una vez se ingresa al panel de administración.