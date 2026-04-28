## Resumen



## Escaneo y enumeración

Comenzaré con un nmap rápido para ver que puertos tiene abiertos, vemos que está el 22, 25, 80, 110, 119 y 4555 abiertos

```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.25.254
```


![](Pasted%20image%2020260428172231.png) 

Dado que no es mucha información procedo a hacer un escaneo de versión para mejor entendimiento de los puertos. 

```bash
sudo nmap -sV -p22,25,80,110,119,4555 10.129.25.254
``` 
