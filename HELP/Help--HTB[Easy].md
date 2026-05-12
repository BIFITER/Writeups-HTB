## Resumen
Help es un sistema Easy Linux con un punto final GraphQL que permite obtener credenciales para un software de soporte técnico. Este software es vulnerable a la inyección SQL ciega, la cual puede explotarse para obtener la contraseña de inicio de sesión SSH. Alternativamente, se puede explotar la carga de archivos arbitrarios sin autenticación para obtener ejecución remota de código (RCE). Posteriormente, se descubre que el kernel es vulnerable y puede explotarse para obtener acceso de superusuario (root).


### Herramientas


## Enumeración

```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.32.141
```

![](Pasted%20image%2020260512163545.png)

```bash
sudo nmap -sCV -p22,80,3000 10.129.32.141
```

![](Pasted%20image%2020260512163639.png)

![](Pasted%20image%2020260512163713.png)

Añadimos la línea en el fichero hosts

```bash
sudo nano /etc/hosts

10.129.32.141   help.htb
```


![](Pasted%20image%2020260512163922.png)



```bash
gobuster dir --url http://help.htb/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt -t 50
```


![](Pasted%20image%2020260512164025.png)



![](Pasted%20image%2020260512164039.png)


```bash
gobuster dir --url http://help.htb/support/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt -t 50
```


De momento no encontramos nada de valor, además de la mayoría de directorios nos redireccionan a el index principal, así que pasaremos a investigar el puerto 3000
![](Pasted%20image%2020260512164156.png)

![](Pasted%20image%2020260512164535.png)
## Explotación


## Escalado de privilegios

