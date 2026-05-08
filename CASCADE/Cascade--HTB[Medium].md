## Resumen

### Herramientas


## Enumeración

Hacemos un escaneo de puertos con nmap y vemos los típicos puertos de un sistema Windows

```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.30.166
```

![](Pasted%20image%2020260508161647.png) 

En el escaneo de versión nos muestra que es un Windows Sever 2008 R2

```bash
sudo nmap -sCV 10.129.30.166
```

![](Pasted%20image%2020260508163203.png)

Intentando listar los shares del servidor SMB como anónimo nos dice que tenemos el acceso denegado

![](Pasted%20image%2020260508163256.png)


Entrando mediante rpc como anónimo nos dará acceso y podemos enumerar los usuarios que hay 

![](Pasted%20image%2020260508163629.png)

También podemos usar la aplicación windapsearch (https://github.com/ropnop/windapsearch), con esta aplicación y las flags -U --full enumeraremos todos los usuarios y toda su información


```bash
git clone https://github.com/ropnop/windapsearch.git
sudo apt install python3-ldap
./windapsearch.py -U --full --dc-ip 10.129.30.166
```

Mirando cada usuario llegamos a r.thompson el cuál tiene un atributo extra que es cascadeLegacyPwd el cuál está codificado en base64

![](Pasted%20image%2020260508164551.png)


## Explotación

Para decodificarlo lo haremos con el siguiente comando y nos saldrá una cadena "rY4an5eva"

```bash
 echo clk0bjVldmE= | base64 -d
```

![](Pasted%20image%2020260508164825.png)

Intentando iniciar sesión mediante winrm no parece que tengamos acceso con esas credenciales. A lo que si parece que tenemos acceso es a los compartidos de SMB, en el cuál se puede ver un disco interesante que es "Data"

![](Pasted%20image%2020260508165104.png)

```bash
smbmap -u r.thompson -p rY4n5eva -H 10.129.30.166
```
![](Pasted%20image%2020260508165226.png)

Así que iniciaremos sesión en SMB en el compartido Data. Donde podemos encontrar los siguientes directorios los cuáles investigaremos para sacar información relevante, para ello nos descargaremos en nuestro sistema los archivos para movernos mejor en ellos, lo que podemos ver es que solo tnemo acceso a /IT

```bash
smbclient //10.129.30.166/Data -U r.thompson%rY4n5eva
```

![](Pasted%20image%2020260508165437.png)

![](Pasted%20image%2020260508165708.png)

Lo primero que podemos observar en lo que parece un correo es que hay un usuario TempAdmin el cuál tiene la contraseña del admin

![](Pasted%20image%2020260508165928.png)
También encontraremos un log en el que se muestra el usuario ArkSvc llevando a la papelera de reciclaje al previo usuario TempAdmin

![](Pasted%20image%2020260508170224.png)

Por último en el fichero VNC Install.reg de s.smith sobre el software TightVNC, encontramos una variable llamada password que está registrado en hexadecimal


![](Pasted%20image%2020260508170349.png)




## Escalado de privilegios

