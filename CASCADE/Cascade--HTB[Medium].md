## Resumen
Cascade es una máquina Windows de dificultad media configurada como controlador de dominio. Las conexiones anónimas LDAP están habilitadas y la enumeración proporciona la contraseña del usuario r.thompson, que da acceso a una copia de seguridad del registro de TightVNC. La copia de seguridad se descifra para obtener la contraseña de s.smith. Este usuario tiene acceso a un ejecutable .NET que, tras la descompilación y el análisis del código fuente, revela la contraseña de la cuenta ArkSvc. Esta cuenta pertenece al grupo AD Recycle Bin y puede ver los objetos eliminados de Active Directory. Se encuentra que una de las cuentas de usuario eliminadas contiene una contraseña codificada, que puede reutilizarse para iniciar sesión como administrador principal del dominio.
### Herramientas
- Nmap
- SmbMap
- Smbclient
- Rpcclient
- Windapsearch
- Vncpwd
- Evil-winrm
- Wine
- Dnspy
- Python

## Enumeración

Hacemos un escaneo de puertos con nmap y vemos los típicos puertos de un sistema Windows

```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.30.166
```

![](/img/Pasted%20image%2020260508161647.png) 

En el escaneo de versión nos muestra que es un Windows Sever 2008 R2

```bash
sudo nmap -sCV 10.129.30.166
```

![](/img/Pasted%20image%2020260508163203.png)

Intentando listar los shares del servidor SMB como anónimo nos dice que tenemos el acceso denegado

![](/img/Pasted%20image%2020260508163256.png)


Entrando mediante rpc como anónimo nos dará acceso y podemos enumerar los usuarios que hay 

![](/img/Pasted%20image%2020260508163629.png)

