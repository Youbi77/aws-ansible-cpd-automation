


# Samba AD — Directorio Activo de InnovateTech
## Índice

1. [Objetivo del servicio](#1-objetivo-del-servicio)
2. [Datos del dominio](#2-datos-del-dominio)
3. [Preparación del servidor Samba AD](#3-preparación-del-servidor-samba-ad)
4. [Instalación de paquetes necesarios](#4-instalación-de-paquetes-necesarios)
5. [Provisionado del dominio](#5-provisionado-del-dominio)
6. [Configuración de DNS y Kerberos](#6-configuración-de-dns-y-kerberos)
7. [Creación de usuarios del dominio](#7-creación-de-usuarios-del-dominio)
8. [Creación de grupos del dominio](#8-creación-de-grupos-del-dominio)
9. [Usuario administrativo admin.itb](#9-usuario-administrativo-adminitb)
10. [Comprobaciones realizadas](#10-comprobaciones-realizadas)
11. [Integración con el servidor web-sftp](#11-integración-con-el-servidor-web-sftp)
12. [Conclusión](#12-conclusión)
    
## 1. Objetivo del servicio
En esta parte del proyecto se configuró un servidor Samba AD para actuar como servicio de Directorio Activo de la empresa ficticia InnovateTech.

El objetivo principal de este servidor es centralizar la gestión de usuarios y grupos del dominio, de forma que otros servicios del proyecto puedan autenticarse contra el mismo sistema de identidad. En nuestro caso, este servicio se utiliza principalmente para:

- Autenticar usuarios en el portal web interno.
- Permitir acceso SFTP con usuarios del dominio.
- Gestionar grupos corporativos como `vendes`, `administracio`, `suport` y `logistica`.
- Centralizar usuarios internos de InnovateTech.

Este servicio es importante porque evita tener usuarios locales independientes en cada máquina y permite una gestión más parecida a un entorno empresarial real.

## 2. Datos del dominio
La configuración del dominio utilizada en el proyecto es la siguiente:

| Elemento | Valor |
|---|---|
| Dominio DNS | `btis.inovate.cat` |
| Realm Kerberos | `BTIS.INOVATE.CAT` |
| NetBIOS | `BTIS` |
| Servidor | `samba-ad.btis.inovate.cat` |
| IP privada | `10.0.141.9` |
| Servicio principal | Samba AD / Directorio Activo |

El dominio se utiliza como base para autenticar usuarios desde otros servicios del proyecto, especialmente desde el servidor `web-sftp`.

## 3. Preparación del servidor Samba AD
Antes de configurar Samba AD, se preparó la máquina que actuaría como controlador de dominio. Esta máquina forma parte de la infraestructura AWS del proyecto y se encuentra en la red privada.
Se configuró el nombre del servidor para identificarlo correctamente dentro del dominio:

<img width="866" height="228" alt="{DE21759E-89AA-4049-A844-64E038C1EB92}" src="https://github.com/user-attachments/assets/0ce62a2f-c3be-40f1-87f7-3c873f415e9b" />

## 4. Instalación de paquetes necesarios
Para poder utilizar la máquina como controlador de dominio se instalaron los paquetes necesarios relacionados con Samba AD, Kerberos, Winbind y herramientas de comprobación de red.

Esta instalación se realizó en la máquina `samba-ad`, ya que es el servidor encargado de centralizar los usuarios y grupos del dominio.
 Paquete | Función |
|---|---|
| `samba` | Servicio principal para implementar Samba AD como controlador de dominio |
| `krb5-user` | Herramientas de Kerberos para autenticación dentro del dominio |
| `winbind` | Integración de usuarios y grupos del dominio en Linux |
| `smbclient` | Cliente SMB para pruebas y comprobaciones |
| `dnsutils` | Herramientas como `host` para comprobar registros DNS |
| `acl` / `attr` | Gestión de permisos y atributos necesarios en entornos Samba |

Los paquetes principales utilizados fueron:
```bash
sudo apt update
sudo apt install -y samba krb5-user winbind smbclient dnsutils acl attr
```
**Evidencia:** captura donde se comprueba que Samba, Kerberos y las herramientas de administración del dominio están instaladas correctamente en el servidor `samba-ad`.


<img width="273" height="95" alt="Comprobación herramientas Samba y Kerberos" src="https://github.com/user-attachments/assets/2050b02e-8e3e-4ba3-aad6-d940fbfaa2e2" />


<img width="699" height="214" alt="Servicio Samba AD activo" src="https://github.com/user-attachments/assets/2191067c-0811-477d-8ba4-d28f99169542" />

## 5. Provisionado del dominio
Una vez instalados los paquetes necesarios, se configuró el servidor `samba-ad` como controlador de dominio de InnovateTech.

El provisionado del dominio permite crear la estructura principal del Directorio Activo, incluyendo el dominio, el realm Kerberos, el nombre NetBIOS y la configuración inicial de usuarios y servicios internos.

Los datos principales utilizados en el dominio fueron:
| Parámetro | Valor |
|---|---|
| Realm Kerberos | `BTIS.INOVATE.CAT` |
| Dominio DNS | `btis.inovate.cat` |
| NetBIOS | `BTIS` |
| Servidor controlador de dominio | `samba-ad.btis.inovate.cat` |
El comando de provisionado utilizado fue:

```bash
sudo samba-tool domain provision \
  --use-rfc2307 \
  --interactive
```
Durante el proceso se indicaron los datos del dominio, como el realm, el dominio DNS, el nombre NetBIOS y la contraseña del usuario administrador del dominio.

Este proceso genera la configuración principal de Samba AD y crea la base interna del dominio.

Después del provisionado, se comprobó la configuración general del dominio con:

```bash
sudo samba-tool domain info 127.0.0.1
```

También se revisó el archivo principal de configuración de Samba:

```bash
sudo testparm
```
El comando `testparm` permite validar que la configuración de Samba no contiene errores de sintaxis.

> **Evidencia:** captura donde se comprueba la información del dominio y la validación de la configuración de Samba.

<img width="413" height="143" alt="{BDCB07D1-B5E5-47EB-ACF3-7C3ED9F75D8B}" src="https://github.com/user-attachments/assets/6904be5b-189d-4d60-aa74-7b48dc4fa36b" />
<img width="518" height="663" alt="{ECC328F6-61ED-480A-8615-56BCA6EE820B}" src="https://github.com/user-attachments/assets/44aa2558-9e78-4817-839e-e9545a46ba4e" />

## 6. Configuración de DNS y Kerberos
Samba AD necesita DNS y Kerberos para funcionar correctamente como controlador de dominio.  
El DNS permite que los equipos del dominio localicen servicios internos como LDAP y Kerberos. Kerberos se utiliza para validar la identidad de los usuarios de forma segura dentro del dominio.

En esta instalación, el propio servidor `samba-ad` actúa como servidor DNS interno del dominio `btis.inovate.cat`.

Para comprobar la resolución del controlador de dominio se ejecutó:

```bash
host -t A samba-ad.btis.inovate.cat 127.0.0.1
```

También se comprobaron los registros SRV utilizados por LDAP y Kerberos:

```bash
host -t SRV _ldap._tcp.btis.inovate.cat 127.0.0.1
host -t SRV _kerberos._udp.btis.inovate.cat 127.0.0.1
```

Estos registros son importantes porque permiten que los clientes del dominio encuentren automáticamente los servicios de autenticación.

Para validar Kerberos se obtuvo un ticket con el usuario administrador del dominio:

```bash
kinit administrator@BTIS.INOVATE.CAT
klist
```

El comando `kinit` solicita un ticket Kerberos para el usuario indicado, y `klist` permite comprobar que el ticket se ha generado correctamente.

**Evidencia:** comprobación de registros DNS del dominio y ticket Kerberos activo.
<img width="598" height="322" alt="{CD971244-28AB-4BAA-B6CE-63932DE6C054}" src="https://github.com/user-attachments/assets/40408e0f-4304-4846-9cd1-03df74847a67" />
<img width="577" height="177" alt="{9E8F716B-7800-422A-8899-135B33438C5E}" src="https://github.com/user-attachments/assets/647a5acd-b36e-4e35-b72a-69a1a41f37b2" />

## 7. Creación de usuarios del dominio
Después de configurar el dominio Samba AD, se crearon los usuarios corporativos que representan a los empleados de InnovateTech.

Estos usuarios se utilizan en diferentes partes del proyecto:

- Inicio de sesión en el portal web interno.
- Acceso SFTP con usuarios del dominio.
- Asignación de permisos según el área de la empresa.
- Pruebas de autenticación contra Samba AD.

Los usuarios principales creados fueron:

| Usuario | Función dentro del proyecto |
|---|---|
| `joan.garcia` | Usuario del área de ventas |
| `maria.lopez` | Usuario del área de administración |
| `pere.martinez` | Usuario del área de soporte |
| `anna.puig` | Usuario del área de logística |
| `admin.itb` | Usuario administrador del portal |

El comando utilizado para crear usuarios en Samba AD fue:

```bash
sudo samba-tool user create nombre.usuario 'ContraseñaSegura' \
  --given-name=Nombre \
  --surname=Apellido \
  --mail-address=nombre.usuario@btis.inovate.cat
```

Por ejemplo, para crear el usuario administrador del portal se utilizó una estructura como esta:

```bash
sudo samba-tool user create admin.itb 'Admin@ITB2026' \
  --given-name=Admin \
  --surname=ITB \
  --mail-address=admin.itb@btis.inovate.cat
```

Una vez creados los usuarios, se comprobó que existían dentro del dominio con:

```bash
sudo samba-tool user list | grep -E "joan.garcia|maria.lopez|pere.martinez|anna.puig|admin.itb"
```

También se revisó la información de usuarios concretos con:

```bash
sudo samba-tool user show joan.garcia
sudo samba-tool user show admin.itb
```

Estas comprobaciones permiten validar que los usuarios están registrados correctamente en Samba AD y pueden ser utilizados por otros servicios del proyecto.

**Evidencia:** captura donde se comprueba la existencia de los usuarios principales del dominio.
<img width="833" height="98" alt="{2A62F4C7-ECCB-4EE4-95AF-F688C5068579}" src="https://github.com/user-attachments/assets/b1802193-a350-4f9b-a806-b6ed05d888df" />
<img width="565" height="563" alt="{A7235AE5-4DDC-473D-8EDB-7912A3E75C33}" src="https://github.com/user-attachments/assets/ff4c53a8-6eec-4aa0-95ef-dfb4b891353e" />
<img width="558" height="566" alt="{349DA2B6-80F3-4D89-9DF5-F557AACE96B9}" src="https://github.com/user-attachments/assets/6a378f89-fc83-4a00-ac9c-6f9de6bd5605" />

## 8. Creación de grupos del dominio

Además de los usuarios, se crearon grupos dentro de Samba AD para representar las diferentes áreas de trabajo de InnovateTech.

El uso de grupos permite organizar los usuarios según su departamento o función dentro de la empresa. Esta separación se utiliza posteriormente en el portal web para mostrar información diferente según el perfil del usuario.

Los grupos principales creados fueron:

| Grupo | Función |
|---|---|
| `vendes` | Grupo del área de ventas |
| `administracio` | Grupo del área de administración |
| `suport` | Grupo del área de soporte técnico |
| `logistica` | Grupo del área de logística |
| `portal_admins` | Grupo de administradores del portal interno |

El comando utilizado para crear grupos en Samba AD fue:

```bash
sudo samba-tool group add nombre_grupo
```

Por ejemplo, para crear el grupo de administradores del portal se utilizó:

```bash
sudo samba-tool group add portal_admins
```

Después se añadieron usuarios a sus grupos correspondientes con:

```bash
sudo samba-tool group addmembers nombre_grupo nombre.usuario
```

Por ejemplo, para añadir el usuario administrador al grupo `portal_admins`:

```bash
sudo samba-tool group addmembers portal_admins admin.itb
```

Para comprobar que los grupos existían correctamente se ejecutó:

```bash
sudo samba-tool group list | grep -E "vendes|administracio|suport|logistica|portal_admins"
```

También se comprobaron los miembros de cada grupo:

```bash
sudo samba-tool group listmembers vendes
sudo samba-tool group listmembers administracio
sudo samba-tool group listmembers suport
sudo samba-tool group listmembers logistica
sudo samba-tool group listmembers portal_admins
```

La comprobación del grupo `portal_admins` mostró que el usuario `admin.itb` pertenece correctamente a este grupo.

**Evidencia:** captura donde se comprueba la existencia de los grupos del dominio y los usuarios asignados a cada grupo.
<img width="785" height="271" alt="{C7ABC031-AD9A-4338-959D-D6D7279800B5}" src="https://github.com/user-attachments/assets/d6d08110-96cb-4e50-bfa7-3407626d07c0" />

En la captura se comprueba que los grupos principales del dominio existen en Samba AD y que el usuario `admin.itb` pertenece al grupo `portal_admins`.
## 9. Usuario administrativo admin.itb
Los usuarios y grupos creados en Samba AD se utilizan posteriormente en el portal web interno de InnovateTech.

El objetivo de esta relación es que cada usuario del dominio tenga un perfil asociado dentro del portal. De esta forma, al iniciar sesión con sus credenciales corporativas, la aplicación web puede mostrar información diferente según el usuario autenticado.

La relación utilizada fue la siguiente:

| Usuario | Grupo Samba AD | Perfil en el portal | Información visible |
|---|---|---|---|
| `joan.garcia` | `vendes` | Área de ventas | Clientes, productos, pedidos y vídeos |
| `maria.lopez` | `administracio` | Área de administración | Empleados, nóminas, departamentos y vídeos |
| `pere.martinez` | `suport` | Área de soporte | Avisos, backups, mediciones de ancho de banda y vídeos |
| `anna.puig` | `logistica` | Área de logística | Productos, pedidos, cistell y vídeos |
| `admin.itb` | `portal_admins` | Administración global | Acceso completo a todas las tablas |

Esta configuración permite simular un entorno empresarial real, donde los usuarios no acceden todos a la misma información, sino que cada perfil consulta únicamente los datos relacionados con su función dentro de la empresa.

El flujo de funcionamiento es el siguiente:

```text
Usuario del dominio
        ↓
Login en el portal web
        ↓
Validación contra Samba AD mediante LDAP/TLS
        ↓
Creación de sesión PHP
        ↓
Asignación de perfil dentro del portal
        ↓
Visualización de tablas permitidas desde MariaDB
```
## 10. Comprobaciones realizadas
<img width="1201" height="291" alt="{BE8C3A3C-267A-4822-BB8E-B6736011C831}" src="https://github.com/user-attachments/assets/993846b3-f52e-4283-9df3-8e16308e44ec" />

En esta captura se comprueba que el usuario `joan.garcia` inicia sesión con credenciales del dominio y accede únicamente a la información asociada al perfil de ventas.

<img width="1202" height="296" alt="{1A45F620-93F7-4E52-8E06-8A55C26604A4}" src="https://github.com/user-attachments/assets/9bc3de0a-6c67-4ce3-943c-ab8bf92b96f0" />

En esta captura se observa que el usuario `admin.itb` pertenece al grupo `portal_admins` y dispone de acceso completo a las secciones del portal.

## 11. Integración con el servidor web-sftp
Una vez configurado el dominio Samba AD, se integró el servidor `web-sftp` con el dominio `BTIS.INOVATE.CAT`.

Esta integración permite que el servidor web y el servicio SFTP puedan reconocer usuarios del dominio, en lugar de depender únicamente de usuarios locales de Linux.

El servidor `web-sftp` utiliza esta integración principalmente para:

- Validar usuarios del dominio.
- Permitir el acceso SFTP con cuentas corporativas.
- Comprobar usuarios y grupos mediante Winbind.
- Integrar el portal web con el sistema centralizado de identidad.

Para comprobar que el servidor `web-sftp` está unido correctamente al dominio se ejecutó:

```bash
sudo net ads testjoin
```

El resultado esperado fue:

```text
Join is OK
```

También se comprobó que desde `web-sftp` se podían listar usuarios del dominio:

```bash
wbinfo -u | grep -E "joan.garcia|maria.lopez|pere.martinez|anna.puig|admin.itb"
```

Además, se verificó que Linux reconocía usuarios concretos del dominio mediante el comando `id`:

```bash
id 'BTIS\joan.garcia'
id 'BTIS\admin.itb'
```

Estas comprobaciones demuestran que `web-sftp` puede resolver correctamente usuarios de Samba AD y utilizarlos en servicios como SFTP y el portal interno.

**Evidencia:** comprobación de unión al dominio y resolución de usuarios AD desde `web-sftp`.
<img width="898" height="153" alt="{9E77BE21-36E7-4C57-A7C3-DF780D818CDE}" src="https://github.com/user-attachments/assets/15aa9986-a3d1-4344-9e4d-3ac461bf8ee6" />

En la evidencia se comprueba que el servidor `web-sftp` está unido correctamente al dominio y que reconoce usuarios de Samba AD como `joan.garcia` y `admin.itb`.

## 12. Conclusión
El servicio Samba AD quedó configurado como sistema central de identidad para InnovateTech.

Gracias a esta configuración se consiguió centralizar la gestión de usuarios y grupos del dominio, permitiendo que otros servicios del proyecto utilicen las mismas credenciales corporativas.

En concreto, Samba AD se integró con:

- El portal web interno, para autenticar usuarios mediante LDAP/TLS.
- El servicio SFTP, para permitir acceso con usuarios del dominio.
- El servidor `web-sftp`, que reconoce usuarios y grupos mediante Winbind.
- La organización por perfiles del portal, donde cada usuario visualiza información según su área.

Esta configuración cumple con el requisito de disponer de un servicio de directorio activo dentro de la infraestructura del proyecto y permite una gestión de usuarios más segura, centralizada y realista.
