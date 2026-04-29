## Escaneo y enumeración


```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.26.133
```


![](Pasted%20image%2020260429150953.png) 
 


Hacemos un escaneo de versión y aparece que en el puerto 3000 usa Apache Hadoop, es un framework de software de código abierto que permite el almacenamiento y procesamiento distribuido de grandes volúmenes de datos, todo apunta a una aplicación web.
```bash
sudo nmap -sCV -p22,3000 10.129.26.133
```


![](Pasted%20image%2020260429151208.png) 


Comprobamos la ip en la web y efectivamente estamos ante una aplicación, la cual procederemos a la enumeración de ésta

![](Pasted%20image%2020260429151353.png) 


Al hacer un gobuster simple nos dará error ya que según que frontend trabaje, porque está usando el puerto 3000 que esto a punta a Node.js ya que así se llama la máquina, redirige todo al index.html ya que es un SPA (Single Page Application) de Node. 

![](Pasted%20image%2020260429152830.png) 


Haciendo una investigación de la web interna mediante las herramientas de desarrollador he visto que hace peticiones a distintos ficheros javascript.

![](Pasted%20image%2020260429155540.png) 

Al investigar el código podemos observar que hace una petición al directorio /api/users directorio que no hemos encontrado antes, vamos a ver si podemos tener acceso a él poniéndolo en la url
 
![](Pasted%20image%2020260429155755.png)

Y gracias a esa manera habremos encontrado la información de todos los usuarios de la web incluyendo el admin. Ahora debemos crackear la contraseña para conseguir acceso inicial como administrador.
![](Pasted%20image%2020260429155901.png)

Desde la herramienta web crackstation.net pongo el hash del tipo sha256 dado y tendremos acceso a la contraseña del admin exitosamente
![](Pasted%20image%2020260429160254.png)

Así que introduciremos las credenciales vistas en el login de la página, lo que nos dará acceso a siguiente mensaje y a poder descargar un backup que nos será necesario.

![](Pasted%20image%2020260429160355.png)


![](Pasted%20image%2020260429160448.png) 

## Explotación

Al mirar dicho fichero podemos observar que parece ser base64 así que debemos decodificarlo para sacar información

![](Pasted%20image%2020260429160809.png) 

Lo decodificaremos con el siguiente comando, al pasarle el comando file para comprobar que tipo de fichero es veremos que es un archivo Zip

```bash
base64 -d myplace.backup > backupNice
```

![](Pasted%20image%2020260429161127.png) 

El primer problema al que nos enfrentamos intentado descomprimirlo es que nos pedirá una contraseña 

![](Pasted%20image%2020260429161727.png) 

Con lo que usaremos la herramienta fcrackzip para averiguar dicha contraseña y tener el acceso de la información del archivo zip. En poco tiempo averiguaremos la contraseña 

```bash
fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u backupNice 
```

![](Pasted%20image%2020260429161910.png)


Ponemos al contraseña y se habrá descomprimido correctamente todo el contenido, ahora debemos investigar y encontrar alguna información de valor

![](Pasted%20image%2020260429162205.png) 

Viendo el fichero app.js podemos observar unas credenciales de mark para mongodb

![](Pasted%20image%2020260429162507.png) 

## Acceso

Las probaremos en ssh y ya tenemos acceso a la víctima como mark

![](Pasted%20image%2020260429162638.png)

![](Pasted%20image%2020260429162657.png)


Investigando los procesos que hay activos podemos ver un servicio que está siendo usado por Tom

![](Pasted%20image%2020260429162948.png)

## Escalada de privilegios 
Investigando el fichero app.js, podemos observar que mark reutiliza las credenciales de mongodb para acceder a la base de datos scheduler. A la misma vez vemos que podemos ejecutar cmd con la línea exec(doc.cmd)

![](Pasted%20image%2020260429163713.png)

Entrado con el siguiente comando a la base de datos tendremos acceso a ella, como habíamos visto podemos ejecutar cmd, así que aprovecharemos esto para conseguir un terminal bash de Tom

```bash
mongo -p -u mark scheduler
```

Con la siguiente línea haremos una copia de bash y que sea propietario tom

```mongodb
db.tasks.insert({"cmd":"/bin/cp /bin/bash /tmp/tom; /bin/chown tom:admin /tmp/tom; chmod g+s /tmp/tom; chmod u+s /tmp/tom"});
```

Una vez hecho debemos ir a donde creamos el fichero en este caso /tmp/tom y ejecutarlo. Simplemente con ./tom -p nos dará acceso a una shell como  tom y todos sus privilegios que nos harán falta

```bash
./tom -p
```

![](Pasted%20image%2020260429164354.png) 

Para la escalda de privilegios he investigado el SUID que tenemos y veo que tengo acceso a /usr/local/bin/backup el cual debemos usar para escalar

![](Pasted%20image%2020260429164755.png)

Recordando el fichero app.js encontrábamos una key para los backups, el cuál seguramente nesitaremos

![](Pasted%20image%2020260429165007.png) 