También podemos usar la aplicación windapsearch (https://github.com/ropnop/windapsearch), con esta aplicación y las flags -U --full enumeraremos todos los usuarios y toda su información


```bash
git clone https://github.com/ropnop/windapsearch.git
sudo apt install python3-ldap
./windapsearch.py -U --full --dc-ip 10.129.30.166
```

Mirando cada usuario llegamos a r.thompson el cuál tiene un atributo extra que es cascadeLegacyPwd el cuál está codificado en base64

![](/img/Pasted%20image%2020260508164551.png)


## Explotación

Para decodificarlo lo haremos con el siguiente comando y nos saldrá una cadena "rY4an5eva"

```bash
 echo clk0bjVldmE= | base64 -d
```

![](/img/Pasted%20image%2020260508164825.png)

Intentando iniciar sesión mediante winrm no parece que tengamos acceso con esas credenciales. A lo que si parece que tenemos acceso es a los compartidos de SMB, en el cuál se puede ver un disco interesante que es "Data"

![](/img/Pasted%20image%2020260508165104.png)

```bash
smbmap -u r.thompson -p rY4n5eva -H 10.129.30.166
```

![](/img/Pasted%20image%2020260508165226.png)

Así que iniciaremos sesión en SMB en el compartido Data. Donde podemos encontrar los siguientes directorios los cuáles investigaremos para sacar información relevante, para ello nos descargaremos en nuestro sistema los archivos para movernos mejor en ellos, lo que podemos ver es que solo tnemo acceso a /IT

```bash
smbclient //10.129.30.166/Data -U r.thompson%rY4n5eva
```

![](/img/Pasted%20image%2020260508165437.png)

![](/img/Pasted%20image%2020260508165708.png)

Lo primero que podemos observar en lo que parece un correo es que hay un usuario TempAdmin el cuál tiene la contraseña del admin

![](/img/Pasted%20image%2020260508165928.png)
También encontraremos un log en el que se muestra el usuario ArkSvc llevando a la papelera de reciclaje al previo usuario TempAdmin

![](/img/Pasted%20image%2020260508170224.png)

Por último en el fichero VNC Install.reg de s.smith sobre el software TightVNC, encontramos una variable llamada password que está registrado en hexadecimal

![](/img/Pasted%20image%2020260508170349.png)

Para descifrar la contraseña usaremos xxd para revertir a binario el contenido, para ello usaremos la flag -r 

```bash
echo '6bcf2a4b6e5aca0f' | xxd -r -p > vnc_contraseña
```

Buscando en internet he dado con esta herramienta https://github.com/jeroennijhof/vncpwd la cuál nos podrá desencriptar la contraseña. La ejecutamos y nos sacará la contraseña de s.smith

![](/img/Pasted%20image%2020260508172118.png)

Probando las credenciales con evil-winrm nos dará acceso al sistema exitosamente como s.smith y conseguiremos la primera flag de la máquina

![](/img/Pasted%20image%2020260508172231.png)

![](/img/Pasted%20image%2020260508172331.png)


Viendo la información del usuario podemos ver que está en un grupo que se llama Audit Share que no suele ser típico

![](/img/Pasted%20image%2020260508172532.png) 


Observamos que el único miembro es él, también en el grupo de IT vemos otros usuarios ya vistos 

![](/img/Pasted%20image%2020260508172639.png) 

Así que veré a que tengo acceso en SMB con el usuario s.smith ya que tenemos las credenciales. Aquí podemos ver el acceso a Audit$ que viene en la información del grupo Audit Share. Así que le echaremos un vistazo

```bash
smbmap -u s.smith -p sT333ve2 -H 10.129.30.166
```

![](/img/Pasted%20image%2020260508172853.png) 

De la misma forma descargaremos todo lo que hay allí para sacar información importante

```bash
smbclient //10.129.30.166/Audit$ -U s.smith%sT333ve2
```

![](/img/Pasted%20image%2020260508173516.png)

Dentro del directorio DB podemos ver un fichero el cuál es para SQLite 3.x

![](/img/Pasted%20image%2020260508173825.png)

Mirando la información de las tablas vemos información del usuario ArkSvc, que intentando decodificar en base64 no sacaremos nada

![](/img/Pasted%20image%2020260508174159.png)

![](/img/Pasted%20image%2020260508174255.png)

Volviendo atrás mirando el fichero bat indicará que ejecuta CascAudit.exe con el fichero Audit.db visto ahora. Viendo CascAutit.exe se ve que es un ejecutable .net

![](/img/Pasted%20image%2020260508174701.png)


![](/img/Pasted%20image%2020260508174806.png)

Para decompilarlo usaremos esta aplicación https://github.com/dnSpy/dnSpy/releases, más usando wine para que funcione en linux. Allí abriremos el fichero .exe

>[!Disclaimer] 
>Si no funciona dnSpy al importar el fichero, probad a cambiar la configuración de wine poniéndolo en Windows 2008 R2 correspondiente a la máquina, con el siguiente comando:
>winecfg
> 

```bash
 unzip dnSpy-net-win64.zip
 wine dnSpy.exe
```

![](/img/Pasted%20image%2020260508181151.png) 


La clave del código está aquí, donde usa una clave encriptada la cuál servirá para conseguir la contraseña

![](/img/Pasted%20image%2020260508181514.png)


Podemos ver en un fichero CascCrypto de la forma que está encondeado en AES-128 y también sabemos el IV

![](/img/Pasted%20image%2020260508182158.png)

Para desencriptarlo haremos un programa en python con la librería pyaes que nos servirá para desencriptarlo con los parámetros que tenemos, la clave, el iv y la cadena en base64 vista en la base de datos del usuario ArkSvc

```
sudo apt install python3-pyaes
```

![](/img/Pasted%20image%2020260508192250.png)

Por último lo ejecutaremos con python y tendremos la cadena desencriptada, con lo que habremos conseguido la contraseña del usuario previamente visto ArkSvc 


![](/img/Pasted%20image%2020260508192329.png) 

## Escalado de privilegios

Ya que estamos dentro veremos la información del usuario y como habíamos visto antes podra borrar usuarios y llevarlos a la papelera de reciclaje, lo que quiero decir con esto es que está en un grupo llamado AD Recycle Bin y el usuario borrado TempAdmin tiene las credenciales del admin, así que podríamos acceder y recuperarla

![](/img/Pasted%20image%2020260508192822.png)


Para sacar la información de TempAdmin pondremos el siguiente comando, que sacará los datos del usuario. Básicamente estamos pidiendo del Active Directory una cuenta llamada "TempAdmin" incluyendo objetos borrados más todas sus propiedades. Veremos el atributo cascadeLegacyPwd que como ya sabíamos es la propia contraseña codificada en base64


```powershell
Get-ADObject -filter { SAMAccountName -eq "TempAdmin" } -includeDeletedObjects -property *
```

![](/img/Pasted%20image%2020260508193215.png)

Así que decodificando en base64 conseguiremos una cadena que será la contraseña del admin

```bash
echo 'YmFDVDNyMWFOMDBkbGVz' | base64 -d
```

![](/img/Pasted%20image%2020260508193322.png)

Para finalizar simplemente iniciaremos sesión con evil-winrm con el usuario Administrator y la contraseña, con el inicio de sesión exitoso concluiremos la máquina encontrando la flag de root en su escritorio

![](/img/Pasted%20image%2020260508193559.png)