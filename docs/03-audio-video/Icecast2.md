# Icecast2 - Streaming de Audio

**Máquina:** EC2-3 - Ubuntu 22.04 LTS  
**IP pública:** 32.198.236.17  
**Puerto:** 8000 TCP

---
## Índice

1. [Descripción del servicio](#1-descripción-del-servicio)
2. [Requerimientos](#2-requerimientos)
3. [Instalación](#3-instalación)
4. [Configuración del servicio](#4-configuración-del-servicio)
5. [Configuración del emisor de audio — ffmpeg](#5-configuración-del-emisor-de-audio--ffmpeg)
   - [Por qué ffmpeg y no relay nativo de Icecast2](#por-qué-ffmpeg-y-no-relay-nativo-de-icecast2)
   - [Paso 1 — Verificar que el stream es accesible](#paso-1--verificar-que-el-stream-es-accesible)
   - [Paso 2 — Crear el servicio systemd](#paso-2--crear-el-servicio-systemd)
   - [Paso 3 — Activar e iniciar el servicio](#paso-3--activar-e-iniciar-el-servicio)
6. [Acceso vía navegador web](#6-acceso-via-navegador-web)
7. [Formatos de audio utilizados](#7-formatos-de-audio-utilizados)
8. [Incidencias y soluciones](#8-incidencias-y-soluciones)
9. [Security Group — SG-MULTIMEDIA](#9-security-group---sg-multimedia)
## 1. Descripción del servicio

Icecast2 es un servidor de streaming de audio de código abierto que permite transmitir audio en tiempo real a múltiples clientes simultáneamente. Actúa como punto central de distribución: recibe el audio de una fuente emisora y lo redistribuye a todos los oyentes conectados.

Soporta los formatos MP3, OGG Vorbis y AAC, y dispone de un panel web integrado accesible desde el navegador para monitorizar el estado del servicio, los canales activos y el número de oyentes en tiempo real.

En este proyecto se utiliza para retransmitir Radio France Inter en directo dentro de la infraestructura de InnovateTech. ffmpeg captura el stream de la radio y lo envía a Icecast2, que lo redistribuye a todos los oyentes conectados.

---

## 2. Requerimientos

- Descripción de la funcionalidad del servicio de audio.
- Instalación y configuración de un servidor de streaming de audio (Icecast2).
- Uso del formato de audio digital MP3.
- Configuración del emisor de audio (ffmpeg) para retransmitir una radio en directo.
- Acceso al servicio via navegador web.
- Documentación de la instalación, configuración y pruebas del servicio.

---

## 3. Instalación

*Servidor: EC2-3 - Ubuntu 22.04 LTS - IP pública: 32.198.236.17 - Puerto 8000 TCP abierto en el Security Group de AWS*

**Paso 1 - Corregir dependencias rotas (necesario en este servidor):**

```bash
sudo dpkg --configure -a
sudo apt --fix-broken install -y
```

**Paso 2 - Actualizar e instalar Icecast2:**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install icecast2 -y
```

Durante la instalación el asistente pregunta:
- Hostname → `32.198.236.17`
- Source password → `pirineus`
- Relay password → `pirineus`
- Admin password → `pirineus`

**Paso 3 - Activar e iniciar el servicio:**

```bash
sudo systemctl enable icecast2
sudo systemctl start icecast2
```

**Paso 4 - Verificar que funciona:**

```bash
sudo systemctl status icecast2
```
<img width="730" height="345" alt="unnamed" src="https://github.com/user-attachments/assets/1414b292-e400-498c-aa8e-32a72929282f" />

---

## 4. Configuración del servicio

El archivo de configuración principal es `/etc/icecast2/icecast.xml`:

```bash
sudo nano /etc/icecast2/icecast.xml
```

Parámetros modificados:

| Parámetro | Valor configurado | Descripción |
|---|---|---|
| `<hostname>` | 32.198.236.17 | IP pública del servidor EC2-3 en AWS |
| `<port>` | 8000 | Puerto de escucha. Abierto en el Security Group de AWS |
| `<source-password>` | pirineus | Contraseña para clientes source |
| `<relay-password>` | pirineus | Contraseña para servidores relay |
| `<admin-user>` | admin | Usuario del panel web de administración |
| `<admin-password>` | pirineus | Contraseña del panel web de administración |

Después de modificar el archivo, reiniciar el servicio:

```bash
sudo systemctl restart icecast2
```

<img width="723" height="200" alt="unnamed" src="https://github.com/user-attachments/assets/cfd360ea-93a3-4aec-81fc-5fdac719ede9" />
<img width="735" height="306" alt="unnamed" src="https://github.com/user-attachments/assets/4c676f75-c7c2-434c-a325-33ae28d91605" />

---

## 5. Configuración del emisor de audio — ffmpeg

Se utiliza ffmpeg para capturar el stream de Radio France Inter y enviarlo a Icecast2. ffmpeg se configura como servicio systemd para que arranque automáticamente con el servidor.

### Por qué ffmpeg y no relay nativo de Icecast2

El relay nativo de Icecast2 no soporta conexiones HTTPS (SSL/TLS). Radio France Inter emite en HTTPS, por lo que el relay nativo da error `Bad Request`. ffmpeg sí soporta HTTPS y resuelve este problema.

### Paso 1 — Verificar que el stream es accesible

```bash
curl -I https://icecast.radiofrance.fr/franceinter-midfi.mp3
```
Debe devolver `HTTP/2 200`.
<img width="1159" height="419" alt="image" src="https://github.com/user-attachments/assets/307aea1b-311d-4ac2-93fd-8f593fd3b33e" />


### Paso 2 — Crear el servicio systemd

```bash
sudo nano /etc/systemd/system/radio-innovatetech.service
```

Contenido:

```ini
[Unit]
Description=InnovateTech Radio Stream
After=network.target icecast2.service

[Service]
ExecStart=ffmpeg -re -i https://icecast.radiofrance.fr/franceinter-midfi.mp3 -c:a copy -f mp3 icecast://source:pirineus@localhost:8000/stream
Restart=always
RestartSec=10
User=adminitb
<img width="1475" height="368" alt="image" src="https://github.com/user-attachments/assets/7fc1931b-f099-4fa6-94e1-c1bf506cdead" />

[Install]
WantedBy=multi-user.target
```

| Parámetro | Valor | Descripción |
|---|---|---|
| `After` | icecast2.service | ffmpeg espera a que Icecast2 esté activo antes de arrancar |
| `-re` | — | Lee el stream a velocidad real (1x) |
| `-i` | URL de Radio France Inter | Fuente del stream de audio en HTTPS |
| `-c:a copy` | — | Copia el audio sin recodificar (menos CPU) |
| `-f mp3` | — | Formato de salida MP3 |
| `Restart=always` | — | Si ffmpeg se cae, se reinicia automáticamente |
| `RestartSec=10` | — | Espera 10 segundos antes de reiniciar |

### Paso 3 — Activar e iniciar el servicio

```bash
sudo systemctl daemon-reload
sudo systemctl enable radio-innovatetech
sudo systemctl start radio-innovatetech
sudo systemctl status radio-innovatetech
```

El servicio arranca automáticamente cada vez que se reinicia el servidor.

---

## 6. Acceso via navegador web

Una vez ffmpeg está emitiendo, el servicio es accesible desde cualquier navegador:

- **Panel de administración:** http://32.198.236.17:8000
- **URL directa del stream:** http://32.198.236.17:8000/stream
- **Estado del servicio:** http://32.198.236.17:8000/status.xsl

<img width="1852" height="973" alt="unnamed" src="https://github.com/user-attachments/assets/400b420b-7aca-488b-9778-f136ed7c467b" />

---

## 7. Formatos de audio utilizados

| Formato | Descripción técnica | Uso en el proyecto |
|---|---|---|
| **MP3** | MPEG-1 Audio Layer III. Compresión con pérdida. Bitrate: 128 kbps. Compatible con todos los navegadores sin plugins. | Formato del stream. Radio France Inter emite en MP3 y ffmpeg lo redistribuye a Icecast2 en el mismo formato. |
| **OGG Vorbis** | Formato libre y abierto. Mejor calidad que MP3 al mismo bitrate. | Soportado por Icecast2. Alternativa libre al MP3. |
| **AAC** | Advanced Audio Coding. Sucesor del MP3 con mejor calidad a menor bitrate. | Soportado por Icecast2. Usado en streaming móvil. |

---

## 8. Incidencias y soluciones

| Incidencia | Solución aplicada |
|---|---|
| `E: Unable to locate package icecast2` | Ejecutar `sudo apt update` antes de instalar |
| `E: dpkg was interrupted` | Ejecutar `sudo dpkg --configure -a` y `sudo apt --fix-broken install -y` |
| `ERR_CONNECTION_TIMED_OUT` al acceder al puerto 8000 | Puerto 8000 no estaba abierto en el Security Group de AWS. Se abrió en Inbound Rules. |
| El relay nativo de Icecast2 da `Bad Request` | Icecast2 no soporta HTTPS. Se sustituyó el relay por ffmpeg que sí soporta HTTPS. |
| El relay de Los 40 no conecta | El servidor de Los 40 no permite relay externo. Se cambió a Radio France Inter. |
| La radio no arranca después de reiniciar el servidor | Se creó el servicio systemd `radio-innovatetech.service` para que ffmpeg arranque automáticamente. |

---

## 9. Security Group - SG-MULTIMEDIA

| Dirección | Protocolo | Puerto | Origen |
|---|---|---|---|
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Salida | TCP/UDP | 514 | 10.0.0.0/16 |
