# Propuesta de CPD — InnovateTech


---
## Índice

1. [Ubicación física](#1-ubicación-física)
   - [1.1 Situación de la sala en el edificio](#11-situación-de-la-sala-en-el-edificio)
   - [1.2 Climatización](#12-climatización)
   - [1.3 Medidas para dificultar la identificación de la sala](#13-medidas-para-dificultar-la-identificación-de-la-sala)
   - [1.4 Distribución y gestión del cableado](#14-distribución-y-gestión-del-cableado)
   - [1.5 Suelo técnico y techo técnico](#15-suelo-técnico-y-techo-técnico)
   - [1.6 Plano de la sala CPD](#16-plano-de-la-sala-cpd)
   - [1.7 Estructuración de los racks](#17-estructuración-de-los-racks)

2. [Infraestructura IT](#2-infraestructura-it)
   - [2.1 Servidores](#21-servidores)
   - [2.2 Patch panels](#22-patch-panels)
   - [2.3 Switches](#23-switches)

3. [Infraestructura eléctrica](#3-infraestructura-eléctrica)
   - [3.1 Alimentación redundante](#31-alimentación-redundante)
   - [3.2 SAI — Sistemas de Alimentación Ininterrumpida](#32-sai--sistemas-de-alimentación-ininterrumpida)

4. [Seguridad física](#4-seguridad-física)
   - [4.1 Control de acceso](#41-control-de-acceso)
   - [4.2 Videovigilancia](#42-videovigilancia)
   - [4.3 Sistemas de prevención y extinción de incendios](#43-sistemas-de-prevención-y-extinción-de-incendios)
   - [4.4 Vías de evacuación](#44-vías-de-evacuación)
     
## 1. Ubicación física

### 1.1 Situación de la sala en el edificio

El CPD de InnovateTech se encuentra en la **primera planta interior** del edificio corporativo, orientada hacia el patio interior, sin fachada exterior y alejada de zonas de paso público.

#### Justificación de la ubicación

La primera planta interior ha sido elegida tras evaluar las alternativas:

| Ubicación | Riesgo inundación | Riesgo acceso | Temperatura | Decisión |
|-----------|------------------|---------------|-------------|----------|
| Sótano | ❌ Alto | ✅ Bajo | ✅ Estable | **Descartado** |
| Planta baja | ⚠️ Medio | ❌ Alto | ⚠️ Variable | **Descartado** |
| **1ª planta interior** | ✅ Nulo | ✅ Bajo | ✅ Estable | **✅ Seleccionado** |
| Última planta | ✅ Nulo | ✅ Bajo | ❌ Calor | **Descartado** |

**Ventajas de la primera planta interior:**

- **Riesgo de inundación nulo**: elevado sobre el nivel de la calle, protegido de filtraciones de agua.
- **Seguridad**: difícil acceso desde el exterior, sin ventanas hacia la calle.
- **Temperatura estable**: la orientación interior evita la incidencia solar directa.
- **Acceso para mantenimiento**: accesible sin ascensor, facilita el transporte de equipos pesados.
- **Estructura del edificio**: las plantas intermedias soportan mejor el peso de los racks (hasta 1.200 kg/m²).

La sala ocupa **50 m²** con dimensiones de 10m x 5m y altura libre de 3m (2,5m útiles con suelo técnico y techo técnico).

<img width="776" height="781" alt="image" src="https://github.com/user-attachments/assets/0d77ced9-da80-45ee-bba8-e9353a88d0d0" />



### 1.2 Climatización

El sistema de climatización es crítico para mantener los equipos en condiciones óptimas de funcionamiento.

#### Parámetros ambientales objetivo

| Parámetro | Valor óptimo | Rango aceptable |
|-----------|-------------|-----------------|
| Temperatura | 21°C | 18°C – 27°C |
| Humedad relativa | 45% | 40% – 60% |
| Partículas en suspensión | < 100.000 p/m³ | — |

#### Sistema instalado

- **2 unidades de climatización de precisión** marca APC InRow RC (9 kW cada una) en configuración N+1 (una de redundante).
- **Sistema de flujo de aire** de abajo a arriba: aire frío entra por el suelo técnico y el aire caliente sale por el techo técnico hacia los conductos de extracción.
- **Configuración de pasillo frío/caliente**: los racks están orientados cara a cara para separar los pasillos de aire frío (delante de los racks) y caliente (detrás de los racks).
- **Sensores ambientales** en 3 puntos de la sala: entrada, centro y salida.
- **Sistema de filtrado HEPA** para eliminar partículas y polvo.

<img width="1027" height="562" alt="image" src="https://github.com/user-attachments/assets/5f36ec8f-5f60-4d06-bc43-443c36d1916d" />


### 1.3 Medidas para dificultar la identificación de la sala

- **Señalización neutra**: la puerta no indica "CPD" ni "Sala de servidores". Aparece identificada como "Mantenimiento Técnico P1-03".
- **Sin ventanas** hacia el exterior ni hacia el pasillo principal.
- **Puerta blindada** sin manilla exterior, con apertura únicamente por tarjeta RFID.
- **Cableado y alimentación enterrados**: no hay cableado visible en los pasillos que pueda indicar la ubicación.
- **Muros reforzados**: las paredes de la sala son de hormigón armado de 20cm, sin indicación visual de su función.

---

### 1.4 Distribución y gestión del cableado

#### Principios de gestión del cableado

- **Separación física** de cableado de red (azul) y cableado eléctrico (rojo/negro) para evitar interferencias.
- **Etiquetado** de todos los cables en los dos extremos con código alfanumérico.
- **Bridas y canales** para mantener el cableado ordenado y accesible.
- **Color coding**:
  - Azul: red de datos (LAN)
  - Verde: conexiones entre switches
  - Amarillo: conexiones de gestión (IPMI/iLO)
  - Rojo: alimentación eléctrica
  - Negro: alimentación SAI

#### Distribución

```
SUELO TÉCNICO
├── Canal 1 (lado izquierdo): Cableado de red
│   ├── Patch cables Cat6A (azul)
│   └── Fibra óptica (amarillo)
├── Canal 2 (lado derecho): Alimentación eléctrica
│   ├── Cables PDU (rojo)
│   └── Cables SAI (negro)
└── Canal central: Conexiones entre racks
```
<img width="1027" height="562" alt="image" src="https://github.com/user-attachments/assets/b24b6b09-8aa1-4838-9151-798630e26db8" />
<img width="1027" height="562" alt="image" src="https://github.com/user-attachments/assets/00514efd-4aca-4f3f-9fa3-cebfc3fd0fb5" />




---

### 1.5 Suelo técnico y techo técnico

#### Suelo técnico

- **Altura**: 40 cm sobre el suelo estructural.
- **Placas**: 60x60 cm de acero galvanizado con acabado antiestático.
- **Función**: distribución de aire frío desde los climatizadores hacia la parte frontal de los racks, y canalización del cableado.
- **Carga máxima**: 1.200 kg/m².

#### Techo técnico

- **Altura**: 30 cm entre falso techo y techo estructural.
- **Función**: retorno de aire caliente a los climatizadores y paso de las instalaciones (eléctrica, detección de incendios, cableado de gestión).
- **Placas desmontables** para facilitar el mantenimiento.

```
┌─────────────────────────────────┐ ← Techo estructural (3m)
│    Conductos retorno aire cal.  │
│    Cables gestión               │
├─────────────────────────────────┤ ← Falso techo (2.7m)
│                                 │
│          ZONA RACKS              │  2.5m útiles
│                                 │
├─────────────────────────────────┤ ← Suelo técnico (0.4m)
│    Cables red / eléctricos      │
│    Distribución aire frío       │
└─────────────────────────────────┘ ← Suelo estructural (0m)
```

---
<img width="1027" height="562" alt="image" src="https://github.com/user-attachments/assets/79e3bf16-0281-4c17-b632-4fd60e759593" />


### 1.6 Plano de la sala CPD

<img width="1027" height="562" alt="image" src="https://github.com/user-attachments/assets/3dd05606-9795-4cd5-a16e-a1d4a66784aa" />


---

### 1.7 Estructuración de los racks

#### Rack 1 — Servidores y red

| U | Equipo |
|---|--------|
| 1-2 | Patch panel Cat6A (48 puertos) |
| 3 | Switch core (Cisco Catalyst 9300) |
| 4 | Switch acceso (Cisco Catalyst 2960) |
| 5 | Firewall (Fortinet FortiGate 100F) |
| 6-8 | Servidor web-sftp (Dell PowerEdge R350) |
| 9-11 | Servidor multimedia (Dell PowerEdge R350) |
| 12-14 | Servidor Jitsi (Dell PowerEdge R350) |
| 15-17 | Servidor Ansible (Dell PowerEdge R350) |
| 18-20 | Servidor Samba AD (Dell PowerEdge R350) |
| 40-42 | PDU vertical (APC AP8959) |

#### Rack 2 — Bases de datos, logs y almacenamiento

| U | Equipo |
|---|--------|
| 1-2 | Patch panel Cat6A (48 puertos) |
| 3 | Switch gestión (Cisco Catalyst 2960) |
| 4-6 | Servidor MariaDB (Dell PowerEdge R350) |
| 7-9 | Servidor logs (Dell PowerEdge R350) |
| 10-14 | NAS (Synology RS3621xs+, 12 bahías) |
| 15-16 | Servidor de copias de seguridad |
| 40-42 | PDU vertical (APC AP8959) |

<img width="1027" height="562" alt="image" src="https://github.com/user-attachments/assets/9be05d7d-6d15-4c8b-83b8-c57fe79dd965" />



---

## 2. Infraestructura IT

### 2.1 Servidores

| Servidor | Modelo | CPU | RAM | Disco | Función |
|----------|--------|-----|-----|-------|---------|
| web-sftp | Dell PowerEdge R350 | Intel Xeon E-2334 | 16 GB | 2x480GB SSD RAID1 | Web + SFTP |
| multimedia | Dell PowerEdge R350 | Intel Xeon E-2334 | 32 GB | 2x960GB SSD RAID1 | Icecast + Jellyfin |
| jitsi-meet | Dell PowerEdge R350 | Intel Xeon E-2334 | 16 GB | 2x480GB SSD RAID1 | Videoconferencia |
| ansible | Dell PowerEdge R350 | Intel Xeon E-2334 | 16 GB | 2x480GB SSD RAID1 | Automatización |
| samba-ad | Dell PowerEdge R350 | Intel Xeon E-2334 | 16 GB | 2x480GB SSD RAID1 | Directorio activo |
| mariadb | Dell PowerEdge R650 | Intel Xeon Silver 4310 | 64 GB | 4x960GB SSD RAID10 | Base de datos |
| logs-server | Dell PowerEdge R350 | Intel Xeon E-2334 | 16 GB | 2x480GB SSD RAID1 | Logs centralizados |

### 2.2 Patch panels

- **2x Patch panel Cat6A de 48 puertos** (uno por rack)
- Conexión entre los puertos del patch panel y los puertos de los switches mediante patch cables cortos (0,5m) para minimizar el cableado dentro del rack.
- Etiquetado de cada puerto con la máquina o dispositivo conectado.

### 2.3 Switches

| Switch | Modelo | Puertos | Función |
|--------|--------|---------|---------|
| Switch core | Cisco Catalyst 9300-24T | 24x1G + 4x10G uplink | Interconexión entre racks y uplink internet |
| Switch acceso 1 | Cisco Catalyst 2960-24TC | 24x1G | Conexión servidores Rack 1 |
| Switch acceso 2 | Cisco Catalyst 2960-24TC | 24x1G | Conexión servidores Rack 2 |
| Switch gestión | Cisco Catalyst 2960-8TC | 8x1G | Gestión IPMI/iLO de los servidores |

---

## 3. Infraestructura eléctrica

### 3.1 Alimentación redundante

- **2 líneas eléctricas independientes** desde el cuadro general del edificio.
- Cada rack dispone de **2 PDUs** (Power Distribution Units) conectadas a líneas eléctricas diferentes.
- Todos los servidores tienen **2 fuentes de alimentación** redundantes (PSU1 → Línea A, PSU2 → Línea B).

<img width="1027" height="562" alt="image" src="https://github.com/user-attachments/assets/c2989b78-186b-4723-95ed-050f92c4cc8a" />


### 3.2 SAI — Sistemas de Alimentación Ininterrumpida

#### Modelo seleccionado

**APC Symmetra LX 16 kVA** con baterías externas.

#### Cálculo de consumo

| Equipo | Consumo estimado |
|--------|-----------------|
| 7 servidores Dell PowerEdge R350 | 7 × 350W = 2.450W |
| 2 switches Cisco Catalyst | 2 × 370W = 740W |
| 1 switch core Cisco 9300 | 1 × 715W = 715W |
| 2 climatizadores APC InRow | 2 × 200W (ventiladores) = 400W |
| 1 NAS Synology | 1 × 100W = 100W |
| **TOTAL** | **4.405W** |
| **Margen de seguridad (+20%)** | **5.286W ≈ 5,3 kW** |

#### Autonomía

- **SAI APC Symmetra LX 16 kVA** con 2 módulos de baterías externos (SYBATT).
- Cada módulo proporciona ~15 minutos a plena carga.
- Con 2 módulos: **~30 minutos de autonomía** a plena carga.
- Tiempo suficiente para un apagado ordenado de todos los servidores o para que el grupo electrógeno entre en funcionamiento.

> **Justificación**: 30 minutos es el tiempo estándar recomendado para CPDs de tamaño medio, suficiente para arrancar un generador de backup o hacer un apagado controlado.

---

## 4. Seguridad física

### 4.1 Control de acceso

- **Puerta blindada** con cierre electrónico y apertura por doble factor:
  1. Tarjeta RFID (HID iClass)
  2. PIN de 6 dígitos
- **Registro de accesos**: todo acceso queda registrado con fecha, hora e identidad.
- **Lista blanca**: únicamente personal autorizado explícitamente puede acceder.
- **Acceso de emergencia**: llave física en caja fuerte en el despacho del responsable IT.
- **Alarma**: activación automática si la puerta se abre sin autenticación o queda abierta más de 60 segundos.

### 4.2 Videovigilancia

- **4 cámaras IP** de 4K (Axis P3245-V) distribuidas:
  - 1 en la entrada de la sala
  - 1 en el pasillo frío (delante de los racks)
  - 1 en el pasillo caliente (detrás de los racks)
  - 1 en la zona del cuadro eléctrico y SAI
- **Grabación**: 24/7 con retención de 30 días en NAS dedicado.
- **Alertas**: detección de movimiento con notificación por correo electrónico.
- **Acceso remoto**: visualización en tiempo real desde el sistema de gestión IT.

<img width="1033" height="566" alt="image" src="https://github.com/user-attachments/assets/3b2202c3-286e-4e2d-88fa-03696e127476" />



### 4.3 Sistemas de prevención y extinción de incendios

- **Detectores de humo** iónicos y ópticos distribuidos cada 9m² (6 unidades).
- **Detectores de temperatura** con alarma a 60°C.
- **Sistema de extinción por gas inerte** (Inergen IG-541, 52% N₂, 40% Ar, 8% CO₂):
  - No daña los equipos electrónicos.
  - Seguro para personas (reducción de O₂ hasta el 12,5%).
  - Activación automática a los 30 segundos de detección confirmada.
  - Activación manual desde el panel de control en la entrada.
- **Extintores manuales** CO₂ (2 unidades) para actuación manual.
- **Panel de control de incendios** en la entrada conectado al sistema central del edificio.

### 4.4 Vías de evacuación

- **Puerta principal** (única entrada/salida): apertura en modo pánico desde el interior.
- **Señalización fotoluminiscente** de la ruta de evacuación.
- **Iluminación de emergencia** autónoma con batería de 3 horas.
- **Punto de encuentro** designado en el aparcamiento exterior, a 50m del edificio.

<img width="1099" height="727" alt="image" src="https://github.com/user-attachments/assets/aac70c1f-d941-4ced-b3a8-6de3af722d00" />


## 5. Seguridad lógica

### 5.1 Restricción de acceso por autorización

- **Autenticación centralizada** via Samba AD (dominio BTIS).
- **Roles diferenciados**: cada usuario tiene acceso únicamente a los datos de su departamento.
- **Acceso SSH** únicamente con clave pública/privada, sin contraseñas.
- **Usuario administrador**: `adminitb` (específico del proyecto, no el usuario por defecto).
- **Sudo sin contraseña** para el usuario adminitb con registro de todas las acciones.

### 5.2 Firewalls — Security Groups AWS

Cada instancia EC2 tiene su Security Group específico con el **principio de mínimo privilegio**:

- Las máquinas de la subnet privada (samba-ad, mariadb, logs-server) no tienen IP pública y solo aceptan tráfico interno.
- Los puertos abiertos al público se limitan al mínimo necesario (80, 443, 22).
- El servidor de bases de datos (MariaDB) solo acepta conexiones desde la VPC interna (puerto 3306).

Ver la documentación completa de los Security Groups en `arquitectura.md`.

### 5.3 Monitorización

- **Logs centralizados** via rsyslog en el servidor `logs-server-private`, almacenados en el EFS.
- **Dashboard de logs** en el portal web (accesible únicamente por `admin.itb`).
- **CloudWatch** de AWS para métricas de CPU, memoria y red de las EC2.
- **Alertas automáticas** via SNS cuando el disco supera el 80% de uso.
- **Logrotate** configurado para rotar logs diariamente y mantener 7 días de historial.

### 5.4 Copias de seguridad

- **Backup diario automático** de MariaDB via event scheduler:
  ```sql
  CREATE EVENT ev_backup_diario
  ON SCHEDULE EVERY 1 DAY
  DO SELECT * INTO OUTFILE '/var/lib/mysql-files/backup_...'
  ```
- **Tabla `backups_control`** en MariaDB para registrar el estado de cada backup.
- **S3 bucket** `s3-innovatetech-media` para copias de seguridad externas.
- **Snapshots EBS** semanales de todas las instancias EC2 via AWS Backup.

### 5.5 RAIDs

Todos los servidores físicos del CPD incorporan RAID para garantizar la disponibilidad de datos:

| Servidor | RAID | Configuración | Justificación |
|----------|------|---------------|---------------|
| Servidores generales | RAID 1 | 2x SSD espejo | Redundancia sin pérdida de rendimiento |
| MariaDB | RAID 10 | 4x SSD | Máximo rendimiento + redundancia para BD |
| NAS backup | RAID 6 | 12x HDD | Tolerancia a 2 fallos simultáneos |

> **Justificación RAID 1 para servidores**: Copia exacta en tiempo real. Si un disco falla, el servidor continúa funcionando sin interrupción.

> **Justificación RAID 10 para MariaDB**: La base de datos es el componente más crítico. RAID 10 combina la velocidad de RAID 0 (striping) con la redundancia de RAID 1 (mirroring), ideal para cargas de lectura/escritura intensas.

---

## 6. Prevención de riesgos laborales

### 6.1 Riesgos identificados en el CPD

| Riesgo | Medida preventiva |
|--------|------------------|
| Caídas por cableado en el suelo | Suelo técnico elevado, cableado canalizado |
| Descargas eléctricas | Todas las instalaciones por profesional certificado, señalización de peligro eléctrico |
| Ruido de los equipos (>85 dB) | EPI obligatorio (protectores auditivos) para trabajos de >30 min en el CPD |
| Temperatura extrema | Sensores y alarmas ambientales, acceso limitado a personal formado |
| Incendio | Detectores, extinción automática, simulacros anuales |
| Atrapamiento en el interior | Puerta abre desde el interior sin código, sistema antipánico |
| Exposición a gases de extinción | Protocolo de evacuación previo a activación automática (retardo 30s) |
| Sobrecarga de racks | Racks certificados por carga máxima, distribución de peso documentada |

### 6.2 Medidas generales

- **Formación obligatoria** para todo el personal que acceda al CPD: riesgos eléctricos, protocolo de incendios y evacuación.
- **Trabajo en solitario prohibido**: siempre mínimo 2 personas durante intervenciones en los equipos.
- **EPIs disponibles** en la entrada: guantes dieléctricos, gafas de protección, protectores auditivos.
- **Señalización** de riesgos eléctricos, cargas pesadas y zonas de paso restringido.
- **Simulacros de evacuación** al menos 1 vez al año.
- **Registro de mantenimiento** documentado para todas las intervenciones.

---

## 7. Implementación en la nube AWS

### 7.1 Servicios implementados

| Servicio | Máquina | IP Privada | Tecnología |
|----------|---------|------------|------------|
| Servicio web | web-sftp | 10.0.5.140 | NGINX + PHP + HTTPS |
| Servicio SFTP | web-sftp | 10.0.5.140 | OpenSSH SFTP + Samba AD |
| Logs centralizados | logs-server-private | 10.0.133.107 | rsyslog + EFS |
| Directorio activo | samba-ad | 10.0.141.9 | Samba AD (dominio BTIS) |
| Base de datos | mariadb | 10.0.142.205 | MariaDB 10.11 |
| Audio/streaming | multimedia | 10.0.8.36 | Icecast2 + ffmpeg |
| Vídeo | multimedia | 10.0.8.36 | Jellyfin |
| Videoconferencia | jitsi-meet | 10.0.14.189 | Jitsi Meet (Docker) |

### 7.2 Servicios por máquina separada

Cada servicio principal está instalado en una instancia EC2 diferente, cumpliendo el requisito del proyecto:

- ✅ Servicio web + SFTP → `web-sftp` (excepción permitida)
- ✅ Logs centralizados → `logs-server-private`
- ✅ Directorio activo → `samba-ad`
- ✅ Base de datos → `mariadb`
- ✅ Audio/Vídeo → `multimedia`
- ✅ Videoconferencia → `jitsi-meet`
- ✅ Automatización → `ansible-controller`

### 7.3 Automatización con Ansible

Más de 2 máquinas configuradas con Ansible:

- ✅ `logs-server-private`: rsyslog receptor + EFS montado
- ✅ `web-sftp`: rsyslog cliente
- ✅ `mariadb`: MariaDB + rsyslog cliente
- ✅ Todas las EC2: rsyslog cliente via playbook `logs_clients.yml`

### 7.4 Administración con usuario específico

- Usuario: `adminitb` (no el usuario por defecto `ubuntu`)
- Autenticación: clave pública/privada (I.pem, S.pem, T.pem)
- Acceso SSH sin contraseña
- Sudo sin contraseña para operaciones de administración
