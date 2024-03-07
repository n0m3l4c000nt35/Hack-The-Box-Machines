# Dancing

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.165.154

PING 10.129.165.154 (10.129.165.154) 56(84) bytes of data.
64 bytes from 10.129.165.154: icmp_seq=1 ttl=127 time=168 ms

--- 10.129.165.154 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 167.795/167.795/167.795/0.000 ms
```

El sistema operativo identificado es **Windows**.

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.165.154 -oG openPorts

PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack ttl 127
139/tcp   open  netbios-ssn  syn-ack ttl 127
445/tcp   open  microsoft-ds syn-ack ttl 127
5985/tcp  open  wsman        syn-ack ttl 127
47001/tcp open  winrm        syn-ack ttl 127
49664/tcp open  unknown      syn-ack ttl 127
49665/tcp open  unknown      syn-ack ttl 127
49666/tcp open  unknown      syn-ack ttl 127
49667/tcp open  unknown      syn-ack ttl 127
49668/tcp open  unknown      syn-ack ttl 127
49669/tcp open  unknown      syn-ack ttl 127
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 135,139,445,5985,47001,49664,49665,49666,49667,49668,49669 -sCV 10.129.165.154 -oN targeted

PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 4h00m01s
| smb2-time: 
|   date: 2024-03-07T01:18:39
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

| Puerto | Servicio     | Versión                                 |
| ------ | ------------ | --------------------------------------- |
| 135    | msrpc        | Microsoft Windows RPC                   |
| 139    | netbios-ssn  | Microsoft Windows netbios-ssn           |
| 445    | microsoft-ds | ?                                       |
| 5985   | wsman        | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP) |
| 47001  | winrm        | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP) |
| 49664  | unknown      | Microsoft Windows RPC                   |
| 49665  | unknown      | Microsoft Windows RPC                   |
| 49666  | unknown      | Microsoft Windows RPC                   |
| 49667  | unknown      | Microsoft Windows RPC                   |
| 49668  | unknown      | Microsoft Windows RPC                   |
| 49669  | unknown      | Microsoft Windows RPC                   |
¿Qué es netbios?
[RFC](https://www.rfc-editor.org/rfc/rfc1002.txt)

🗹 Listar servicios disponibles en el servidor sin proporcionar contraseña:

```shell
smbclient --no-pass -L //10.129.165.154

Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk      
```

## Flag:

La flag se encuentra en el directorio `/`
