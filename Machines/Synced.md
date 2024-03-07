# Synced

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.144.8

PING 10.129.144.8 (10.129.144.8) 56(84) bytes of data.
64 bytes from 10.129.144.8: icmp_seq=1 ttl=63 time=166 ms

--- 10.129.144.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 166.092/166.092/166.092/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.144.8 -oG openPorts

PORT    STATE SERVICE REASON
873/tcp open  rsync   syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 873 -sCV 10.129.144.8 -oN targeted

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
```

| Puerto | Servicio | Versión |
| ------ | -------- | ------- |
| 873    | rsync    | 31      |

¿Qué es rsync?  
[RFC](https://www.rfc-editor.org/rfc/rfc5781.html)  
[https://rsync.samba.org/](https://rsync.samba.org/)

🗹 Conectarse al servidor mediante **rsync**:

```shell
rsync 10.129.144.8::

public         	Anonymous Share
```

🗹 Acceder a los diferentes recursos hasta obtener la flag:

```shell
rsync 10.129.144.8::public

drwxr-xr-x          4,096 2022/10/24 19:02:23 .
-rw-r--r--             33 2022/10/24 18:32:03 flag.txt
```
## Flag

La flag se encuentra en el directorio `public`, con el punto se indica que se va a guardar una copia en el directorio actual de trabajo:

```shell
rsync 10.129.144.8::public/flag.txt .

cat flag.txt
```
