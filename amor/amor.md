# amor — DockerLabs.es 🐳

**Plataforma**: DockerLabs | **OS**: Linux | **Dificultad**: Fácil

---

## 1. Reconocimiento

```bash
sudo nmap -p- --min-rate 2000 -A -sS -Pn -n 172.17.0.2 -oN escaneo.txt
```

Enumeramos servicios y directorios disponibles en el objetivo.

---

## 2. Explotación

Identificamos el vector de acceso inicial y logramos obtener una shell en el sistema.

---

## 3. Escalada de Privilegios

Escalamos privilegios para obtener acceso root.

**✅ Máquina rooteada con éxito.**

---

## 4. Técnicas utilizadas

- Reconocimiento con Nmap
- Enumeración de servicios
- Escalada de privilegios
