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

## Explotación


## Escalado de privilegios