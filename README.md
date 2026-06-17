# Proyecto Transversal ASIXc1 — InnovateTech CPD

> Diseño e implementación de un Centro de Procesamiento de Datos (CPD) en la nube AWS para la empresa **InnovateTech**, incluyendo servicios multimedia, base de datos, directorio activo y gestión centralizada de logs.

**Grupo:** G4 (Taylor · Izan · Bilal · Serhii)  
**Ciclo:** CFGS Administración de Sistemas Informáticos en Red  
**Centro:** Institut Tecnològic de Barcelona  
**Curso:** 2025–2026  
**Entrega:** 28 de mayo de 2026

---
# InnovateTech — Proyecto Transversal ASIX

## Índice general de documentación

### 01. CPD — Centro de Procesamiento de Datos

- [CPD — Diseño físico e infraestructura](docs/01-cpd/cpd.md)

---

### 02. AWS — Infraestructura cloud

- [Arquitectura AWS](docs/02-aws/arquitectura.md)
- [Automatización con Ansible](docs/02-aws/ansible.md)
- [Servidor Samba AD](docs/02-aws/samba-ad.md)
- [Servidor Web-SFTP](docs/02-aws/web-sftp.md)
- [Logs centralizados](docs/02-aws/logs.md)

---

### 03. Audio y vídeo

- [Icecast2 — Radio en streaming](docs/03-audio-video/Icecast2.md)
- [Jitsi Meet — Videoconferencia](docs/03-audio-video/Jitsi.md)
- [Jellyfin — Streaming de vídeo](docs/03-audio-video/multimedia.md)

---

### 04. Base de datos

- [MariaDB — Diseño, configuración y seguridad](docs/04-BaseDeDades/MariaDB.md)

---


## Estructura del repositorio

```text
docs/
├── 01-cpd/
│   └── cpd.md
│
├── 02-aws/
│   ├── ansible.md
│   ├── arquitectura.md
│   ├── logs.md
│   ├── samba-ad.md
│   └── web-sftp.md
│
├── 03-audio-video/
│   ├── Icecast2.md
│   ├── Jitsi.md
│   └── multimedia.md
│
└── 04-BaseDeDades/
    └── MariaDB.md
```
## Descripción del proyecto

InnovateTech es una empresa dedicada a la provisión de servicios tecnológicos que necesita modernizar su capacidad operativa. Este proyecto diseña e implementa un CPD eficiente en la nube AWS que integra:

- Distribución de contenidos de audio y vídeo en streaming (radio en directo + vídeo bajo demanda)
- Canal de vídeo en directo (live streaming)
- Sistema de videoconferencias para comunicación interna
- Base de datos para gestionar personal, comunicaciones y actividad
- Directorio Activo para centralizar la gestión de usuarios
- Servidor web (intranet) autenticado contra el Directorio Activo
- Gestión centralizada de logs de todos los servidores via rsyslog + EFS
- Configuración automatizada con Ansible

---

## Arquitectura AWS

<div align="center">

![Arquitectura AWS InnovateTech](https://github.com/user-attachments/assets/499953be-8c26-431a-8676-1be027ae1b61)

</div>


---

## Equipo y responsabilidades

| Miembro | Rol | Máquinas |
|---------|-----|----------|
| **Bilal** | Infraestructura AWS + Logs + Ansible | logs-server-private |
| **Izan** | Base de datos + Ansible | mariadb + ansible-controller |
| **Taylor** | Web + SFTP + Directorio Activo | web-sftp + samba-ad |
| **Serhii** | Multimedia + Videoconferencias | multimedia + jitsi-meet |

---

## Máquinas y servicios

| Máquina | IP Pública | IP Privada | Servicios | Subnet |
|---------|-----------|------------|-----------|--------|
| web-sftp | 52.1.67.249 | 10.0.5.140 | NGINX + PHP + SFTP + Portal web | Pública |
| multimedia | 32.198.236.17 | 10.0.8.36 | Jellyfin + Icecast2 + Live stream | Pública |
| jitsi-meet | 3.219.249.6 | 10.0.14.189 | Jitsi Meet (Docker) | Pública |
| ansible-controller | 32.193.193.146 | 10.0.7.201 | Ansible | Pública |
| samba-ad | — | 10.0.141.9 | Samba AD (dominio BTIS) | Privada |
| mariadb | — | 10.0.142.205 | MariaDB 10.11 | Privada |
| logs-server-private | — | 10.0.133.107 | Rsyslog + EFS | Privada |

---

## Stack tecnológico

| Componente | Tecnología | Función |
|------------|-----------|---------|
| Cloud | AWS (VPC, EC2, ALB, EFS, S3) | Infraestructura |
| SO | Ubuntu 24.04 LTS | Todas las instancias |
| Directorio Activo | Samba AD | Gestión de usuarios (dominio BTIS) |
| Servidor web | NGINX + PHP 8.3 | Intranet InnovateTech |
| Transferencia ficheros | SFTP + AD auth | Acceso ficheros por usuarios |
| Vídeo bajo demanda | Jellyfin | Distribución vídeo |
| Vídeo en directo | ffmpeg + HLS + NGINX | Canal live streaming |
| Radio en directo | Icecast2 + ffmpeg | Retransmisión RNE Radio 1 |
| Videoconferencias | Jitsi Meet (Docker) | Comunicación interna |
| Base de datos | MariaDB 10.11 | Gestión datos empresa |
| Automatización | Ansible | Configuración servidores |
| Logs centralizados | Rsyslog + EFS | Monitorización en tiempo real |
| Control de versiones | Git + GitHub | Documentación y código |

---

## Seguridad

| Capa | Medidas aplicadas |
|------|------------------|
| Red | Subnet privada para BD, AD y logs. Security Groups por servicio |
| Acceso SSH | Clave pública/privada, usuario `adminitb` dedicado, sin contraseña |
| Web | HTTPS, X-Frame-Options, X-XSS-Protection, X-Content-Type-Options |
| BD | Roles diferenciados, bind-address privado, prepared statements, triggers |
| Logs | Rsyslog centralizado → EFS, logrotate diario, dashboard admin |
| AWS | ALB delante de web, NAT Gateway para subnet privada |

---




## Pruebas de funcionamiento

| Prueba | Resultado |
|--------|----------|
| SSH con clave pública a todas las máquinas | ✅ |
| Conectividad entre subnets | ✅ |
| NAT Gateway subnet privada | ✅ |
| Logs centralizados (7 máquinas → EFS) | ✅ |
| Dashboard logs en el portal web | ✅ |
| Ansible ping todas las máquinas | ✅ |
| Ansible playbooks (logs_baseline, logs_clients, mariadb) | ✅ |
| Portal web con autenticación Samba AD | ✅ |
| HTTPS + certificado SSL | ✅ |
| Radio en directo (RNE Radio 1 via Icecast2) | ✅ |
| Jellyfin vídeo bajo demanda | ✅ |
| Jitsi Meet videoconferencia | ✅ |
| MariaDB roles y permisos | ✅ |
| SFTP autenticado contra AD | ✅ |
| Live streaming vídeo en directo | ✅ |
