## Resumen
Help es un sistema Easy Linux con un punto final GraphQL que permite obtener credenciales para un software de soporte técnico. Este software es vulnerable a la inyección SQL ciega, la cual puede explotarse para obtener la contraseña de inicio de sesión SSH. Alternativamente, se puede explotar la carga de archivos arbitrarios sin autenticación para obtener ejecución remota de código (RCE). Posteriormente, se descubre que el kernel es vulnerable y puede explotarse para obtener acceso de superusuario (root).


### Herramientas



## Enumeración

Realizaremos un escaneo de puertos rápido hacía la máquina
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.32.141
```

![](/img/Pasted%20image%2020260512163545.png)

Seguidamente haremos el escaneo de versiones a los puertos que hemos encontrado. Al comprobar el puerto 80 nos redirigirá a un dns llamado help.htb
```bash
sudo nmap -sCV -p22,80,3000 10.129.32.141
```

![](/img/Pasted%20image%2020260512163639.png)

![](/img/Pasted%20image%2020260512163713.png)

Añadimos la línea en el fichero hosts para tener acceso

```bash
sudo nano /etc/hosts

10.129.32.141   help.htb
```

Allí vemos que es una página Apache default
![](/img/Pasted%20image%2020260512163922.png)

Así que haremos fuzzin por si encontramos algun directorio interesante, podemos encontrar support en el que tenemos acceso

```bash
gobuster dir --url http://help.htb/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt -t 50
```


![](/img/Pasted%20image%2020260512164025.png)


Al entrar veremos una página web con el software HelpDeskZ

![](/img/Pasted%20image%2020260512164039.png)

Por lo tanto haremos fuzzing de nuevo al presente directorio, para encontrar ficheros y más caminos donde encontrar información. Allí veremos un fichero UPGRADING.txt
```bash
gobuster dir --url http://help.htb/support/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,txt
```

![](/img/Pasted%20image%2020260512165527.png)

Al comprobarlo nos dirá la versión del software la 1.0.2
![](/img/Pasted%20image%2020260512165208.png)


### Node

Al entrar en el puerto 3000 de Node nos saldrá un mensaje en formato json, que nos da una pista de que tenemos que encontrar las credenciales en una consulta

![](Pasted%20image%2020260512164535.png)

Mirando la tecnología que usa podemos observar que usa Express

![](Pasted%20image%2020260512165657.png)


Buscando en internet vemos que Express suele usar graphql para las query, las cuales queremos saber, también he encontrado que el endpoint está en /graphql. Pero al ponerlo nos dice que falta la consulta 

![](Pasted%20image%2020260512170433.png)


Podemos hacerle una petición intentando pedir el usuario, nos dará error y nos dice que user tiene subcampos que debemos seleccionar, así que vamos a probarlos
```bash
 curl -s -G http://help.htb:3000/graphql --data-urlencode "query={user}" | jq
```

![](Pasted%20image%2020260512171144.png)

Probé con user, pero me dijo que no existía, si quería decir username y al probarlo con username nos dará un correo, ahora probaremos si hay algún tipo de contraseña

```bash
curl -s -G http://help.htb:3000/graphql --data-urlencode "query={user { username }}" | jq
```

![](Pasted%20image%2020260512171306.png)

Al igual que antes, probando pass nos dirá si queríamos poner password, poniéndolo nos saldrá una contraseña encodeada.

```bash
curl -s -G http://help.htb:3000/graphql --data-urlencode "query={user { password }}" | jq
```

![](Pasted%20image%2020260512171442.png)


En la página https://crackstation.net/ crackearemos la contraseña encodeada en md5, el resultado saldrá "godhelpmeplz"


![](Pasted%20image%2020260512171623.png) 


## Explotación

Una vez sabemos esas credenciales vamos probar a usarlas en el puerto 80 en el cuál teníamos el software Help Desk. Y habremos conseguido acceso exitosamente

![](Pasted%20image%2020260512171910.png)

Buscando información sobre la versión del software visto antes la 1.0.2 con searchsploit veremos que hay disponibles 2 exploits

![](Pasted%20image%2020260512172026.png)

Ya que estamos autenticados probaré a usar el siguiente https://www.exploit-db.com/exploits/41200, para usarlo debemos crear un ticket nuevo, rellenamos todos los campos obligatorios como sea y añadir una imagen a la opción de attachments, una vez rellenado todo lo entregramos

![](Pasted%20image%2020260512173508.png)

No me ha dado resultados, así que lo haré de otra forma con Burpsuite, una vez hemos hecho la petición mientras escuchamos con Burpsuite abriremos el enlace del attachment en una pestaña nueva

![](Pasted%20image%2020260512174703.png)

Al hacer eso nos llegará una petición hacia dónde está la imagen, como sabemos que tiene SQLi lo pasaremos por la aplicación sqlmap

![](Pasted%20image%2020260512174753.png)

Guardaremos la petición en un fichero y después lo usaremos con sqlmap. Al usarlo empezará a testear consultas, tras un breve tiempo dará una tabla llamada Staff en la que estará disponible 
el login del admin y su contraseña, la cuál saldrá crackeada gracias a sqlmap, la cuál será "Welcome1" 


![](Pasted%20image%2020260512174834.png)

```bash
sqlmap -r sqliHelp.txt --dump
```


![](Pasted%20image%2020260512183932.png)

Así que haciendo ssh al sistema con el usuario help estaremos dentro

![](Pasted%20image%2020260512184513.png)

Dentro podremos encontrar la primera flag de user.txt 

![](Pasted%20image%2020260512184609.png)


## Escalado de privilegios

Comprobando la versión de Linux que hay vemos que usa una versión de kernel antigua, la cuál investigando posibles exploits encontré este https://www.exploit-db.com/exploits/44298 


![](Pasted%20image%2020260512185518.png)

Así que debemos descargarlo e instalarlo mediante un servidor python que creemos, seguidamente compilaremos el programa y lo ejecutaremos. Por lo que terminaremos escalando privilegios como root exitosamente

```bash
gcc -o exploit 44298.c
```

![](Pasted%20image%2020260512185734.png)

Para concluir leeremos la última flag de root y habremos completado al máquina

![](Pasted%20image%2020260512185831.png)