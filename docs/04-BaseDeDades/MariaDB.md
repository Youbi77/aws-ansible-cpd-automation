# Base de Datos — InnovateTech
---
## Índice

- [1. Contexto](#1-contexto)
- [2. Información general](#2-información-general)
- [3. Instalación de MariaDB](#3-instalación-de-mariadb)
- [4. Creación de la base de datos](#4-creación-de-la-base-de-datos)
- [5. Diseño Entidad-Relación](#5-diseño-entidad-relación)
- [6. Modelo relacional](#6-modelo-relacional)
- [7. Creación de tablas](#7-creación-de-tablas)
- [8. Estructura de tablas](#8-estructura-de-tablas)
- [9. Datos de prueba](#9-datos-de-prueba)
- [10. Plantilla de mediciones de ancho de banda](#10-plantilla-de-mediciones-de-ancho-de-banda)
- [11. Roles y permisos](#11-roles-y-permisos)
- [12. Usuario web con permisos mínimos](#12-usuario-web-con-permisos-mínimos)
- [13. Script automatizado de usuarios](#13-script-automatizado-de-usuarios)
- [14. Triggers de auditoría](#14-triggers-de-auditoría)
- [15. Events y backups automáticos](#15-events-y-backups-automáticos)
- [16. Seguridad aplicada](#16-seguridad-aplicada)
- [17. Validación final](#17-validación-final)
- [18. Plantilla de mediciones de ancho de banda](#18-plantilla-de-mediciones-de-ancho-de-banda)
- [19. Conclusión](#19-conclusión)

---

# 1. Contexto

InnovateTech necesita una base de datos centralizada para gestionar:

- empleados,
- departamentos,
- videollamadas,
- streaming multimedia,
- auditoría,
- backups,
- medidas de ancho de banda.

La solución implementada utiliza MariaDB sobre una instancia EC2 privada.

---

# 2. Información general

| Elemento | Valor |
|---|---|
| SGBD | MariaDB |
| Base de datos | innovatetech |
| Servidor | EC2 privada |
| IP privada | 10.0.142.205 |

---

# 3. Instalación de MariaDB

Instalación automatizada mediante Ansible.

Playbook utilizado:

```bash
playbooks/mariadb.yml
```

📸 Captura:

<img width="1036" height="906" alt="imatge" src="https://github.com/user-attachments/assets/91368929-6df0-4965-bdc0-4e88f5047608" />

---

# 4. Creación de la base de datos

Comandos utilizados:

```sql
SHOW DATABASES;
USE innovatetech;
```

📸 Captura:

<img width="609" height="329" alt="imatge" src="https://github.com/user-attachments/assets/15fd4481-9d41-45f7-9ef2-61125ceb4d59" />

---

# 5. Diseño Entidad-Relación

El modelo E/R representa las entidades principales del sistema:

- empleats,
- departaments,
- clients,
- trucades,
- videos,
- auditoría,
- backups.

📸 Captura:

<img width="985" height="582" alt="Captura de pantalla de 2026-05-26 08-12-42" src="https://github.com/user-attachments/assets/65681468-798a-48c7-8c2b-3c151fd4e0e0" />

---

# 6. Modelo relacional

| Tabla | Atributos | PK | FK |
|---|---|---|---|
| departaments | codi, nom, telefon | codi | - |
| empleats | dni, nom, cognoms, adreca, telefon, departament | dni | departament → departaments(codi) |
| nomines | id, dni_empleat, salari, data_nomina | id | dni_empleat → empleats(dni) |
| usuaris_comunicacio | id, nom, email, extensio, estat, tipus, quota_minuts_mes | id | - |
| trucades | id, origen, destinatari, inici, fi, durada, qualitat | id | origen → usuaris_comunicacio(id), destinatari → usuaris_comunicacio(id) |
| videos | id, titol, descripcio, categoria, durada, data_publicacio, enllac | id | - |
| mesures_amplada_banda | id, data_mesura, usuari, download_mbps, upload_mbps, latencia_ms, resultat, observacions | id | - |
| avisos | id, usuari, taula_afectada, operacio, data_hora, descripcio | id | - |
| backups_control | id, data_hora, taules, resultat, ruta | id | - |
| clients | id, nom, email, telefon | id | - |
| productes | id, nom, preu | id | - |
| comandes | id, client_id, data_comanda | id | client_id → clients(id) |
| cistell | id, client_id, producte_id, quantitat | id | client_id → clients(id), producte_id → productes(id) |

📸 Captura:

<img width="1536" height="1024" alt="imatge" src="https://github.com/user-attachments/assets/df8337cc-49ad-4be5-9fee-035d5a6f4871" />

---

# 7. Creación de tablas

Archivo SQL utilizado:

```bash
files/sql/schema.sql
```

Comandos de validación:

```sql
SHOW TABLES;
```

📸 Captura:

<img width="336" height="353" alt="imatge" src="https://github.com/user-attachments/assets/865fb565-8834-4f5e-98ec-b11a52a307c5" />

---

# 8. Estructura de tablas

Comandos utilizados:

```sql
DESCRIBE empleats;
DESCRIBE trucades;
SHOW CREATE TABLE trucades;
```

📸 Captura:

<img width="886" height="474" alt="imatge" src="https://github.com/user-attachments/assets/35294af7-b876-4755-8e01-8353744f42a8" />

<img width="1091" height="726" alt="imatge" src="https://github.com/user-attachments/assets/55d888fd-e768-4d15-bc5e-87c9ed5d4026" />

---

# 9. Datos de prueba

Archivo SQL utilizado:

```bash
files/sql/seed.sql
```

Consultas realizadas:

```sql
SELECT * FROM empleats;
SELECT * FROM videos;
SELECT * FROM trucades;
SELECT * FROM clients;
```

📸 Captura:

<img width="1086" height="709" alt="imatge" src="https://github.com/user-attachments/assets/884f928f-f7dc-4a55-9cc8-3ab25d3f961e" />

---
# 10. Plantilla de mediciones de ancho de banda

Para facilitar la integración entre el equipo audiovisual y la base de datos de InnovateTech, se ha definido una plantilla estándar en formato CSV para registrar las pruebas de ancho de banda realizadas sobre los servicios de streaming y videoconferencia.

La plantilla permite almacenar:

- fecha y hora de la medición,
- usuario u operario responsable,
- velocidad de descarga,
- velocidad de subida,
- latencia,
- resultado de la prueba,
- observaciones adicionales.

Formato utilizado:

```csv
data_mesura,usuari,download_mbps,upload_mbps,latencia_ms,resultat,observacions
2026-05-21 10:30:00,operari_audiovisual,850,420,2,acceptable,Streaming 1080p estable
```

Archivo utilizado:

```bash
files/templates/mesures_amplada_banda_template.csv
```

📸 Captura:

<img width="897" height="59" alt="imatge" src="https://github.com/user-attachments/assets/226b18cf-6ee1-4f9a-970c-35f23883d716" />

---

# 11. Roles y permisos

Roles implementados:

| Rol | Función |
|---|---|
| admin | Acceso total |
| vendes | Gestión comercial |
| administracio | Gestión administrativa |
| treballador | Acceso limitado |

Archivo SQL utilizado:

```bash
files/sql/roles.sql
```

📸 Captura:

<img width="518" height="262" alt="imatge" src="https://github.com/user-attachments/assets/7113a35b-9a33-4719-9dd6-8527eed47d96" />

<img width="753" height="787" alt="imatge" src="https://github.com/user-attachments/assets/50528f9a-1559-4c4b-89d6-d2efe7449905" />

---

# 12. Usuario web con permisos mínimos

Usuario creado:

```text
web_innovate
```

Permisos:
- SELECT
- INSERT

Restricción:
- acceso únicamente desde `10.0.1.0/24`

Consulta utilizada:

```sql
SHOW GRANTS FOR 'web_innovate'@'10.0.1.%';
```

📸 Captura:

<img width="1066" height="208" alt="imatge" src="https://github.com/user-attachments/assets/f725379f-2e2f-4050-93bd-a8e39771d1f9" />

---

# 13. Script automatizado de usuarios

Script Python utilizado:

```bash
files/scripts/crear_usuaris.py
```

Este script:
- crea usuarios,
- asigna roles,
- genera automáticamente un fichero `.sql`.

Archivo generado:

```bash
usuarios_generados.sql
```

📸 Captura:

<img width="651" height="547" alt="imatge" src="https://github.com/user-attachments/assets/d746e903-fd7b-4a35-ace7-ee9bad091a8c" />

<img width="654" height="375" alt="imatge" src="https://github.com/user-attachments/assets/50e80578-2853-4745-9226-f4c90d620f3a" />

---

# 14. Triggers de auditoría

Triggers implementados:

- control de usuarios bloqueados,
- límite diario de llamadas,
- cuota mensual,
- auditoría automática.

Consultas utilizadas:

```sql
SHOW TRIGGERS;
SELECT * FROM avisos;
```

📸 Captura:

- Triggers
  
<img width="1849" height="951" alt="imatge" src="https://github.com/user-attachments/assets/da25f9f4-e3df-4018-874e-990dba73e4b7" />

<img width="959" height="224" alt="imatge" src="https://github.com/user-attachments/assets/bc84d4af-4a02-4b7a-888b-1ffb3d2558c7" />

---

# 15. Events y backups automáticos

Event implementado:

```text
ev_backup_diari
```

Consultas utilizadas:

```sql
SHOW EVENTS;
SELECT * FROM backups_control;
```

📸 Captura:

<img width="1850" height="221" alt="imatge" src="https://github.com/user-attachments/assets/38f98479-2b64-4af1-a5a9-0a951224d92b" />

<img width="932" height="137" alt="imatge" src="https://github.com/user-attachments/assets/375da85e-b340-4c1e-bee6-c970004699e8" />

---

# 16. Seguridad aplicada

Medidas implementadas:

- MariaDB escuchando únicamente en IP privada,
- acceso restringido a `10.0.1.0/24`,
- usuario web con mínimos privilegios,
- triggers de auditoría,
- backups automáticos,
- automatización mediante Ansible.

Archivo modificado:

```bash
/etc/mysql/mariadb.conf.d/50-server.cnf
```

📸 Captura:

<img width="739" height="41" alt="imatge" src="https://github.com/user-attachments/assets/9b77ad0e-2da6-4b34-928c-c4cd83d88f73" />

---

# 17. Validación final

Consultas utilizadas:

```sql
SHOW TABLES;
SHOW TRIGGERS;
SHOW EVENTS;
SELECT * FROM backups_control;
```

📸 Captura:

<img width="331" height="355" alt="imatge" src="https://github.com/user-attachments/assets/ef310c2a-7ef5-4f78-88c0-8f187d8c9d41" />

-Triggers

<img width="1848" height="932" alt="imatge" src="https://github.com/user-attachments/assets/fc0e2798-dd90-4030-89d3-fbc4a470f010" />

<img width="1852" height="368" alt="imatge" src="https://github.com/user-attachments/assets/cca97b9b-9d79-4d85-9a10-e5030a8cff5d" />

---

# 18. Plantilla de mediciones de ancho de banda

Para facilitar la integración entre el equipo audiovisual y la base de datos de InnovateTech, se ha definido una plantilla estándar en formato CSV para registrar las pruebas de ancho de banda realizadas sobre los servicios de streaming y videoconferencia.

La plantilla permite almacenar:

- fecha y hora de la medición,
- usuario u operario responsable,
- velocidad de descarga,
- velocidad de subida,
- latencia,
- resultado de la prueba,
- observaciones adicionales.

Formato utilizado:

csv:

data_mesura,usuari,download_mbps,upload_mbps,latencia_ms,resultat,observacions
2026-05-21 10:30:00,operari_audiovisual,850,420,2,acceptable,Streaming 1080p estable

---

# 19. Conclusión

La base de datos de InnovateTech ha sido implementada utilizando MariaDB y automatizada mediante Ansible.

La solución cumple los requisitos de:
- gestión de usuarios,
- roles,
- auditoría,
- seguridad,
- automatización,
- backups,
- monitorización básica.
