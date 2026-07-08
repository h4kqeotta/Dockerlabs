# BuscaLove — DockerLabs.es 🐳

<img width="548" height="298" alt="1" src="https://github.com/user-attachments/assets/a5e8c3c6-e59a-49a3-b501-38803e9eaf58" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos un escaneo con nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.18.0.2 -oN escaneo.txt
```

<img width="654" height="319" alt="2" src="https://github.com/user-attachments/assets/4af110ea-a23f-41fb-9030-9b51f5a70b4c" />

Tenemos los puertos 22 y 80 abiertos.

Si revisamos el servidor web, no encontramos nada interesante

<img width="931" height="550" alt="3" src="https://github.com/user-attachments/assets/cb63ace4-d8d0-4869-bb8d-88cfe9a6b9ab" />

Por lo que hacemos un fuzzing web en busca de otros directorios disponibles

<img width="735" height="411" alt="4" src="https://github.com/user-attachments/assets/d72aa082-c6ea-4004-8cc0-604c075f06e6" />

Entramos el directorio ”/wordpress” 

<img width="830" height="484" alt="5" src="https://github.com/user-attachments/assets/3d975ab1-d8b1-4fa3-a3f7-904816de8ae8" />

Hacemos un fuzzing web a este directorio y encontramos el “/index.php”

<img width="666" height="375" alt="6" src="https://github.com/user-attachments/assets/fdc1241f-e389-4ece-8fb1-84d352aba806" />

Y después hacemos un fuzzing para ver qué palabra es la que falta para poder hacer un LFI

```bash
wfuzz -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://172.18.0.2/wordpress/index.php?FUZZ=../../../../../../etc/passwd"
```

<img width="1343" height="248" alt="7" src="https://github.com/user-attachments/assets/05266080-1dd2-4df0-92f0-f149d1b164db" />

Encontramos la palabra “love” y la probamos en la url 

<img width="880" height="462" alt="8" src="https://github.com/user-attachments/assets/7bf0c02a-c2fb-4eb1-bb82-c99dc42c9c6c" />


Los usuario encontrados fueron “pedro” y “rosa” por lo que podemos hacer una ataque de fuerza bruta con hydra.

---

## 2. Explotación

Probamos con el usuario “rosa”

```bash
hydra -l rosa -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2/
```

<img width="838" height="306" alt="9" src="https://github.com/user-attachments/assets/718accfa-e24a-4e00-bd61-6ef33085b0f9" />

Una vez dentro, si hacemos un sudo -l podemos ver los binarios “cat” y “ls” disponibles

<img width="528" height="100" alt="10" src="https://github.com/user-attachments/assets/12de9cd7-3fa4-4eab-b28b-0c1e33d0b38c" />

Podemos usar el binario ls para listar los archivos del directorio /root  

<img width="357" height="35" alt="11" src="https://github.com/user-attachments/assets/7505789a-a438-4fd3-8dd8-b979a4a544ef" />

Encontramos el archivo “secret.txt”, podemos ver que contiene adentro con el binario cat

<img width="586" height="36" alt="12" src="https://github.com/user-attachments/assets/a91ae2d6-6e68-4bc1-b6c4-e05c1e33449a" />

4E 5A 58 57 43 59 33 46 4F 4A 32 47 43 34 54 42 4F 4E 58 58 47 32 49 4B

Ingresamos el output en la página de cyberchef para descifrar el contenido

<img width="1081" height="407" alt="13" src="https://github.com/user-attachments/assets/205c0d3b-66a0-46ee-b0d4-2bb2c39ce97d" />

`noacertarasosi`

Si vamos al /home y listamos los archivos, podemos ver que estan los directorios “rosa” y “pedro”

<img width="239" height="36" alt="14" src="https://github.com/user-attachments/assets/34c762c2-83b6-4254-a3c5-fffef3b91bbd" />


---

## 3. Escalada de Privilegios

Probamos la contraseña “noacertarasosi” con el usuario “pedro”.

<img width="571" height="133" alt="15" src="https://github.com/user-attachments/assets/77853e85-6224-484d-b47c-d44385e4620c" />


Y luego de poner el comando sudo -l vemos que tenemos disponible el binario env.

Para explotar este binario utilizamos el siguiente comando:

```bash
sudo env /bin/bash
```

<img width="371" height="119" alt="16" src="https://github.com/user-attachments/assets/60478628-c3cd-4dc5-a03d-5b072b71e9db" />


**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
