# Hack The Box - Crocodile Writeup

![Hack The Box](https://img.shields.io/badge/Hack%20The%20Box-Crocodile-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Very%20Easy-brightgreen?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Linux-blue?style=for-the-badge&logo=linux)

Writeup y documentación del proceso de resolución de la máquina **Crocodile** (Starting Point) de Hack The Box.

---

## 📌 Resumen

* **Máquina:** Crocodile
* **Dificultad:** Very Easy
* **Sistema Operativo:** Linux
* **Vector de entrada:** Servidor FTP con acceso anónimo (*Anonymous Login*).
* **Vulnerabilidad principal:** Exposición de archivos con credenciales en FTP e interfaz de administración web desprotegida.

---

## 🛠️ Herramientas Utilizadas

* `ping` — Verificación de conectividad con la máquina objetivo.
* `nmap` — Escaneo e identificación de puertos y servicios abiertos.
* `ftp` — Cliente para la transferencia y descarga de archivos.
* `gobuster` — Fuerza bruta de directorios y archivos web (`dir`).

---

## 🔍 Fase 1: Reconocimiento (Enumeración)

Comenzamos comprobando la conectividad y ejecutando un escaneo completo de puertos y detección de versiones con `nmap`:

```bash
nmap -sC -sV 10.129.x.x
```

### Resultados del escaneo:
* **Puerto 21/tcp (FTP):** Servicio `vsftpd 3.0.3` con el acceso anónimo habilitado (`Anonymous FTP login allowed`).
* **Puerto 80/tcp (HTTP):** Servidor web `Apache httpd 2.4.41`.

---

## 📂 Fase 2: Explotación del Servicio FTP

Al detectar que el servidor FTP permite la entrada anónima:

1. Nos conectamos usando el usuario `Anonymous`:
   ```bash
   ftp 10.129.x.x
   ```
2. Listamos el contenido disponible en el servidor:
   ```ftp
   ls -la
   ```
3. Descargamos los dos archivos identificados en el directorio:
   ```ftp
   get allowed.userlist
   get allowed.userlist.passwd
   exit
   ```
4. Revisamos los datos obtenidos en local:
   ```bash
   cat allowed.userlist
   cat allowed.userlist.passwd
   ```

---

## 🌐 Fase 3: Enumeración Web y Autenticación

### 1. Búsqueda de Directorios con Gobuster
Realizamos una enumeración de rutas en el puerto 80 buscando específicamente archivos con extensión `.php`:

```bash
gobuster dir -u [http://10.129.x.x/](http://10.129.x.x/) -w /usr/share/wordlists/dirb/common.txt -x php
```

**Resultado:** Identificamos la ruta `/login.php`.

### 2. Acceso al Panel de Administración y Captura de Flag
1. Abrimos el navegador y accedemos a `http://10.129.x.x/login.php`.
2. Introducimos el usuario `admin` (obtenido de `allowed.userlist`) y la clave correspondiente hallada en `allowed.userlist.passwd`.
3. Al autenticarnos correctamente, la aplicación nos redirige al panel `/dashboard.php`, donde se encuentra la **flag**.

---

## 🛡️ Medidas de Mitigación / Remediación

1. **Desactivar el acceso anónimo en FTP:**
   Configurar la directiva `anonymous_enable=NO` en `/etc/vsftpd.conf` y reiniciar el servicio.
2. **Principio de mínimo privilegio:**
   No almacenar listados de usuarios ni archivos de claves en carpetas públicas o accesibles sin autenticación.
3. **Hardening web:**
   Asegurar las rutas administrativas e implementar políticas de contraseñas robustas.
