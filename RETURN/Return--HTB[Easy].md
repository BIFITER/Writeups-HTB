## Resumen

## Herramientas



## Enumeración

Empezamos haciendo un escaneo de puertos completo y se puede observar la estructura que suelen tener los sistemas Windows. Hay puertos interesantes como el 80, 445... 
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.95.241
```

![](Pasted%20image%2020260507152904.png) 

En el escaneo de versiones podemos ver algo interesante como que el Host se llama PRINTER, lo que apunta a haber algún servicio de impresora

```bash
sudo nmap -sCV 10.129.95.241
```

![](Pasted%20image%2020260507153242.png)

Intentado entrar como anónimo en el servicio de SMB nos deniega el acceso, así que de momento no tenemos nada que hacer son esto

```bash
smbmap -u Guest -p "" -H 10.129.95.241
```

![](Pasted%20image%2020260507153405.png)

Entrando al servicio web del puerto 80 podemos ver una página de impresoras, además de que estamos en el Admin Panel, por lo que veremos hasta dónde podemos llegar con este privilegio


![](Pasted%20image%2020260507153529.png)


Entrando al apartado "Settings" encontramos un usuario, el puerto por el que está el servicio de impresora, lo cuál seguramente sirva  

![](Pasted%20image%2020260507153718.png)

Haciendo fuzzing no encuentro nada fuera de lo común así que no podemos mirar más desde la web, así que pasaremos a ver que hay en ese puerto del server

![](Pasted%20image%2020260507154059.png)




## Explotación


## Escalada de privilegios

