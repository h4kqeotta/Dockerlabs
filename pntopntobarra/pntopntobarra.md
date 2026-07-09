# Pntopntobarra — DockerLabs.es 🐳

<img width="553" height="303" alt="1" src="https://github.com/user-attachments/assets/ee0582db-c243-4df4-833e-917c5ff1cf84" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.17.0.2 -oN escaneo.txt
```

<img width="604" height="308" alt="2" src="https://github.com/user-attachments/assets/6c543039-0021-4be1-b245-0939df9c2b6f" />

Vemos que tenemos el puerto 80 abierto.

Vamos a la web a ver qué contiene…

<img width="1342" height="556" alt="3" src="https://github.com/user-attachments/assets/d6475416-ebb5-4051-be51-5fe84d24219d" />

Si hacemos click en “Ejemplos de computadoras infectadas” nos llevará a una ruta la cuál almacena imagenes supuestamente

<img width="1128" height="564" alt="4" src="https://github.com/user-attachments/assets/fbf28b2e-4112-435c-8ef0-96890736ed1b" />



---

## 2. Explotación

Como vemos en la ruta… se está llamando a una imagen en concreto, pero si nosotros le decimos que en vez de mostrarnos una foto nos muestre el fichero passwd pasa lo siguiente:

<img width="1351" height="510" alt="5" src="https://github.com/user-attachments/assets/0401451e-d643-45fe-8974-4696887fd437" />

Abrimos el código fuente para ver el contenido con más claridad y encontramos el usuario “nico”

<img width="835" height="561" alt="6" src="https://github.com/user-attachments/assets/f8f4342b-b6d1-412c-ae6f-e4977c5e9d22" />

De la misma forma podemos averiguar si podemos acceder al fichero donde se almacena el id_rsa del usuario “nico” de la siguiente manera:

<img width="886" height="582" alt="7" src="https://github.com/user-attachments/assets/22b40402-ef7b-4f17-b043-aa1cc852bfbd" />

Luego copiamos todo este código y lo pegamos en un archivo en nuestra máquina local

<img width="618" height="521" alt="8" src="https://github.com/user-attachments/assets/18fe5ad5-9900-48d5-9ddc-310564c48289" />

Luego le damos los permisos 600 (lectura y escritura) para el nuestro usuario al id_rsa

<img width="196" height="62" alt="9" src="https://github.com/user-attachments/assets/1bd906ab-3060-4958-be5b-7ad79879a1a3" />

Luego ingresamos a ssh mediante la clave pública adquirida de la siguiente manera:

```bash
ssh -i id_rsa nico@172.17.0.2
```

<img width="770" height="215" alt="10" src="https://github.com/user-attachments/assets/5e3960c2-3e80-43b0-92fe-f2a5864bb9cd" />




---

## 3. Escalada de Privilegios

Una vez dentro ponemos el comando sudo -l para ver los binarios disponibles

<img width="484" height="102" alt="11" src="https://github.com/user-attachments/assets/7612755f-4629-43ea-a1e8-4803b9288f98" />

Y tenemos el binario “env”, para explotarlo utilizamos el comando:

```bash
sudo /usr/bin/env /bin/bash
```

<img width="399" height="139" alt="12" src="https://github.com/user-attachments/assets/71f88e99-7ee9-43e2-b10b-e84d7a0446f7" />



**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
