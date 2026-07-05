# DockerLabs — DockerLabs.es 🐳

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.17.0.2 -oN escaneo.txt
```

Encontramos el **puerto 80** abierto.

Realizamos fuzzing web a la IP para ver qué directorios se encuentran disponibles. Entramos a la web y no encontramos nada interesante, tampoco en el código fuente.

Al entrar al directorio `/machine.php` nos encontramos con un **panel para subir archivos**.

---

## 2. Explotación

Creamos una reverse shell con la IP de nuestra máquina atacante usando **PHP PentestMonkey**, la guardamos como `shell.php` e intentamos subirla.

El servidor solo acepta archivos `.zip`, por lo que interceptamos el paquete con **BurpSuite** para cambiar la extensión:

1. Activamos **FoxyProxy** en el navegador
2. Abrimos BurpSuite → Proxy → activamos la interceptación
3. Enviamos el archivo `shell.php` y lo capturamos
4. Lo enviamos al **Intruder** (`Ctrl+I`)
5. Seleccionamos la palabra `php` del nombre del archivo → clic en **Add**
6. Tipo de payload: **Simple list** con distintas extensiones PHP

Al ejecutar el ataque, la extensión **`.phar`** responde con éxito: *"El archivo shell.phar ha sido subido correctamente"*.

Volvemos al Proxy, modificamos la extensión a `.phar` y hacemos **Forward**.

Una vez subido, nos ponemos en escucha con netcat:

```bash
nc -lvnp 443
```

Accedemos al directorio `/uploads` y hacemos clic en `shell.phar` → obtenemos acceso a la máquina.

Hacemos tratamiento de la TTY:

```bash
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
reset xterm

export TERM=xterm
export SHELL=bash
```

---

## 3. Escalada de Privilegios

Ejecutamos `sudo -l` para ver los binarios disponibles. No encontramos nada útil de inmediato.

Exploramos los directorios del sistema y en `/opt` encontramos un archivo `nota.txt` que nos indica la ruta `/root/clave.txt`.

Leemos el archivo usando el binario `grep`:

```bash
grep "" /root/clave.txt
```

Obtenemos la contraseña: `dockerlabsmolamogollon123`

La probamos con el usuario root y logramos escalar privilegios.

**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Fuzzing web
- File upload bypass (extensión `.phar` via BurpSuite Intruder)
- Reverse shell PHP
- TTY treatment
- Enumeración manual de directorios
- Lectura de archivos con `grep`
