# Upload — DockerLabs.es 🐳

<img width="550" height="301" alt="1" src="https://github.com/user-attachments/assets/56eb9501-f637-42a9-a2c0-4854da33684a" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.17.0.2 -oN escaneo.txt
```

<img width="604" height="239" alt="2" src="https://github.com/user-attachments/assets/8af13e24-b12e-49bf-9ace-506a6a58099e" />

Tenemos el puerto 80 abierto.

Ingresamos a la web y nos encontramos con un panel de subida de archivos.

<img width="929" height="490" alt="3" src="https://github.com/user-attachments/assets/0820925f-a591-463d-b0d2-97db93b17555" />



---

## 2. Explotación

Creamos una reverse shell y lo guardamos, en este caso le puse le nombre “virus.php”.

Subimos el reverse shell

<img width="761" height="443" alt="4" src="https://github.com/user-attachments/assets/0601a681-9ba9-43a3-a04a-a1bc2b5d582f" />

Y luego hacemos un fuzzing web para saber en dónde se subió el archivo subido, en este caso se subió en el directorio “/uploads”

<img width="668" height="218" alt="5" src="https://github.com/user-attachments/assets/61f7e7ca-49ea-4571-9924-daff95ceb7fc" />

<img width="587" height="390" alt="6" src="https://github.com/user-attachments/assets/b33a1913-c92b-4116-991d-bc819ae9cc96" />

Luego nos ponemos en escucha con netcat y clickeamos en el archivo “virus.php”

<img width="525" height="97" alt="7" src="https://github.com/user-attachments/assets/f2beb7fb-6d5f-4552-85d7-5f6685d4b596" />

<img width="587" height="390" alt="8" src="https://github.com/user-attachments/assets/ceb3e105-6752-46e5-b567-4be0b7a482b9" />

<img width="849" height="194" alt="15" src="https://github.com/user-attachments/assets/b554d30f-27db-4aa2-973a-de521fd1c925" />


Una vez tenemos acceso, realizamos el tratamiento de la TTY:

```bash
script /dev/null -c bash
control_z

stty raw -echo; fg
reset xterm

export TERM=xterm
export SHELL=bash
```



---

## 3. Escalada de Privilegios

Luego hacemos un sudo -l para listar los binarios sudoers del usuario actual disponibles

<img width="762" height="166" alt="10" src="https://github.com/user-attachments/assets/e7144b9f-f02d-4c1a-8225-191c8cb9438d" />


Para explotar el binario “env” utilizamos el siguiente comando

<img width="382" height="132" alt="11" src="https://github.com/user-attachments/assets/6f97ab85-3781-469e-bd8b-e91922345918" />


**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
