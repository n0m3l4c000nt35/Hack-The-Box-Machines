# Ignition

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.202.124

PING 10.129.202.124 (10.129.202.124) 56(84) bytes of data.
64 bytes from 10.129.202.124: icmp_seq=1 ttl=63 time=163 ms

--- 10.129.202.124 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 162.744/162.744/162.744/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.202.124 -oG openPorts

PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 80 -sCV 10.129.202.124 -oN targeted

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://ignition.htb/
```

| Puerto | Servicio | Versión      |
| ------ | -------- | ------------ |
| 80     | http     | nginx 1.14.2 |

🗹 Ingresar al panel de logueo `/admin` y probar el usuario `admin` y la contraseña `qwerty123`.
## Flag

La flag se encuentra en el dashboard.
