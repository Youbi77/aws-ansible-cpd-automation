# Logs Centralizados — Rsyslog + EFS

---
## Índice

1. [Descripción](#1-descripción)
2. [Servidor de logs — logs-server-private](#2-servidor-de-logs--logs-server-private)
3. [Configuración de los clientes](#3-configuración-de-los-clientes-todas-las-ec2)
4. [web-sftp — Acceso al EFS para el portal web](#4-web-sftp--acceso-al-efs-para-el-portal-web)
5. [Verificación](#5-verificación)
6. [Estado del sistema](#6-estado-del-sistema)
7. [Dashboard de logs en el portal web](#7-dashboard-de-logs-en-el-portal-web)
8. [Evidencias](#8-evidencias)
## 1. Descripción

Sistema de logs centralizado basado en **rsyslog** con almacenamiento compartido en **AWS EFS**. Todas las instancias EC2 envían sus logs al servidor `logs-server-private` por TCP/UDP puerto 514. El servidor almacena los logs clasificados por hostname y programa en el sistema de ficheros EFS (`efs-innovatetech-logs`), accesible simultáneamente desde `web-sftp` para su visualización en el portal web.

### Arquitectura del sistema

```
EC2 (todas) ──rsyslog:514──► logs-server-private ──escribe──► EFS /mnt/efs-logs
                                                                      │
web-sftp ──lee──────────────────────────────────────────────────────►┘
Portal web muestra logs en tiempo real (admin.itb)
```

---

## 2. Servidor de logs — logs-server-private

### Instalación

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install rsyslog -y
sudo systemctl enable rsyslog
sudo systemctl start rsyslog
```

### Montaje EFS

```bash
sudo apt install nfs-common -y
sudo mkdir -p /mnt/efs-logs
sudo mount -t nfs4 -o nfsvers=4.1 10.0.142.148:/ /mnt/efs-logs

# Montaje permanente en fstab
echo "10.0.142.148:/ /mnt/efs-logs nfs4 nfsvers=4.1,_netdev 0 0" | sudo tee -a /etc/fstab
```

### Configuración rsyslog receptor

Editar `/etc/rsyslog.conf` y habilitar recepción TCP/UDP:

```bash
module(load="imudp")
input(type="imudp" port="514")
module(load="imtcp")
input(type="imtcp" port="514")
```

Añadir template para organizar logs por máquina en el EFS:

```bash
$template RemoteLogs,"/mnt/efs-logs/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs
```

### Permisos y AppArmor

```bash
# Permisos al directorio EFS
sudo chmod -R 777 /mnt/efs-logs/

# Desactivar AppArmor para rsyslog (necesario para escribir en el EFS)
sudo apt install apparmor-utils -y
sudo aa-disable /usr/sbin/rsyslogd

# Reiniciar rsyslog
sudo systemctl restart rsyslog
```

### Logrotate — rotación automática de logs

Configuración en `/etc/logrotate.d/efs-logs`:

```
/mnt/efs-logs/*/*.log {
    su root root
    daily
    rotate 7
    compress
    missingok
    notifempty
    size 50M
    copytruncate
}
```

---

## 3. Configuración de los clientes (todas las EC2)

En cada máquina crear `/etc/rsyslog.d/99-remote.conf`:

```bash
echo "*.* @@10.0.133.107:514" | sudo tee /etc/rsyslog.d/99-remote.conf
sudo systemctl restart rsyslog
```

> `@@` indica TCP (más fiable que UDP `@`)

---

## 4. web-sftp — Acceso al EFS para el portal web

```bash
sudo apt install nfs-common -y
sudo mkdir -p /mnt/efs-logs
sudo mount -t nfs4 -o nfsvers=4.1 10.0.50.145:/ /mnt/efs-logs

# Permisos para www-data
sudo chmod -R 755 /mnt/efs-logs/

# Montaje permanente
echo "10.0.50.145:/ /mnt/efs-logs nfs4 nfsvers=4.1,_netdev 0 0" | sudo tee -a /etc/fstab
```

---

## 5. Verificación

### Puertos escuchando en el servidor de logs

```bash
$ sudo ss -tlnup | grep 514
udp   UNCONN  0.0.0.0:514   users:(("rsyslogd"))
tcp   LISTEN  0.0.0.0:514   users:(("rsyslogd"))
```

### EFS montado correctamente

```bash
$ df -h | grep mnt
10.0.142.148:/  8.0E  623M  8.0E  1%  /mnt/efs-logs   # logs-server-private
10.0.50.145:/   8.0E    0   8.0E  0%  /mnt/efs-logs   # web-sftp
```

### Logs recibidos de todas las máquinas

```bash
$ ls /mnt/efs-logs/
ansible-controller  jitsi-meet  logs-server-private  mariadb  multimedia  samba-ad  web-sftp
```

---

## 6. Estado del sistema

| Máquina | IP | Logs en EFS | Estado |
|---------|----|-------------|--------|
| web-sftp | 10.0.5.140 | ✅ | Activo |
| multimedia | 10.0.8.36 | ✅ | Activo |
| jitsi-meet | 10.0.14.189 | ✅ | Activo |
| ansible-controller | 10.0.7.201 | ✅ | Activo |
| samba-ad | 10.0.141.9 | ✅ | Activo |
| mariadb | 10.0.142.205 | ✅ | Activo |
| logs-server-private | 10.0.133.107 | ✅ | Activo |

---

## 7. Dashboard de logs en el portal web

El portal web (`admin.itb`) muestra los logs en tiempo real con las siguientes funcionalidades:

- Visualización de los últimos 8 registros por fichero y servidor
- **Filtros** por tipo: Todos / Errores / Accesos OK / Warnings / SSH / Sudo
- **Buscador** en tiempo real por todos los servidores
- Conversión automática de timestamps UTC → Europe/Madrid
- Color rojo para errores, verde para accesos correctos, amarillo para warnings

### Acceso

1. Entrar al portal web con `admin.itb`
2. Desplazarse hasta la sección **Logs del sistema** al final del dashboard

---

## 8. Evidencias

### Logs recibidos en el EFS
<img width="912" height="618" alt="Logs en EFS" src="https://github.com/user-attachments/assets/5a76e6b0-5c09-4b3c-84be-145179c9eb4e" />

### Dashboard de logs en el portal web
<img width="1828" height="934" alt="Dashboard logs" src="https://github.com/user-attachments/assets/3396af65-9bfa-4efe-9b3c-40de95cab54d" />

### rsyslog escuchando en el puerto 514
<img width="926" height="117" alt="rsyslog puerto 514" src="https://github.com/user-attachments/assets/35746984-d894-4f71-8160-516ee0434324" />
