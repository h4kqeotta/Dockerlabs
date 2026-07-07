# Move — DockerLabs.es 🐳


<img width="547" height="298" alt="1" src="https://github.com/user-attachments/assets/0b8d1e8c-736f-425a-b32d-f66785a5ab3a" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.17.0.2 -oN escaneo.txt
```
<img width="563" height="213" alt="2" src="https://github.com/user-attachments/assets/ab439de9-ef15-40b7-bf4f-4e5d2c8507af" />

Encontramos los puertos 22, 80 y 3000 abiertos.

Luego hacemos un fuzzing web a la ip

<img width="569" height="413" alt="3" src="https://github.com/user-attachments/assets/36cf224a-fcfc-426c-ba9a-8386339b1b47" />

Si entramos a la web del puerto 80 nos encontramos con un servidor apache el cuál no tiene nada interesante 

<img width="874" height="561" alt="4" src="https://github.com/user-attachments/assets/e466d6f7-d479-4f33-a7f7-5aa2ea472829" />

Pero si entramos al directorio “/maintenance.html” nos encontramos con lo siguiente:


<img width="950" height="173" alt="5" src="https://github.com/user-attachments/assets/def3f7d9-e90d-46f9-b61f-5e26066a095b" />

Luego si entramos en el puerto 3000 no encontramos con un panel de login de Grafana

<img width="830" height="575" alt="6" src="https://github.com/user-attachments/assets/701f4cf5-1671-4a03-b171-7b8ac94261ba" />

---

## 2. Explotación

Nos indica la versión de Grafana: 8.3.0 por lo que podemos buscarlo en searchsploit de la siguiente manera:

```bash
searchsploit grafana 8.3.0
```
Una vez encontrado el exploit, lo descargamos con el comando:

```bash
searchsploit -m multiple/webapps/50581.py
```

<img width="1344" height="284" alt="7" src="https://github.com/user-attachments/assets/04dfaed7-87ac-4ad0-a304-5df9fcf409f8" />

Luego lo ejecutamos de la siguiente manera:

```bash
python 50581.py -H http://172.17.0.2:3000
```

<img width="632" height="466" alt="8" src="https://github.com/user-attachments/assets/4e45ea86-02a1-4e43-b6d0-f0f7be3e80e8" />


Encontramos el usuario “freddy”. Y también podemos probar con leer el archivo que nos mencionaba el directorio /maintenance 

<img width="259" height="61" alt="9" src="https://github.com/user-attachments/assets/29797930-5dcd-433d-89e4-0aaccf6aa829" />


`t9sH76gpQ82UFeZ3GXZS`

Probamos estas credenciales en el servicio ssh y obtenemos acceso

<img width="768" height="394" alt="10" src="https://github.com/user-attachments/assets/69800c95-6618-437c-b2cf-97114d4588c9" />


---

## 3. Escalada de Privilegios

Ahora podemos utilizar el comando sudo -l para listar los binario disponibles 

<img width="964" height="116" alt="11" src="https://github.com/user-attachments/assets/c62cba65-8117-42f5-a37a-a1ee87ca8cb5" />


Nos muestra el binario python3 y un archivo python que se ejecuta.

Vamos al directorio /opt y listamos el archivo para ver los permisos que tiene, como somos el usuario freddy tenemos permiso para escribir en este archivo, por lo que vamos a modificar el contenido del mismo y vamos a darle permisos SUID al binario bash

<img width="517" height="226" alt="12" src="https://github.com/user-attachments/assets/9ae7b8e3-cfcb-4473-b426-0d6fc42daee5" />


Para modificarlo hacemos un “nano maintenance.py” y ponemos lo siguiente:

<img width="333" height="134" alt="13" src="https://github.com/user-attachments/assets/50f7d33a-f9a2-43a1-bf27-f0eabf361a1d" />

Guardamos el archivo y ejecutamos el siguiente comando:
```bash
sudo /usr/bin/python3 /opt/maintenance.py

bash -p
```
<img width="546" height="69" alt="14" src="https://github.com/user-attachments/assets/9b39f247-74cd-4d72-93dc-a1131215321c" />


**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
