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


