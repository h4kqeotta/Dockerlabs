# DockerLabs — DockerLabs.es 🐳

<img width="547" height="298" alt="1" src="https://github.com/user-attachments/assets/5aff2756-379a-4dca-a8b2-0a0848221f7d" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:
```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.17.0.2 -oN escaneo.txt
```

<img width="828" height="335" alt="2" src="https://github.com/user-attachments/assets/268d2ddf-93bb-44a6-b772-c43ea84d67e0" />

Encontramos el puerto 80 abierto.

Realizamos fuzzing web a la ip para ver qué directorios se encuentran disponibles.

<img width="740" height="442" alt="3" src="https://github.com/user-attachments/assets/bb3dbb58-b1cf-4c5f-890f-682e89a09fad" />

Entramos a la web para ver qué contiene, y no hay nada interesante, tampoco en el código fuente.

<img width="1235" height="548" alt="4" src="https://github.com/user-attachments/assets/8addea51-330f-45e4-b339-348c64f45a1b" />

Entramos a su directorio /machine.php y nos encontramos con un panel para cargar archivos.

<img width="880" height="391" alt="5" src="https://github.com/user-attachments/assets/2151584f-838e-4ba0-a05c-d0d069280462" />


---

## 2. Explotación

Creamos una reverse shell con la ip de nuestra máquina atacante, al puerto que queremos y seleccionamos el código de PHP PentestMonkey, copiamos el output del código, lo pegamos en un archivo y lo guardamos con el nombre “shell.php”

<img width="587" height="448" alt="6" src="https://github.com/user-attachments/assets/1ad64d8d-feea-443c-9ab1-3d52c3f68696" />

Luego subimos el archivo creado.

<img width="717" height="322" alt="7" src="https://github.com/user-attachments/assets/a01329a5-8c89-4020-ab02-1f927d477495" />

Y nos sale un mensaje de que solo acepta archivos .zip, en este caso podemos probar con cambiarle la extensión al archivo interceptando este paquete con BurpSuite 

<img width="557" height="152" alt="8" src="https://github.com/user-attachments/assets/28798143-3e5c-442a-bc22-c51100f4d233" />

Primero activamos el FoxyProxy

<img width="300" height="368" alt="9" src="https://github.com/user-attachments/assets/c56edf01-81c1-42fc-94de-4d571c42519e" />

Luego abrimos BurpSuite y en el apartado de “Proxy” activamos la interceptación:

<img width="424" height="222" alt="10" src="https://github.com/user-attachments/assets/1f5161e1-f517-46d7-9db7-ffef98de4e5a" />

Una vez activado el FoxyProxy y BurpSuite, vamos a reenviar el “shell.php” en la página para que sea interceptado.

<img width="849" height="600" alt="11" src="https://github.com/user-attachments/assets/c7ea44ca-fefe-4346-aff2-e898a0b9040a" />

Luego enviamos la trama al “intruder” con control_i. Seleccionamos todo el texto y le damos a “Clear”.

Paso siguiente seleccionamos la palabra “php” en el nombre de nuestro archivo y apretamos en “Add” para que se modifique solo esa palabra

<img width="919" height="575" alt="12" src="https://github.com/user-attachments/assets/ee0a6e54-3493-404f-becb-1000a43faa73" />

Luego seleccionamos el tipo de payload, en este caso un “Simple list” y abajo le ponemos una serie de extensiones para que se vayan probando una por una.

<img width="395" height="442" alt="13" src="https://github.com/user-attachments/assets/60310959-e476-474b-b06b-042099bc810b" />

Una vez realizado estos pasos, le damos a “Start attack” para que se inicie el ataque.

Y como veremos a continuación la extensión “phar” responde a la página enviando un mensaje de “El archivo shell.phar ha sido subido correctamente”

<img width="1037" height="479" alt="14" src="https://github.com/user-attachments/assets/5b449273-d95f-455c-8d09-3baec500cf6f" />

Luego volvemos al apartado “Proxy” y le modificamos la extensión a nuestro archivo a .phar

Y para finalizar hacemos click en “Forward”

<img width="1018" height="565" alt="15" src="https://github.com/user-attachments/assets/336d91b8-951d-451d-b0f7-235e1760808d" />

<img width="559" height="124" alt="16" src="https://github.com/user-attachments/assets/b9488976-ee22-472e-bad2-c50e9288e595" />

Una vez subido el archivo, vamos al directorio “/uploads” que es dónde se suben los archivos subidos en el servidor.

<img width="550" height="231" alt="17" src="https://github.com/user-attachments/assets/8183a63c-cd32-4105-bf12-2e96fc60e718" />

Pero antes de hacer nada, nos ponemos en escucha con netcat en el puerto 443

<img width="537" height="84" alt="18" src="https://github.com/user-attachments/assets/d9869244-9f0a-4f62-8e11-cb664bc57c53" />

Luego hacemos click en el archivo “shell.php”

<img width="491" height="266" alt="19" src="https://github.com/user-attachments/assets/f4d13591-5139-43d7-8555-2ba46d4f6398" />

Y así logramos obtener acceso a la máquina

<img width="650" height="214" alt="20" src="https://github.com/user-attachments/assets/ab7987ea-da4e-4bdf-a8e2-65d37b32d66e" />

Hacemos el tratamiento de la TTY:

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

Si usamos el comando sudo -l para ver los binario disponibles podemos ver los siguientes:

<img width="511" height="57" alt="21" src="https://github.com/user-attachments/assets/350c22de-aa4e-4992-bbb3-e2a969346551" />

Como no hay nada que leer, vamos a todos los directorios en busca de información extra, hasta que llegamos al /opt y encontramos un archivo llamado “nota.txt”

<img width="1043" height="112" alt="23" src="https://github.com/user-attachments/assets/b955a495-012d-4af6-8eb2-dfac81d0bdac" />

La cual nos muestra un directorio junto a un archivo dentro: /root/clave.txt

Leemos el archivo con el binario grep de la siguiente manera

<img width="413" height="52" alt="24" src="https://github.com/user-attachments/assets/ea59e1c6-58e1-49e8-891e-52e8ab04e561" />

`dockerlabsmolamogollon123`

Probamos esta contraseña con el usuario root

<img width="357" height="128" alt="25" src="https://github.com/user-attachments/assets/6f64617e-925d-486b-8bb2-54470c365cea" />


**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Fuzzing web
- File upload bypass (extensión `.phar` via BurpSuite Intruder)
- Reverse shell PHP
- TTY treatment
- Lectura de archivos con `grep`
