# Multimedia - Jellyfin (Streaming de Vídeo)

**Máquina:** EC2-3 - Ubuntu 22.04 LTS  
**IP pública:** 32.198.236.17  
**Puerto:** 8096 TCP

---
## Índice

1. [Descripción del servicio](#1-descripción-del-servicio)
2. [Requerimientos](#2-requerimientos)
3. [Instalación](#3-instalación)
4. [Configuración del servicio](#4-configuración-del-servicio)
   - [Paso 1 — Crear directorio y descargar vídeo de prueba](#paso-1--crear-directorio-y-descargar-vídeo-de-prueba)
   - [Paso 2 — Configuración inicial vía wizard web](#paso-2--configuración-inicial-via-wizard-web)
   - [Paso 3 — Usuarios creados en Jellyfin](#paso-3--usuarios-creados-en-jellyfin)
5. [Configuración de streaming en directo — archivos .strm](#5-configuración-de-streaming-en-directo--archivos-strm)
   - [Paso 1 — Crear directorio para directos](#paso-1--crear-directorio-para-directos)
   - [Paso 2 — Crear archivo .strm con la URL del directo](#paso-2--crear-archivo-strm-con-la-url-del-directo)
   - [Paso 3 — Desactivar transcoding en Jellyfin](#paso-3--desactivar-transcoding-en-jellyfin-crítico)
   - [Paso 4 — Añadir biblioteca en Jellyfin](#paso-4--añadir-biblioteca-en-jellyfin)
6. [Formatos y codecs utilizados](#6-formatos-y-codecs-utilizados)
7. [Reproducción desde el navegador](#7-reproducción-desde-el-navegador)
8. [Integración con MariaDB](#8-integración-con-mariadb)
9. [Validación del servicio](#9-validación-del-servicio)
10. [Incidencias y soluciones](#10-incidencias-y-soluciones)
11. [Security Group — SG-MULTIMEDIA](#11-security-group--sg-multimedia)
12. [Comprobaciones de Ancho de Banda](#12-comprobaciones-de-ancho-de-banda)
    - [12.1 Objetivo](#121-objetivo)
    - [12.2 Requisitos mínimos obligatorios](#122-requisitos-mínimos-obligatorios)
    - [12.3 Herramientas utilizadas](#123-herramientas-utilizadas)
    - [12.4 Prueba 1 — Sin servicios activos](#124-prueba-1---sin-servicios-activos-línea-base)
    - [12.5 Prueba 2 — Con todos los servicios activos](#125-prueba-2---con-todos-los-servicios-activos)
    - [12.6 Análisis de resultados](#126-análisis-de-resultados)
    - [12.7 Clasificación del sistema](#127-clasificación-del-sistema)
    - [12.8 Propuestas de optimización](#128-propuestas-de-optimización)
      
## 1. Descripción del servicio

Jellyfin es un servidor de streaming de vídeo de código abierto que permite organizar, gestionar y distribuir contenido audiovisual a través de una interfaz web intuitiva. Soporta transcodificación en tiempo real y es accesible desde cualquier navegador moderno sin plugins adicionales.

Se ha elegido Jellyfin porque el enunciado lo menciona explícitamente como opción válida, tiene interfaz web visual que facilita la demostración y soporta de forma nativa H.264 y MP4. Además permite reproducir streams HLS en directo mediante archivos `.strm`.

---

## 2. Requerimientos

- Descripción de la funcionalidad del servicio de vídeo.
- Instalación y configuración de Jellyfin como servidor de vídeo.
- Subida de al menos un vídeo accesible desde el servicio.
- Uso del formato MP4 y el codec H.264.
- Configuración de streaming en directo mediante archivos `.strm`.
- Verificación de la reproducción desde navegador.
- Documentación de la instalación, configuración y pruebas.

---

## 3. Instalación

*Servidor: EC2-3 - Ubuntu 22.04 LTS - IP pública: 32.198.236.17 - Puerto 8096 TCP abierto en el Security Group de AWS*

**Paso 1 - Instalar usando el script oficial de Jellyfin:**

```bash
curl -fsSL https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

> **Nota:** Si el script falla por espacio insuficiente en `/tmp`, ampliar primero:
> ```bash
> sudo mount -o remount,size=4G /tmp
> ```

**Paso 2 — Activar e iniciar:**

```bash
sudo systemctl enable jellyfin
sudo systemctl start jellyfin
```

> <img width="734" height="476" alt="unnamed" src="https://github.com/user-attachments/assets/514273f0-aa61-4e6a-b34c-2b00fc0331b4" />
> <img width="734" height="476" alt="unnamed" src="https://github.com/user-attachments/assets/d8f4b2b0-6b9c-44ac-a706-979eb81ed230" />

---

## 4. Configuración del servicio

### Paso 1 — Crear directorio y descargar vídeo de prueba

```bash
sudo mkdir -p /media/videos
sudo chown jellyfin:jellyfin /media/videos
sudo wget -O /media/videos/video_prueba.mp4 "https://www.w3schools.com/html/mov_bbb.mp4"
sudo chown jellyfin:jellyfin /media/videos/video_prueba.mp4
```

Verificación de que el archivo se descargó correctamente:

```bash
ls -la /media/videos/
```

### Paso 2 — Configuración inicial via wizard web

Acceder a `http://32.198.236.17:8096` y seguir el wizard:

| Paso | Acción |
|---|---|
| 1. Bienvenida | Hacer clic en Siguiente |
| 2. Idioma | Seleccionar el idioma y hacer clic en Siguiente |
| 3. Usuario administrador | Crear usuario: `jellyfin` con contraseña: `pirineus` |
| 4. Biblioteca de medios | Hacer clic en Añadir biblioteca de medios |
| 5. Tipo de biblioteca | Seleccionar *Vídeos y fotos personales* (acepta cualquier MP4) |
| 6. Directorio | Añadir la ruta: `/media/videos` |
| 7. Finalizar | Hacer clic en Siguiente y Terminar |

> <img width="904" height="542" alt="unnamed" src="https://github.com/user-attachments/assets/fb304ec5-356b-41c8-8b15-c57c0eab0ec6" />
> <img width="1082" height="588" alt="unnamed" src="https://github.com/user-attachments/assets/71cd22c3-1dc4-4336-8f5b-b24eee1f64ee" />

### Paso 3 — Usuarios creados en Jellyfin

Se han creado los siguientes usuarios para que puedan acceder a Jellyfin:

| Usuario | Contraseña | Rol |
|---|---|---|
| jellyfin | pirineus | Administrador |
| joan.garcia | pirineus | Usuario estándar |
| maria.lopez | pirineus | Usuario estándar |
| pere.martinez | pirineus | Usuario estándar |
| anna.puig | pirineus | Usuario estándar |

Los usuarios se crearon via API con el siguiente comando:

```bash
curl -s -X POST "http://localhost:8096/Users/New" \
  -H "X-Emby-Token: TOKEN_ADMIN" \
  -H "Content-Type: application/json" \
  -d '{"Name":"joan.garcia","Password":"pirineus"}'
```

---

## 5. Configuración de streaming en directo — archivos .strm

Jellyfin permite reproducir streams HLS en directo sin transcoding mediante archivos `.strm`. Cada archivo `.strm` contiene únicamente la URL del stream HLS.

### Paso 1 — Crear directorio para directos

```bash
sudo mkdir -p /media/videos/directos
sudo chown jellyfin:jellyfin /media/videos/directos
```

### Paso 2 — Crear archivo .strm con la URL del directo

```bash
echo "https://rtvelivestream.rtve.es/rtvesec/24h/24h_main_dvr_720.m3u8" | sudo tee /media/videos/directos/rtve24h.strm
```

### Paso 3 — Desactivar transcoding en Jellyfin (CRÍTICO)

Sin este paso el stream HLS da error de reproducción.

1. Panel admin → **Usuaris** → seleccionar usuario → **Edita**
2. Pestaña **Reproducció**
3. Desactivar:
   - Reproducción de audio que requiere transcoding
   - Reproducción de vídeo que requiere transcoding
   - Reproducción de vídeo que requiere conversión sin recodificación
   - Forzar transcoding de fuentes externas
4. Dejar activado únicamente: **Reproducción multimedia**
5. Guardar

### Paso 4 — Añadir biblioteca en Jellyfin

1. Panel admin → **Mediateques** → **+**
2. Tipo → **Vídeos**
3. Ruta → `/media/videos/directos`
4. Guardar y escanear

El directo aparece en Jellyfin como un vídeo normal y se reproduce directamente desde el navegador.

---

## 6. Formatos y codecs utilizados

| Elemento | Valor | Descripción |
|---|---|---|
| Contenedor | MP4 (`.mp4`) | Formato estándar universal para distribución de vídeo en web |
| Codec vídeo | H.264 (AVC) | Codec más compatible. Soportado por todos los navegadores modernos sin plugins |
| Codec audio | AAC | Audio comprimido de alta calidad incluido en el contenedor MP4 |
| Protocolo vídeo local | HTTP Progressive | Jellyfin sirve el vídeo via HTTP. El navegador lo reproduce progresivamente |
| Protocolo streaming directo | HLS (HTTP Live Streaming) | Protocolo de streaming adaptativo usado por RTVE y otras plataformas |
> <img width="604" height="379" alt="image" src="https://github.com/user-attachments/assets/f5a2524e-4920-4dfe-a01b-541cad541d2d" />
> <img width="927" height="1013" alt="image" src="https://github.com/user-attachments/assets/c69d1a54-0e25-4906-96c8-98e0c4b6fd7c" />

---

## 7. Reproducción desde el navegador

Una vez el vídeo está en la biblioteca, se accede desde cualquier navegador en `http://32.198.236.17:8096`. Jellyfin proporciona un player web integrado con controles de reproducción, volumen y calidad.

> <img width="1018" height="573" alt="unnamed" src="https://github.com/user-attachments/assets/5313df71-e4a5-4e43-8799-13e1255932f1" />
> <img width="993" height="621" alt="unnamed" src="https://github.com/user-attachments/assets/e3826473-33b6-4501-ab1b-0c740559702e" />

---

## 8. Integración con MariaDB

Los vídeos de Jellyfin se sincronizan automáticamente con la base de datos MariaDB de Izan mediante el script `jellyfin_sync.py`.

**Flujo automático:**
```
Nuevo vídeo en Jellyfin → jellyfin_sync.py (cron cada 2h) → MariaDB → Portal web de Taylor
```

**Sincronización manual:**
```bash
cd /home/adminitb/multimedia_scripts
python3 jellyfin_sync.py
```

---

## 9. Validación del servicio

- Reproducción correcta del vídeo local desde el navegador web.
- Reproducción del stream HLS en directo de RTVE 24h.
- Acceso funcional desde diferentes dispositivos y usuarios.
- Sincronización automática con MariaDB cada 2 horas.

---

## 10. Incidencias y soluciones

| Incidencia | Solución aplicada |
|---|---|
| `Insufficient free space for /tmp` durante instalación | Ejecutar: `sudo mount -o remount,size=4G /tmp` antes de instalar |
| El vídeo no aparece en la biblioteca *Películas* | Cambiar tipo de biblioteca a *Vídeos y fotos personales* |
| `Permission denied` al hacer scp del vídeo | Descargar el vídeo directamente en el servidor con `wget` |
| La biblioteca no escanea el vídeo nuevo | Hacer clic en *Escanear todas las mediatecas* desde el panel de administración |
| Jellyfin crashea con error core-dump (restart counter 74+) | Borrar BD corrupta: `sudo rm -rf /var/lib/jellyfin/data` y reinstalar |
| Stream HLS da "Error de reproducció" en Jellyfin | Desactivar transcoding en la configuración del usuario en Jellyfin |
| Live TV / M3U Tuner no reproduce los canales | Usar archivos `.strm` en lugar de Live TV. Más ligero y sin errores de transcoding |
| URL de RTVE no accesible desde AWS | Usar la URL directa del stream HLS obtenida desde las herramientas de desarrollo del navegador (F12 → Network → filtrar m3u8) |

---

## 11. Security Group — SG-MULTIMEDIA

| Dirección | Protocolo | Puerto | Origen |
|---|---|---|---|
| Entrada | TCP | 8096 | 0.0.0.0/0 |
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Salida | TCP/UDP | 514 | 10.0.0.0/16 |

---

## 12. Comprobaciones de Ancho de Banda

### 12.1 Objetivo

Garantizar que la infraestructura desplegada es capaz de soportar los servicios de audio, vídeo y videoconferencia sin degradación del servicio.

### 12.2 Requisitos mínimos obligatorios

| | Requisito |
|---|---|
| ✅ | Realización de al menos 2 pruebas de ancho de banda |
| ✅ | Mostrar download, upload y latencia en cada prueba |
| ✅ | Relacionar los resultados con los tres servicios multimedia |
| ✅ | Clasificar el sistema como Aceptable o No aceptable |
| ✅ | Incluir evidencias (capturas) y conclusión técnica |

### 12.3 Herramientas utilizadas

```bash
sudo apt install iperf3 -y
```

| Herramienta | Función |
|---|---|
| `ping` | Mide la latencia hacia un destino específico |
| `curl` | Mide la velocidad de descarga contra un servidor externo |
| `dd + curl` | Mide la velocidad de subida enviando datos a un servidor externo |
| `iperf3` | Mide el rendimiento de red entre dos equipos de la infraestructura |

### 12.4 Prueba 1 - Sin servicios activos (línea base)

*Condiciones: ningún servicio multimedia activo. Medida de referencia.*

```bash
ping -c 20 8.8.8.8
curl -o /dev/null -w "Download: %{speed_download} bytes/s\n" https://proof.ovh.net/files/10Mb.dat
dd if=/dev/zero bs=1M count=10 | curl -o /dev/null -w "Upload: %{speed_upload} bytes/s\n" -X POST -T - https://httpbin.org/post
```

> <img width="729" height="472" alt="unnamed" src="https://github.com/user-attachments/assets/f24bbd6b-d3ba-4acd-ac1b-85f51dd7c7e6" />
> <img width="858" height="117" alt="unnamed" src="https://github.com/user-attachments/assets/21a6649a-46dc-4e09-ab3a-b7f2f87ecb8b" />
> <img width="854" height="195" alt="unnamed" src="https://github.com/user-attachments/assets/172b7e82-f45d-450b-9480-096bd3afbd23" />

| Medida | Valor obtenido |
|---|---|
| Download | ~47 Mbps (5.929.028 bytes/s) |
| Upload | ~8,5 Mbps (1.061.317 bytes/s) |
| Latencia (ping) | 1,97 ms |

### 12.5 Prueba 2 - Con todos los servicios activos

*Condiciones: Icecast2 emitiendo + Jellyfin reproduciendo vídeo + Jitsi Meet en videollamada activa.*

```bash
ping -c 20 8.8.8.8
curl -o /dev/null -w "Download: %{speed_download} bytes/s\n" https://proof.ovh.net/files/100Mb.dat
dd if=/dev/zero bs=1M count=10 | curl -o /dev/null -w "Upload: %{speed_upload} bytes/s\n" -X POST -T - https://httpbin.org/post
```

> <img width="854" height="413" alt="unnamed" src="https://github.com/user-attachments/assets/4b568cbd-9426-4eff-9807-140cc0c7d511" />

| Medida | Valor obtenido |
|---|---|
| Download | ~191 Mbps (23.897.610 bytes/s) |
| Upload | ~6,7 Mbps (843.120 bytes/s) |
| Latencia (ping) | 1,95 ms |

### 12.6 Análisis de resultados

| Servicio | Consumo estimado de red | Tipo de tráfico |
|---|---|---|
| Icecast2 (MP3 128kbps) | ~0,13 Mbps por oyente | Upload desde el servidor |
| Jellyfin (H.264 720p) | ~2–4 Mbps por stream | Download hacia el cliente |
| Jitsi Meet (720p, 2 usuarios) | ~1,5–3 Mbps por participante | Upload y download bidireccional |
| **Total estimado** | **~4–8 Mbps** | Combinado |

| Medida | Prueba 1 (sin servicios) | Prueba 2 (con servicios) | Diferencia |
|---|---|---|---|
| Download (Mbps) | ~47 | ~191 | +144 Mbps |
| Upload (Mbps) | ~8,5 | ~6,7 | -1,8 Mbps |
| Latencia (ms) | 1,97 | 1,95 | -0,02 ms |

### 12.7 Clasificación del sistema

✅ **ACEPTABLE** - La infraestructura soporta los tres servicios sin degradación significativa. La latencia se mantiene estable por debajo de los 2 ms en ambas pruebas. La velocidad de descarga es muy superior al consumo estimado de los servicios (~4–8 Mbps). La ligera bajada de upload con servicios activos es normal y no afecta al funcionamiento.

### 12.8 Propuestas de optimización

| Propuesta | Beneficio esperado |
|---|---|
| Aumentar tipo de instancia EC2 (`t3.micro` → `t3.medium`) | Más CPU y RAM para procesar streams simultáneos |
| Reducir bitrate Icecast2 de 128kbps a 96kbps | Menor consumo de ancho de banda manteniendo calidad aceptable |
| Separar Jellyfin en su propio EC2 | Cada servicio tiene recursos dedicados sin competencia |
| Configurar QoS en la VPC de AWS | Priorizar tráfico de videoconferencia sobre el resto |

Propuesta de mejora de calidad de servicio multimedia
Durante las pruebas de ancho de banda y funcionamiento de los servicios multimedia se detectó que la calidad del servicio no era completamente aceptable en determinadas situaciones de carga.
Los resultados obtenidos mostraron:
latencias elevadas,
limitaciones en la velocidad de subida,
degradación de calidad en streaming y videollamadas,
y consumo elevado de ancho de banda en algunos servicios multimedia.
Tras analizar el comportamiento de la infraestructura, se identificó que el principal problema estaba relacionado con el bitrate utilizado y el volumen de tráfico multimedia generado simultáneamente por Jellyfin, Icecast y Jitsi Meet.
Como propuesta de mejora para un entorno empresarial real, se plantean las siguientes medidas:
reducir el bitrate de streaming de audio en Icecast para disminuir el tamaño de los paquetes transmitidos,
reducir la calidad de vídeo en Jellyfin (por ejemplo de 1080p a 720p) para minimizar el consumo de ancho de banda,
aplicar compresión multimedia más eficiente mediante códecs optimizados,
implementar políticas QoS (Quality of Service) para priorizar tráfico crítico como videollamadas,
segmentar servicios multimedia en redes o VLAN independientes,
y ampliar la capacidad de red e infraestructura cloud en caso de alta concurrencia.
Estas medidas permitirían mejorar la estabilidad de la plataforma, reducir la latencia y optimizar el rendimiento general de los servicios multimedia desplegados en la infraestructura InnovateTech.
