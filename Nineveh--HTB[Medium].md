### Enumeración y escaneo
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.23.159  
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-23 17:45 +0200
Nmap scan report for 10.129.23.159
Host is up (0.067s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https
```


![HTTP](Pasted%20image%2020260423174644.png)

![HTTPS](Pasted%20image%2020260423174757.png) 



#### Enumeración HTTP

```bash
gobuster dir --url http://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt
```

![](Pasted%20image%2020260423175008.png) 

```bash
gobuster dir --url http://10.129.23.159/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --no-tls-validation
```
![](Pasted%20image%2020260423182707.png)


![](Pasted%20image%2020260423175051.png) 

![](Pasted%20image%2020260423182723.png)
#### Enumeración HTTPS 

```bash 
gobuster dir --url https://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt --no-tls-validation
```

![](Pasted%20image%2020260423175642.png) 


![](Pasted%20image%2020260423175747.png) 

## Explotación
### HTTPS
Ya que es un simple login con solo el input de la contraseña le haremos fuerza bruta con hydra fácilmente, ya que es https hay que especificar los parámetros para que concluya el login.

```bash
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.129.23.159  https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password" -t 64 -V
```

![](Pasted%20image%2020260423180737.png)


![](Pasted%20image%2020260423181116.png)

Investigamos y vemos que tiene varios exploits, procederé a usar este https://www.exploit-db.com/exploits/24044

![](Pasted%20image%2020260423175849.png) 

Para el exploit debemos renonmbrar la base de datos, en mi caso la llamaré ninevehNotes.php
![](Pasted%20image%2020260423190525.png)

Una vez creado debemos hacer una tabla nueva 

![](Pasted%20image%2020260423181506.png)


![](Pasted%20image%2020260423190633.png)


Vemos que la base de datos está en el directorio /var/tmp
![](Pasted%20image%2020260423190525.png) 

### HTTP
Mediante una pequeña trampa podemos saber que el usuario que acepta te indica si es válido o no 
![](Pasted%20image%2020260423182953.png) 

![](Pasted%20image%2020260423183028.png) 

Así que haciendo fuerza bruta al login intentaremos averiguar la contraseña.
```bash
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.129.23.159  http-post-form "/department/login.php:username=admin&password=^PASS^:Invalid Password" -t 64 -V
``` 


![](Pasted%20image%2020260423183147.png) 

Una vez dentro buscaremos posibles vectores de ataque
![](Pasted%20image%2020260423183250.png) 

Al darle al apartado de Notas veremos que tenemos una vulnerabilidad de LFI la cuál explotaremos para escuchar el payload previamente creado en la base de datos

![](Pasted%20image%2020260423183352.png) 


Ya que sabíamos previamente donde está la base de datos porque lo habíamos visto procederemos ponerlo.  Y probamos el exploit y veremos que correctamente nos ha funcionado. Ahora procederemos a conseguir una reverse shell en nuestro sistema.
 ![](Pasted%20image%2020260423190738.png)

Pillamos la petición mediante burpsuite y cambiamos el método de esta a POST
![](Pasted%20image%2020260423191114.png) 

Y lo cambiamos para tener iniciar la reverse shell, enviamos y esperamos. 

![](Pasted%20image%2020260423191403.png) 

Habremos entrado al sistema exitosamente

![](Pasted%20image%2020260423191513.png) 

Investigamos las carpetas y entramos a ssl 

![](Pasted%20image%2020260423191742.png)

Entramos a secure_notes y vemos un png en el cual hacemos el comando strings, lo cuál nos muestra una private key de ssh del usuario amorois. 

![](Pasted%20image%2020260423192016.png)

Sabemos que no tenía abierto ssh en el  escaneo previo así que investigo sus puertos abiertos y descubro que tiene un puerto 22 abierto
![](Pasted%20image%2020260423192233.png) 

Como no tiene ssh veremos los programas que tiene desde /etc/init.d y comprobamos que está usando knockd que es un programa que sirve para ocultar algún puerto para mayor seguridad.
![](Pasted%20image%2020260423192747.png) 

En el fichero propio de configuración podemos comprobar que tiene bloqueado el puerto 22, así que debemos jugar con la secuencia dada para abrir el puerto.
![](Pasted%20image%2020260423193024.png) 

Usaremos la propia herramienta y simplemente poniendo la secuencia podremos comprobar que se ha abierto el puerto 22 del ssh.
![](Pasted%20image%2020260423193434.png) 


Así que ahora simplemente con la clave privada previamente copiada la usaremos para iniciar sesión en ssh con el usuario amrois

![](Pasted%20image%2020260423193647.png) 

Ya dentro podemos acceder a la primera flag de user.txt
![](Pasted%20image%2020260423193958.png)

Comprobando el crontab veremos que hay un proceso ejecutandose por el root llamado report-reset.sh

![](Pasted%20image%2020260423194045.png) 

Así que lo configuraremos para escalar privilegios como root

![](Pasted%20image%2020260423194448.png) 

