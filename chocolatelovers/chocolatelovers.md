# ChocolateLovers — DockerLabs.es 🐳

<img width="547" height="296" alt="1" src="https://github.com/user-attachments/assets/487d7006-db2e-437b-98bb-c1ed654eaeac" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

<img width="563" height="223" alt="2" src="https://github.com/user-attachments/assets/0f556b50-0945-44bd-86dd-42105a289087" />

Tenemos abierto el puerto 80. 

Entramos a la web y en el index no aparece nada interesante, pero cuando entramos al codigo fuente nos indica un directorio “/nibbleblog”

<img width="906" height="367" alt="3" src="https://github.com/user-attachments/assets/f983201b-d28c-418a-bc2e-25042d87d55c" />

Entramos al directorio y tenemos lo siguiente:

<img width="784" height="495" alt="4" src="https://github.com/user-attachments/assets/9788b7fe-77dd-415d-9cb5-fd95301f4cac" />

Una vez dentro nos encontramos con un panel de login, probamos las credenciales por defecto admin:admin 

<img width="795" height="564" alt="5" src="https://github.com/user-attachments/assets/c1a1453a-f206-4981-b7d8-13d2cd4eddc2" />

Y logramos tener acceso.

---

## 2. Explotación

Si vamos al apartado de “Settings”, en la parte inferior de la página podemos ver la versión de este blog

<img width="840" height="476" alt="6" src="https://github.com/user-attachments/assets/cfaefef5-7c53-4914-ad1c-ff9d4c8b8588" />

<img width="615" height="228" alt="7" src="https://github.com/user-attachments/assets/270e1980-8a86-427c-a635-406d842abde5" />

Una vez que sabemos la versión, la buscamos en internet para encontrar alguna vulnerabilidad relacionada.

Y encontramos un CVE la cuál indica hay una vulnerabilidad de carga de archivos sin restricciones en el plugin My Image que permite a administradores remotos ejecutar código arbitrario cargando un archivo y accediendo a él mediante una petición directa al archivo en el directorio content/private/plugins/my_image/image.php.

<img width="924" height="488" alt="8" src="https://github.com/user-attachments/assets/bd89d973-1835-4987-8dad-088d039c461e" />

Por lo que vamos a seguir los pasos:

<img width="981" height="477" alt="9" src="https://github.com/user-attachments/assets/3fbc92c8-3153-47ed-8cad-4297bddd90bc" />

Instalamos el plugin “My image”

<img width="946" height="577" alt="10" src="https://github.com/user-attachments/assets/6bc897c8-a8bb-47cc-859d-caf9a795bf0f" />

Y llegado a este punto cargamos una reverse shell y la subimos

<img width="1042" height="529" alt="11" src="https://github.com/user-attachments/assets/a095ed39-1793-48c3-8661-bc76b2c0cda3" />

<img width="1103" height="587" alt="12" src="https://github.com/user-attachments/assets/53df3c0e-32ad-4d6e-8d44-e4a1f4157635" />

Luego vamos a la ruta anteriormente mencionada por el CVE

Y antes de hacer click en el archivo “image.php” nos ponemos en escucha con netcat

<img width="885" height="410" alt="13" src="https://github.com/user-attachments/assets/6b7601f5-44fb-4f3a-8341-fc88140ea416" />

<img width="974" height="189" alt="14" src="https://github.com/user-attachments/assets/0d6469cf-4585-4f0c-b07b-f7628bcdfb4e" />



---

## 3. Escalada de Privilegios

Una vez dentro nos fijamos los binarios disponibles y nos encontramos con “php” con el usuario “chocolate”

<img width="761" height="178" alt="15" src="https://github.com/user-attachments/assets/2b197b66-9a86-46be-b28d-4aba7c04d4b3" />

Para explotar este binario ingresamos los siguientes comandos para movernos al usuario chocolate

<img width="633" height="97" alt="16" src="https://github.com/user-attachments/assets/8c4dbcc2-26fa-4ea5-894d-64c37501a3fa" />

Luego buscamos formas de escalar privilegios con el usuario chocolate y nos encontramos que el usuario root está ejecutando un script en el directorio /opt   

<img width="1249" height="61" alt="17" src="https://github.com/user-attachments/assets/800c644f-1199-40cb-b3b7-f86716cc608d" />

Para escalar privilegios mediante este script que se ejecuta cada 5 segundos primero vamos a crear una reverse shell en nuestra máquina local con el nombre “script.php” y lo subimos a la máquina victima en el directorio /opt  

<img width="596" height="116" alt="18" src="https://github.com/user-attachments/assets/96b042c8-e856-4674-82f0-5fedd39efdd3" />

<img width="625" height="170" alt="19" src="https://github.com/user-attachments/assets/1b59dbd2-9fff-44ac-b881-9861cbd9c7c0" />

Una vez adquirido el archivo, nos podemos en escucha en el puerto mencionado en el script, en este caso el puerto 445, esperamos 5 segundos y obtenemos la shell siendo root

<img width="956" height="251" alt="20" src="https://github.com/user-attachments/assets/15f3b3bb-3f03-408c-ac07-a6c048907be2" />




**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
