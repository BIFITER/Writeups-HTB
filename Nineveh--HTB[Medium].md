### Enumeración y escaneo
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.23.159  
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-23 17:45 +0200
Nmap scan report for 10.129.23.159
Host is up (0.067s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https
```


![HTTP](Pasted%20image%2020260423174644.png)

![HTTPS](Pasted%20image%2020260423174757.png) 



#### Enumeración HTTP

```bash
gobuster dir --url http://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt
```

![](Pasted%20image%2020260423175008.png) 

```bash
gobuster dir --url http://10.129.23.159/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --no-tls-validation
```
![](Pasted%20image%2020260423182707.png)


![](Pasted%20image%2020260423175051.png) 

![](Pasted%20image%2020260423182723.png)
#### Enumeración HTTPS 

```bash 
gobuster dir --url https://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt --no-tls-validation
```

![](Pasted%20image%2020260423175642.png) 


![](Pasted%20image%2020260423175747.png) 

## Explotación
### HTTPS
Ya que es un simple login con solo el input de la contraseña le haremos fuerza bruta con hydra fácilmente, ya que es https hay que especificar los parámetros para que concluya el login.

```bash
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.129.23.159  https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password" -t 64 -V
```

![](Pasted%20image%2020260423180737.png)


![](Pasted%20image%2020260423181116.png)

Investigamos y vemos que tiene varios exploits, procederé a usar este https://www.exploit-db.com/exploits/24044

![](Pasted%20image%2020260423175849.png) 

Para el exploit debemos crear una base de datos nueva en mi caso la llamaré hack.php
![](Pasted%20image%2020260423181228.png)

Una vez creado debemos hacer una tabla nueva
![](Pasted%20image%2020260423181506.png)

![](Pasted%20image%2020260423181815.png) 


Vemos que la base de datos está en el directorio /var/tmp
![](Pasted%20image%2020260423181840.png) 

### HTTP
Mediante una pequeña trampa podemos saber que el usuario que acepta te indica si es válido o no 
![](Pasted%20image%2020260423182953.png) 

![](Pasted%20image%2020260423183028.png) 

Así que haciendo fuerza bruta al login intentaremos averiguar la contraseña.
```bash
hydra -l none -P /usr/share/wordlists/rockyou.txt 10.129.23.159  http-post-form "/department/login.php:username=admin&password=^PASS^:Invalid Password" -t 64 -V
``` 


![](Pasted%20image%2020260423183147.png) 

Una vez dentro buscaremos posibles vectores de ataque
![](Pasted%20image%2020260423183250.png) 

Al darle al apartado de Notas veremos que tenemos una vulnerabilidad de LFI la cuál explotaremos para escuchar el payload previamente creado en la base de datos

![](Pasted%20image%2020260423183352.png) 


Ya que sabíamos previamente donde está la base de datos porque lo habíamos visto procederemos ponerlo.
![](Pasted%20image%2020260423183506.png) 

