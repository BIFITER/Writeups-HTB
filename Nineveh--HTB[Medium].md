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



![](Pasted%20image%2020260423175051.png) 

#### Enumeración HTTPS 

```bash 
gobuster dir --url https://10.129.23.159/ --wordlist /usr/share/wordlists/DirectoryDiscovery/common.txt --no-tls-validation
```

![](Pasted%20image%2020260423175642.png) 


![](Pasted%20image%2020260423175747.png) 

## Explotación


![](Pasted%20image%2020260423175849.png)