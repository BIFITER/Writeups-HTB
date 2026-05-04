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

Después de buscar entre los archivos encontramos un fichero xml "Groups.xml" en el cual se pueden ver lo que parecen ser credenciales, con una contraseña cifrada y el usuario del grupo SVC_TGS


![](Pasted%20image%2020260504162704.png)



## Explotación

## Escalada de privilegios

