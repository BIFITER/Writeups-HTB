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
![](Pasted%20image%2020260424162705.png)