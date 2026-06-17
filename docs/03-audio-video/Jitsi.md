# Jitsi Meet - Videoconferencia

**Máquina:** EC2-4 - Ubuntu 22.04 LTS  
**IP pública:** 3.219.249.6  
**Puertos:** 80, 443 TCP - 10000 UDP

---
## Índice

1. [Descripción del servicio](#1-descripción-del-servicio)
2. [Protocolo WebRTC](#2-protocolo-webrtc)
3. [Requerimientos](#3-requerimientos)
4. [Instalación con Docker en EC2-4](#4-instalación-con-docker-en-ec2-4)
5. [Configuración del archivo .env](#5-configuración-del-archivo-env)
6. [Prueba de videollamada](#6-prueba-de-videollamada)
7. [Validación del servicio](#7-validación-del-servicio)
8. [Incidencias y soluciones](#8-incidencias-y-soluciones)
9. [Security Group — SG-JITSI](#9-security-group---sg-jitsi)
    
## 1. Descripción del servicio

Jitsi Meet es una plataforma de videoconferencia de código abierto basada en WebRTC que permite realizar videollamadas directamente desde el navegador sin instalar software adicional. Se ha desplegado en un servidor EC2 separado (EC2-4) usando Docker Compose, que es el método oficial recomendado.

Se utiliza en InnovateTech para la comunicación interna entre departamentos, reuniones de equipo y formación corporativa.

---

## 2. Protocolo WebRTC

WebRTC (Web Real-Time Communication) es un estándar abierto que permite comunicación de audio, vídeo y datos en tiempo real directamente entre navegadores.

| Componente | Función |
|---|---|
| ICE / STUN / TURN | Encuentra la mejor ruta de red entre los clientes y gestiona conexiones detrás de NAT |
| DTLS | Cifra todas las conexiones de extremo a extremo |
| SRTP | Protocolo seguro para transmitir audio y vídeo en tiempo real |
| SDP | Negocia los parámetros y capacidades de cada cliente |

---

## 3. Requerimientos

- Descripción del servicio Jitsi Meet y del protocolo WebRTC.
- Instalación y configuración de Jitsi Meet en servidor separado (EC2-4).
- Acceso al servicio desde el navegador.
- Realización de al menos una videollamada funcional entre dos usuarios.

---

## 4. Instalación con Docker en EC2-4

*Servidor: EC2-4 — Ubuntu 22.04 LTS — Servidor separado del EC2-3. Puertos: 80, 443 TCP y 10000 UDP abiertos en Security Group.*

**Paso 1 - Actualizar e instalar Docker:**

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://get.docker.com | sudo bash
sudo apt install docker-compose-plugin -y
sudo systemctl enable docker && sudo systemctl start docker
```

**Paso 2 - Descargar y configurar Jitsi Meet:**

```bash
cd /opt && sudo git clone https://github.com/jitsi/docker-jitsi-meet
cd docker-jitsi-meet && sudo cp env.example .env
sudo bash gen-passwords.sh
mkdir -p ~/.jitsi-meet-cfg/{web,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}
```

**Paso 3 - Arrancar los contenedores:**

```bash
sudo docker-compose up -d
```

**Paso 4 - Verificar que todos los contenedores están activos:**

```bash
sudo docker-compose ps
```

> <img width="681" height="199" alt="unnamed" src="https://github.com/user-attachments/assets/b3de8ca3-e348-414c-89c2-09cb1de5d764" />
> <img width="1084" height="134" alt="unnamed" src="https://github.com/user-attachments/assets/dbd53ee0-4e99-4df9-849a-bc2137b6ef96" />


---

## 5. Configuración del archivo .env

El archivo `.env` contiene la configuración principal. Se edita con:

```bash
sudo nano /opt/docker-jitsi-meet/.env
```

| Parámetro | Valor configurado | Descripción |
|---|---|---|
| `HTTP_PORT` | 8000 | Puerto HTTP de acceso a Jitsi Meet |
| `HTTPS_PORT` | 8443 | Puerto HTTPS de acceso a Jitsi Meet |
| `TZ` | UTC | Zona horaria del servidor |
| `PUBLIC_URL` | https://3.219.249.6:8443 | URL pública completa con puerto. Necesario para WebRTC. |
| `ENABLE_AUTH` | 0 | Sin autenticación (cualquiera puede crear sala) |
| `ENABLE_GUESTS` | 1 | Permite invitados sin cuenta |
| `JVB_PORT` | 10000 | Puerto UDP para tráfico de vídeo. OBLIGATORIO abrirlo en AWS |
| `JITSI_IMAGE_VERSION` | stable-9646 | Versión estable de Jitsi. Necesario para evitar error SCRAM-SHA-1. |

> <img width="911" height="447" alt="unnamed" src="https://github.com/user-attachments/assets/23da9ccd-df9c-4d43-9e3b-775de4c949f6" />

---

## 6. Prueba de videollamada

Acceder desde el navegador a `https://3.219.249.6:8443` (aceptar el certificado autofirmado: *Avanzado > Continuar*) y:

1. **Usuario 1:** crear sala con nombre `prueba-innovatetech` y entrar.
2. **Usuario 2:** abrir la misma URL en modo incógnito (`Ctrl+Shift+N`) o en otro dispositivo.
3. Verificar que ambos usuarios se ven y escuchan correctamente.

> ⚠️ WebRTC requiere HTTPS para acceder a cámara y micrófono. Si el navegador bloquea el micrófono, aceptar el certificado autofirmado correctamente.

> <img width="1084" height="514" alt="unnamed" src="https://github.com/user-attachments/assets/e26976e5-706e-4861-80f7-4cc9e37412c3" />
> <img width="1084" height="597" alt="unnamed" src="https://github.com/user-attachments/assets/29202144-4367-412f-ad57-da02076f0395" />

---

## 7. Validación del servicio

- Videollamada funcional en tiempo real entre dos usuarios.
- Audio y vídeo bidireccional correcto.
- Acceso desde el navegador sin instalar software adicional.

---

## 8. Incidencias y soluciones

| Incidencia | Solución aplicada |
|---|---|
| `E: Unable to locate package docker-compose-plugin` | Instalar Docker moderno: `curl -fsSL https://get.docker.com \| sudo bash` |
| `unknown shorthand flag: d` / `docker: unknown command` | Versión antigua de Docker. Instalar versión moderna con el script oficial de Docker. |
| `KeyError: ContainerConfig` al ejecutar `docker-compose up` | Incompatibilidad entre docker-compose 1.29.2 y las imágenes nuevas de Jitsi. Instalar Docker moderno. |
| `SASLError SCRAM-SHA-1: not-authorized` en logs del jvb | Usar versión estable de Jitsi. Añadir en `.env`: `JITSI_IMAGE_VERSION=stable-9646`. Borrar `~/.jitsi-meet-cfg` y regenerar con `gen-passwords.sh` |
| *Ha sido desconectado* al entrar a la reunión | `PUBLIC_URL` no incluía el puerto. Cambiado a: `PUBLIC_URL=https://3.219.249.6:8443` |
| `ERR_CONNECTION_REFUSED` al acceder a Jitsi | Puertos 8000 y 8443 no estaban abiertos en el Security Group del EC2-4 en AWS. |

---

## 9. Security Group - SG-JITSI

| Dirección | Protocolo | Puerto | Origen |
|---|---|---|---|
| Entrada | TCP | 80 | 0.0.0.0/0 |
| Entrada | TCP | 443 | 0.0.0.0/0 |
| Entrada | UDP | 10000 | 0.0.0.0/0 |
| Entrada | TCP | 4443 | 0.0.0.0/0 |
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Salida | TCP/UDP | 514 | 10.0.0.0/16 |
