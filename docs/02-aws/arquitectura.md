# Infraestructura AWS — InnovateTech


---
## Índice

1. [Visión general](#1-visión-general)
2. [Diagrama de la arquitectura](#2-diagrama-de-la-arquitectura)
3. [VPC y Red](#3-vpc-y-red)
4. [Instancias EC2](#4-instancias-ec2)
5. [EFS — Sistema de ficheros compartido](#5-efs--sistema-de-ficheros-compartido)
6. [Security Groups](#6-security-groups)
   - [SG-WEB](#sg-web)
   - [SG-MULTIMEDIA](#sg-multimedia)
   - [SG-JITSI](#sg-jitsi)
   - [SG-AD Samba AD](#sg-ad-samba-ad)
   - [SG-DB MariaDB](#sg-db-mariadb)
   - [SG-ANSIBLE](#sg-ansible)
   - [SG-LOGS](#sg-logs)
   - [SG-EFS](#sg-efs)
7. [Justificación de la arquitectura](#7-justificación-de-la-arquitectura)
   - [¿Por qué subnet pública y privada?](#por-qué-subnet-pública-y-privada)
   - [¿Por qué EFS en lugar de disco local para los logs?](#por-qué-efs-en-lugar-de-disco-local-para-los-logs)
   - [¿Por qué ALB delante de web-sftp?](#por-qué-alb-delante-de-web-sftp)
8. [Evidencias](#8-evidencias)
   - [Instancias EC2 en ejecución](#instancias-ec2-en-ejecución)
   - [VPC y Subnets](#vpc-y-subnets)
   - [EFS montado](#efs-montado)
   - [Security Groups](#security-groups)
## 1. Visión general

Infraestructura desplegada en AWS región `us-east-1` con VPC dedicada, subnets pública y privada, 7 instancias EC2, ALB, EFS compartido y servicios de red.

---

## 2. Diagrama de la arquitectura
<div align="center">

![Arquitectura AWS InnovateTech](https://github.com/user-attachments/assets/499953be-8c26-431a-8676-1be027ae1b61)

</div>

---

## 3. VPC y Red

| Parámetro | Valor |
|-----------|-------|
| Nombre | vpc-innovatetech-vpc |
| ID | vpc-0837b38c8e1749eeb |
| CIDR | 10.0.0.0/16 |
| Región | us-east-1 |
| DNS Hostnames | Activado |
| DNS Resolution | Activado |

### Subnets

| Nombre | ID | CIDR | Tipo | AZ |
|--------|----|------|------|----|
| innovatetech-subPublica | subnet-001c139d2c6686a83 | 10.0.0.0/20 | Pública | us-east-1a |
| innovatetech-subPublica2 | subnet-040af8871129f2119 | 10.0.50.0/24 | Pública | us-east-1b |
| innovatetech-subPrivada | subnet-0e35e1475b888ea55 | 10.0.128.0/20 | Privada | us-east-1a |

### Gateways y routing

| Recurso | Función |
|---------|---------|
| Internet Gateway | Conexión subnet pública a internet |
| NAT Gateway | Conexión subnet privada a internet (salida) |
| ALB alb-innovatetech | Load Balancer delante de web-sftp, HTTP→HTTPS |

---

## 4. Instancias EC2

| Máquina | IP Pública | IP Privada | Subnet | Key |
|---------|-----------|------------|--------|-----|
| web-sftp | 52.1.67.249 | 10.0.5.140 | Pública | T.pem |
| multimedia | 32.198.236.17 | 10.0.8.36 | Pública | S.pem |
| jitsi-meet | 3.219.249.6 | 10.0.14.189 | Pública | S.pem |
| ansible-controller | 32.193.193.146 | 10.0.7.201 | Pública | I.pem |
| samba-ad | — | 10.0.141.9 | Privada | T.pem |
| mariadb | — | 10.0.142.205 | Privada | I.pem |
| logs-server-private | — | 10.0.133.107 | Privada | I.pem |

### Configuración común

- SO: Ubuntu 24.04 LTS
- Disco: 8 GB gp3 (logs-server-private: 20 GB)
- Usuario administrador: `adminitb`
- Autenticación: clave pública/privada (sin contraseña)
- Sudo: sin contraseña

---

## 5. EFS — Sistema de ficheros compartido

| Parámetro | Valor |
|-----------|-------|
| Nombre | efs-innovatetech-logs |
| ID | fs-09f8489813ea7d132 |
| Mount target pública2 | 10.0.50.145 (subnet-040af8871129f2119) |
| Mount target privada | 10.0.142.148 (subnet-0e35e1475b888ea55) |
| Security Group | SG-EFS (sg-002a5243a5fb9a320) |
| Montado en | /mnt/efs-logs |
| Capacidad | 8.0E (elástica) |

El EFS es compartido entre `web-sftp` y `logs-server-private`. El servidor de logs escribe los logs de todas las EC2 en el EFS via rsyslog, y web-sftp los lee directamente para mostrarlos en el portal web.

---

## 6. Security Groups

### SG-WEB
| Dirección | Protocolo | Puerto | Origen/Destino |
|-----------|-----------|--------|----------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Entrada | TCP | 80 | 0.0.0.0/0 |
| Entrada | TCP | 443 | 0.0.0.0/0 |
| Salida | TCP | 3306 | 10.0.0.0/16 |
| Salida | TCP | 389 | 10.0.0.0/16 |
| Salida | TCP/UDP | 514 | 10.0.0.0/16 |
| Salida | TCP | 2049 | 10.0.0.0/16 |

### SG-MULTIMEDIA
| Dirección | Protocolo | Puerto | Origen/Destino |
|-----------|-----------|--------|----------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 8096 | 0.0.0.0/0 |
| Salida | Todo | Todo | 0.0.0.0/0 |

### SG-JITSI
| Dirección | Protocolo | Puerto | Origen/Destino |
|-----------|-----------|--------|----------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 8443 | 0.0.0.0/0 |
| Entrada | UDP | 10000 | 0.0.0.0/0 |
| Salida | TCP/UDP | 514 | 10.0.0.0/16 |

### SG-AD (Samba AD)
| Dirección | Protocolo | Puerto | Origen/Destino |
|-----------|-----------|--------|----------------|
| Entrada | TCP | 22 | 10.0.0.0/16 |
| Entrada | TCP/UDP | 88 | 10.0.0.0/16 |
| Entrada | TCP/UDP | 389 | 10.0.0.0/16 |
| Entrada | TCP | 445 | 10.0.0.0/16 |
| Entrada | TCP | 636 | 10.0.0.0/16 |
| Salida | TCP | 80/443 | 0.0.0.0/0 |

### SG-DB (MariaDB)
| Dirección | Protocolo | Puerto | Origen/Destino |
|-----------|-----------|--------|----------------|
| Entrada | TCP | 22 | 10.0.0.0/16 |
| Entrada | TCP | 3306 | 10.0.0.0/16 |
| Salida | TCP | 80/443 | 0.0.0.0/0 |

### SG-ANSIBLE
| Dirección | Protocolo | Puerto | Origen/Destino |
|-----------|-----------|--------|----------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Salida | TCP | 22 | 10.0.0.0/16 |
| Salida | TCP | 80/443 | 0.0.0.0/0 |

### SG-LOGS
| Dirección | Protocolo | Puerto | Origen/Destino |
|-----------|-----------|--------|----------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Entrada | TCP | 514 | 10.0.0.0/16 |
| Entrada | UDP | 514 | 10.0.0.0/16 |
| Entrada | ICMP | Todo | 10.0.0.0/16 |
| Salida | TCP | 80/443 | 0.0.0.0/0 |
| Salida | TCP | 2049 | 10.0.0.0/16 |

### SG-EFS
| Dirección | Protocolo | Puerto | Origen/Destino |
|-----------|-----------|--------|----------------|
| Entrada | TCP | 2049 | 0.0.0.0/0 |
| Entrada | Todo | Todo | 0.0.0.0/0 |
| Salida | TCP | 2049 | 0.0.0.0/0 |
| Salida | Todo | Todo | 0.0.0.0/0 |

---

## 7. Justificación de la arquitectura

### ¿Por qué subnet pública y privada?

Las máquinas que deben ser accesibles desde internet (web, multimedia, jitsi, ansible) están en la subnet pública. Las máquinas que solo deben ser accesibles internamente (samba-ad, mariadb, logs-server) están en la subnet privada con acceso a internet via NAT Gateway para actualizaciones.

### ¿Por qué EFS en lugar de disco local para los logs?

El EFS permite que `logs-server-private` escriba los logs y que `web-sftp` los lea simultáneamente sin necesidad de una API intermedia. Es elástico, no se puede llenar y tiene alta disponibilidad multi-AZ.

### ¿Por qué ALB delante de web-sftp?

El ALB permite añadir futuras instancias web en alta disponibilidad sin cambiar la URL pública. También gestiona la terminación SSL y redirige HTTP a HTTPS.

---

## 8. Evidencias

### Instancias EC2 en ejecución
<img width="1582" height="278" alt="instancias EC2" src="https://github.com/user-attachments/assets/03216ae4-d534-42e8-908b-a30de0119032" />

### VPC y Subnets
<img width="1593" height="193" alt="VPC" src="https://github.com/user-attachments/assets/9af54659-2c7e-4b91-bcfa-f4e768b21391" />
<img width="1596" height="251" alt="Subnets" src="https://github.com/user-attachments/assets/06956a98-ba05-4697-93f5-7b6892df5a18" />

### EFS montado
```bash
$ df -h | grep mnt
10.0.50.145:/   8.0E     0  8.0E   0% /mnt/efs-logs   # web-sftp
10.0.142.148:/  8.0E  623M  8.0E   1% /mnt/efs-logs   # logs-server-private
```
<img width="1547" height="852" alt="EFS" src="https://github.com/user-attachments/assets/51ff99e1-4311-4a54-88f1-8d49d8d9d506" />

### Security Groups
<img width="1595" height="482" alt="Security Groups" src="https://github.com/user-attachments/assets/8b904866-dd46-41a4-b022-a450bd47cc93" />
