# Web-SFTP — Portal interno, NGINX y transferencia segura
## Índice

1. [Objetivo del servidor web-sftp](#1-objetivo-del-servidor-web-sftp)
2. [Datos del servidor](#2-datos-del-servidor)
3. [Usuario de administración y acceso SSH](#3-usuario-de-administración-y-acceso-ssh)
4. [Preparación del servidor](#4-preparación-del-servidor)
5. [Instalación de paquetes necesarios](#5-instalación-de-paquetes-necesarios)
6. [Configuración de DNS y unión al dominio Samba AD](#6-configuración-de-dns-y-unión-al-dominio-samba-ad)
7. [Instalación y configuración de NGINX](#7-instalación-y-configuración-de-nginx)
8. [Configuración HTTPS y redirección HTTP a HTTPS](#8-configuración-https-y-redirección-http-a-https)
9. [Certificado SSL con Subject Alternative Name](#9-certificado-ssl-con-subject-alternative-name)
10. [Portal interno InnovateTech](#10-portal-interno-innovatetech)
11. [Autenticación web con Samba AD mediante LDAP/TLS](#11-autenticación-web-con-samba-ad-mediante-ldaptls)
12. [Conexión del portal con MariaDB](#12-conexión-del-portal-con-mariadb)
13. [Control de acceso por usuario en el portal](#13-control-de-acceso-por-usuario-en-el-portal)
14. [Descarga CSV/SQL desde el portal](#14-descarga-csvsql-desde-el-portal)
15. [Configuración del servicio SFTP](#15-configuración-del-servicio-sftp)
16. [Estructura de carpetas SFTP](#16-estructura-de-carpetas-sftp)
17. [Integración SFTP con MariaDB](#17-integración-sftp-con-mariadb)
18. [Automatización con cron](#18-automatización-con-cron)
19. [Integración con servicios multimedia](#19-integración-con-servicios-multimedia)
20. [Conclusión](#22-conclusión)

    
## 1. Objetivo del servidor web-sftp
En esta parte del proyecto se configuró el servidor `web-sftp`, encargado de ofrecer el portal web interno de InnovateTech y el servicio de transferencia segura de ficheros mediante SFTP.

Este servidor cumple dos funciones principales dentro de la infraestructura:

- Publicar el portal web interno mediante NGINX y PHP.
- Permitir acceso SFTP utilizando usuarios del dominio Samba AD.

Además, el portal web se conecta con la base de datos MariaDB del proyecto para mostrar información real de la empresa, como empleados, productos, vídeos, llamadas, mediciones de ancho de banda y otros datos gestionados por el sistema.

El objetivo de esta máquina es actuar como punto central de acceso para los usuarios internos. Desde el portal, los usuarios pueden iniciar sesión con sus credenciales corporativas, consultar información según su perfil y descargar datos en formato CSV o SQL.

El servidor también integra accesos hacia los servicios multimedia del proyecto:

- Radio / Icecast.
- Jitsi Meet.
- Jellyfin.

De esta forma, el servidor `web-sftp` conecta la parte web, la autenticación centralizada, la base de datos y los servicios multimedia en un único portal interno.

## 2. Datos del servidor
La máquina utilizada para esta parte del proyecto fue `web-sftp`.

| Elemento | Valor |
|---|---|
| Nombre del servidor | `web-sftp` |
| FQDN | `web-sftp.btis.inovate.cat` |
| IP pública | `52.1.67.249` |
| IP privada | `10.0.5.140` |
| Usuario de administración | `adminitb` |
| Servicios principales | NGINX, PHP-FPM, SFTP, integración con Samba AD |
| Dominio | `BTIS.INOVATE.CAT` |
| Servidor Samba AD | `samba-ad.btis.inovate.cat` |
| Base de datos | `innovatetech` en MariaDB |

Para comprobar el nombre del servidor, el usuario activo y la configuración de red se utilizaron los siguientes comandos:

```bash
hostname -f
whoami
ip -4 a
```
<img width="852" height="222" alt="{6C5918A6-AF23-4FCE-98E5-CD6FD851F5CB}" src="https://github.com/user-attachments/assets/7c9f786d-df3b-42c7-82f5-de43c99674d4" />

## 3. Usuario de administración y acceso SSH
Para administrar el servidor `web-sftp` se utilizó el usuario específico `adminitb`, evitando trabajar directamente con el usuario por defecto de la instancia.

Esto cumple con el requisito del proyecto de administrar las máquinas con un usuario propio y acceder mediante clave pública/privada.

El usuario `adminitb` se configuró con permisos administrativos mediante `sudo`, permitiendo realizar tareas de mantenimiento, instalación y configuración del servidor.

La conexión al servidor se realiza mediante SSH:

```bash
ssh -i Taylor.pem adminitb@52.1.67.249
```

Una vez dentro del servidor, se comprobó el usuario activo y el nombre de la máquina:

```bash
whoami
hostname -f
```

También se verificaron los permisos administrativos del usuario:

```bash
sudo -l
```

Para comprobar la configuración de permisos de `sudo`, se revisó el archivo correspondiente:

```bash
sudo cat /etc/sudoers.d/adminitb
```

Además, se comprobó la carpeta `.ssh` del usuario, donde se almacenan las claves autorizadas para el acceso remoto:

```bash
ls -la ~/.ssh
```

Con estas comprobaciones se valida que el servidor se administra mediante el usuario `adminitb`, con acceso SSH por clave y permisos administrativos.

**Evidencia:** captura donde se ve el usuario activo `adminitb`.
<img width="583" height="146" alt="{7771A40B-3E19-4D59-9310-EC58FCFAA163}" src="https://github.com/user-attachments/assets/ce003a10-afd4-41ea-bc23-884c4416d45e" />

## 4. Preparación del servidor
Antes de instalar y configurar los servicios principales, se preparó la máquina `web-sftp` para asegurar que tenía conectividad, nombre correcto, sistema actualizado y resolución de red funcional.

Esta preparación es importante porque el servidor `web-sftp` debe comunicarse con otros servicios internos del proyecto:

- `samba-ad`, para autenticar usuarios del dominio.
- `mariadb`, para consultar y sincronizar datos.
- Servidor multimedia, para redirigir accesos a radio, Jitsi y Jellyfin.

Primero se comprobó el nombre del servidor y la configuración de red:

```bash
hostname -f
ip -4 a
```

Después se verificó la conectividad con los servidores internos principales:

```bash
ping -c 3 10.0.141.9
ping -c 3 10.0.142.205
```

Donde:

| IP | Servicio |
|---|---|
| `10.0.141.9` | Servidor Samba AD |
| `10.0.142.205` | Servidor MariaDB |

También se actualizó la lista de paquetes del sistema:

```bash
sudo apt update
```

Esta preparación permitió dejar el servidor listo para instalar NGINX, PHP, herramientas de integración con Samba AD y el servicio SFTP.

**Evidencia:** captura donde se ve la conectividad hacia Samba AD y MariaDB.
<img width="578" height="321" alt="{0044128D-548C-44D9-AB7E-13D01DEDFF04}" src="https://github.com/user-attachments/assets/cf2ff7cf-af5c-4517-b8e4-cbf9f2d5756f" />

## 5. Instalación de paquetes necesarios
Para que el servidor `web-sftp` pudiera ofrecer el portal web interno, integrarse con Samba AD y permitir acceso SFTP con usuarios del dominio, se instalaron varios paquetes del sistema.

Los paquetes principales utilizados fueron:

| Paquete | Función |
|---|---|
| `nginx` | Servidor web utilizado para publicar el portal interno |
| `php8.3-fpm` | Procesamiento de páginas PHP desde NGINX |
| `php8.3-mysql` | Conexión del portal PHP con MariaDB mediante PDO |
| `php8.3-ldap` | Autenticación del portal contra Samba AD mediante LDAP/TLS |
| `mariadb-client` | Cliente para realizar pruebas de conexión con MariaDB |
| `samba-common-bin` | Herramientas para integración con dominio Samba |
| `winbind` | Resolución de usuarios y grupos del dominio en Linux |
| `libnss-winbind` | Integración de usuarios del dominio con NSS |
| `libpam-winbind` | Integración de autenticación del dominio con PAM |
| `krb5-user` | Herramientas Kerberos para autenticación del dominio |
| `openssh-server` | Servicio SSH/SFTP |
| `curl` | Pruebas HTTP/HTTPS y comprobaciones de servicios |
| `openssl` | Generación y comprobación de certificados SSL |

El comando utilizado para instalar los paquetes principales fue:

```bash
sudo apt update
sudo apt install -y nginx php8.3-fpm php8.3-mysql php8.3-ldap mariadb-client samba-common-bin winbind libnss-winbind libpam-winbind krb5-user openssh-server curl openssl
```

Después de la instalación, se comprobaron las versiones y servicios principales:

```bash
nginx -v
php -v
php -m | grep -E "PDO|pdo_mysql|ldap"
which wbinfo
which net
which sshd
```

También se comprobó el estado de los servicios principales:

```bash
sudo systemctl status nginx --no-pager
sudo systemctl status php8.3-fpm --no-pager
sudo systemctl status ssh --no-pager
sudo systemctl status winbind --no-pager
```

Con estos paquetes instalados, el servidor quedó preparado para:

- Publicar el portal web con NGINX.
- Ejecutar código PHP mediante PHP-FPM.
- Conectar con MariaDB desde PHP.
- Autenticar usuarios contra Samba AD mediante LDAP/TLS.
- Reconocer usuarios del dominio mediante Winbind.
- Permitir acceso SSH/SFTP.

**Evidencia:** captura donde se comprueban las versiones de NGINX/PHP, módulos PHP necesarios y servicios principales activos.

<img width="614" height="317" alt="{78874DD4-A036-4943-BA63-582D0A387167}" src="https://github.com/user-attachments/assets/86ec0ba9-4255-4af5-af54-9c40c8153b9a" />
<img width="777" height="73" alt="{2152506F-3186-4B49-BD89-F167E1C11FE3}" src="https://github.com/user-attachments/assets/d7d3ca1f-a664-4e87-8caf-d88fd2554bfc" />
<img width="826" height="74" alt="{647EA3C0-F688-4F48-8786-547621794E0C}" src="https://github.com/user-attachments/assets/3f834752-846c-46bf-bb0a-ee936703f294" />
<img width="797" height="74" alt="{4EEB6E92-AE48-4C7B-9F72-64DABEF0C293}" src="https://github.com/user-attachments/assets/878db0c5-1796-4bbc-825b-0ce0f9504f72" />

## 6. Configuración de DNS y unión al dominio Samba AD
Para que el servidor `web-sftp` pudiera autenticarse contra Samba AD y reconocer usuarios del dominio, fue necesario integrarlo dentro del dominio `BTIS.INOVATE.CAT`.

Esta integración permite que el servidor `web-sftp` pueda utilizar usuarios centralizados del dominio para servicios como:

- Inicio de sesión en el portal web.
- Acceso SFTP con usuarios corporativos.
- Resolución de usuarios y grupos mediante Winbind.

Antes de unir el servidor al dominio, se comprobó que `web-sftp` pudiera comunicarse con el servidor Samba AD:

```bash
ping -c 3 10.0.141.9
```

También se comprobó la resolución del servidor Samba AD por nombre:

```bash
getent hosts samba-ad.btis.inovate.cat
```

El servidor se integró con el dominio utilizando herramientas de Samba y Kerberos. Una vez unido al dominio, se comprobó el estado de la unión con:

```bash
sudo net ads testjoin
```

El resultado esperado fue:

```text
Join is OK
```

Después se comprobó que `web-sftp` podía listar usuarios y grupos del dominio:

```bash
wbinfo -u | grep -E "joan.garcia|maria.lopez|pere.martinez|anna.puig|admin.itb"
wbinfo -g | grep -E "vendes|administracio|suport|logistica|portal_admins"
```

También se validó que Linux reconocía usuarios concretos del dominio:

```bash
id 'BTIS\joan.garcia'
id 'BTIS\admin.itb'
```

Estas comprobaciones demuestran que el servidor `web-sftp` está correctamente unido al dominio Samba AD y puede utilizar usuarios corporativos en el portal web y en el servicio SFTP.

**Evidencia:** captura donde se comprueba la unión al dominio y la resolución de usuarios del dominio desde `web-sftp`.

<img width="920" height="349" alt="{3F38E3DB-1861-41CD-BC98-7A75BF581184}" src="https://github.com/user-attachments/assets/5842d308-ec3b-4997-b5c6-1fe1f6530c01" />

## 7. Instalación y configuración de NGINX
Para publicar el portal interno de InnovateTech se utilizó NGINX como servidor web.

NGINX se encarga de recibir las peticiones HTTP/HTTPS de los usuarios y servir la aplicación PHP ubicada en el directorio:

```text
/var/www/innovatetech
```

La configuración principal del sitio se guardó en:

```text
/etc/nginx/sites-available/innovatetech
```

Y se habilitó mediante un enlace simbólico en:

```text
/etc/nginx/sites-enabled/
```

El paquete principal instalado fue:

```bash
sudo apt install -y nginx
```

Después se creó el directorio del portal:

```bash
sudo mkdir -p /var/www/innovatetech
```

La configuración del sitio de NGINX incluye:

- Escucha en el puerto `80`.
- Escucha en el puerto `443` con SSL.
- Directorio raíz `/var/www/innovatetech`.
- Uso de `index.php` como página principal.
- Procesamiento PHP mediante PHP-FPM.
- Cabeceras básicas de seguridad.
- Reverse proxy o redirección hacia servicios multimedia.

El bloque principal del sitio se revisó con:

```bash
sudo cat /etc/nginx/sites-available/innovatetech
```

Para comprobar que la sintaxis de NGINX era correcta, se utilizó:

```bash
sudo nginx -t
```

Después de aplicar cambios, se reinició el servicio:

```bash
sudo systemctl restart nginx
```

También se comprobó el estado del servicio:

```bash
sudo systemctl status nginx --no-pager
```

Y los puertos abiertos:

```bash
sudo ss -tulpn | grep -E ':80|:443'
```

Con esta configuración, el servidor `web-sftp` quedó preparado para publicar el portal interno de InnovateTech mediante NGINX.

**Evidencia:** captura donde se comprueba la configuración de NGINX, , el servicio activo y los puertos `80` y `443`.

<img width="1206" height="440" alt="{CCDBB08A-85F3-4458-A3B4-540BB655503B}" src="https://github.com/user-attachments/assets/172d167a-cf35-448e-b973-5a838dc03146" />

## 8. Configuración HTTPS y redirección HTTP a HTTPS
Para mejorar la seguridad del portal interno, se configuró NGINX para servir la web mediante HTTPS.

La configuración utiliza dos bloques principales:

- Un bloque en el puerto `80`, encargado de redirigir todo el tráfico HTTP hacia HTTPS.
- Un bloque en el puerto `443`, encargado de servir el portal web con certificado SSL.

La redirección HTTP a HTTPS se configuró en NGINX con la siguiente estructura:

```nginx
server {
    listen 80;
    server_name _;

    return 301 https://$host$request_uri;
}
```

Con esta configuración, cualquier acceso por HTTP se redirige automáticamente a la misma URL utilizando HTTPS.

El bloque HTTPS escucha en el puerto `443`:

```nginx
server {
    listen 443 ssl;
    server_name _;

    ssl_certificate /etc/ssl/certs/innovatetech.crt;
    ssl_certificate_key /etc/ssl/private/innovatetech.key;

    root /var/www/innovatetech;
    index index.php;
}
```

Después de modificar la configuración, se validó la sintaxis de NGINX:

```bash
sudo nginx -t
```

También se comprobó la redirección HTTP a HTTPS con:

```bash
curl -I http://localhost/login.php
```

El resultado esperado fue una respuesta `301 Moved Permanently` hacia HTTPS:

```text
HTTP/1.1 301 Moved Permanently
Location: https://localhost/login.php
```

Finalmente, se comprobó que el portal respondía correctamente por HTTPS:

```bash
curl -k -I https://localhost/login.php
curl -k -I https://52.1.67.249/login.php
```

El resultado esperado fue:

```text
HTTP/1.1 200 OK
```

Con estas pruebas se valida que el servidor redirige correctamente HTTP hacia HTTPS y que el portal interno funciona mediante conexión cifrada.

**Evidencia:** captura donde se ve la redirección HTTP a HTTPS y la respuesta correcta del portal mediante HTTPS.

<img width="571" height="659" alt="{AE578978-675B-4B0C-9F41-522C877A2D03}" src="https://github.com/user-attachments/assets/e4b03814-3410-4975-ba77-c5bd1b9d9b53" />
<img width="551" height="87" alt="{D6E3C6BC-5DFD-4F5A-B6B5-82C0163184A6}" src="https://github.com/user-attachments/assets/47c5d2f3-ce4a-44bd-a46c-ec7bf3b5d143" />

Además, se comprobó visualmente desde el navegador que el portal carga mediante `https://52.1.67.249/login.php`. El navegador muestra el aviso de “No es seguro” porque el certificado utilizado es autofirmado, pero la conexión se realiza mediante HTTPS.

## 9. Certificado SSL con Subject Alternative Name
Para servir el portal mediante HTTPS se generó un certificado SSL autofirmado para el servidor `web-sftp`.

Inicialmente, el certificado funcionaba para cifrar la conexión, pero el navegador mostraba un aviso de seguridad porque el certificado no incluía la extensión `Subject Alternative Name` y no estaba firmado por una autoridad certificadora pública.

Para corregir esta parte, se generó un nuevo certificado incluyendo los nombres alternativos necesarios:

| Tipo | Valor |
|---|---|
| IP Address | `52.1.67.249` |
| DNS | `web-sftp.btis.inovate.cat` |
| DNS | `localhost` |

Primero se creó un archivo de configuración para OpenSSL:

```bash
sudo nano /tmp/innovatetech-openssl.cnf
```

En este archivo se definieron los datos del certificado y los valores de `Subject Alternative Name`.

Después se generó el certificado y la clave privada:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/innovatetech.key \
  -out /etc/ssl/certs/innovatetech.crt \
  -config /tmp/innovatetech-openssl.cnf
```

Posteriormente se ajustaron los permisos de los ficheros SSL:

```bash
sudo chmod 600 /etc/ssl/private/innovatetech.key
sudo chmod 644 /etc/ssl/certs/innovatetech.crt
```

Para comprobar el contenido del certificado se ejecutó:

```bash
openssl x509 -in /etc/ssl/certs/innovatetech.crt -noout -subject -issuer -dates -ext subjectAltName
```

También se comprobó el certificado servido realmente por NGINX:

```bash
echo | openssl s_client -connect 52.1.67.249:443 -servername 52.1.67.249 2>/dev/null | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

En la salida se comprobó que el certificado incluía:

```text
IP Address:52.1.67.249
DNS:web-sftp.btis.inovate.cat
DNS:localhost
```

Aunque el navegador sigue mostrando un aviso porque el certificado es autofirmado, la conexión HTTPS funciona correctamente y el certificado contiene los valores SAN necesarios. Para eliminar completamente el aviso del navegador sería necesario utilizar un dominio público y un certificado emitido por una autoridad certificadora reconocida, como Let's Encrypt.

**Evidencia:** captura donde se comprueba que el certificado SSL contiene el `Subject Alternative Name` con la IP pública `52.1.67.249`, el DNS `web-sftp.btis.inovate.cat` y `localhost`.

<img width="1105" height="134" alt="{8875B6AE-47DA-4F22-945A-DE0E62B0130B}" src="https://github.com/user-attachments/assets/0c9f682a-e9f7-4522-a488-4699fb74405c" />


## 10. Portal interno InnovateTech
En el servidor `web-sftp` se desarrolló y desplegó el portal interno de InnovateTech.

Este portal no es una página estática, sino una aplicación web en PHP que permite a los usuarios internos iniciar sesión con sus credenciales del dominio, consultar información almacenada en MariaDB y acceder a los servicios integrados del proyecto.

El portal se encuentra en el directorio:

```text
/var/www/innovatetech
```

Los archivos principales de la aplicación son:

| Archivo | Función |
|---|---|
| `login.php` | Formulario de inicio de sesión y autenticación contra Samba AD |
| `index.php` | Dashboard principal del portal interno |
| `logout.php` | Cierre de sesión del usuario |
| `download.php` | Descarga de tablas permitidas en formato CSV o SQL |

El portal ofrece las siguientes funcionalidades:

- Inicio de sesión con usuarios del dominio Samba AD.
- Visualización de datos reales procedentes de MariaDB.
- Control de acceso por usuario.
- Descarga de información en formato CSV o SQL.
- Acceso a servicios multimedia integrados.
- Interfaz tipo dashboard corporativo.
- Diseño responsive para adaptarse a diferentes tamaños de pantalla.

La estructura principal de la web se comprobó con:

```bash
ls -la /var/www/innovatetech
```

También se validó que los archivos PHP no tuvieran errores de sintaxis:

```bash
php -l /var/www/innovatetech/login.php
php -l /var/www/innovatetech/index.php
php -l /var/www/innovatetech/download.php
php -l /var/www/innovatetech/logout.php
```

El portal se publica mediante NGINX y es accesible desde el navegador mediante la URL:

```text
https://52.1.67.249/login.php
```

Aunque el navegador muestra un aviso porque el certificado es autofirmado, la web se sirve mediante HTTPS y el acceso está cifrado.

**Evidencia:** captura del directorio del portal y validación de sintaxis de los archivos PHP.
<img width="696" height="278" alt="{941F3023-5257-4504-8882-7D3364CF77A0}" src="https://github.com/user-attachments/assets/24771400-80c0-4bce-88d0-e204b2757d18" />

**Evidencia visual:** captura del portal interno cargando correctamente desde el navegador.
<img width="927" height="1176" alt="{578BC624-AD7A-45F5-ABDF-033498426BF9}" src="https://github.com/user-attachments/assets/62ac54df-043c-41fe-912c-c4e00c6f18dd" />

## 11. Autenticación web con Samba AD mediante LDAP/TLS

El portal interno de InnovateTech utiliza Samba AD como sistema centralizado de autenticación.

Cuando un usuario introduce sus credenciales en `login.php`, la aplicación PHP se conecta al servidor Samba AD mediante LDAP/TLS y valida el usuario contra el dominio `BTIS.INOVATE.CAT`.

El flujo de autenticación es el siguiente:

```text
Usuario introduce usuario y contraseña
        ↓
login.php recibe las credenciales
        ↓
PHP conecta con Samba AD mediante LDAP/TLS
        ↓
Samba AD valida las credenciales
        ↓
Si son correctas, PHP crea una sesión
        ↓
El usuario accede al dashboard interno
```

El archivo encargado de esta autenticación es:

```text
/var/www/innovatetech/login.php
```

En este archivo se configura la conexión LDAP hacia el servidor Samba AD y se utiliza el usuario introducido en el formulario para validar el inicio de sesión.

Para comprobar que el archivo PHP no contiene errores de sintaxis se ejecutó:

```bash
php -l /var/www/innovatetech/login.php
```

También se comprobó que el módulo LDAP de PHP estuviera disponible:

```bash
php -m | grep ldap
```

La prueba funcional se realizó accediendo al portal desde el navegador:

```text
https://52.1.67.249/login.php
```

Usuarios probados:

| Usuario | Resultado |
|---|---|
| `joan.garcia` | Login correcto |
| `maria.lopez` | Login correcto |
| `pere.martinez` | Login correcto |
| `anna.puig` | Login correcto |
| `admin.itb` | Login correcto |

El usuario `admin.itb` se utiliza como usuario administrador del portal, mientras que el resto de usuarios acceden a vistas limitadas según su perfil.

Esta integración permite que el portal web no dependa de usuarios locales propios, sino de las cuentas corporativas almacenadas en Samba AD.

**Evidencia:** captura donde se comprueba el módulo LDAP de PHP y la validación del archivo `login.php`.
<img width="1065" height="331" alt="{4146D64F-0691-4C1C-AB1F-3239D6297D95}" src="https://github.com/user-attachments/assets/a41119b1-faa1-41a2-a3f6-8ad701edbbe3" />

**Evidencia visual:** captura del inicio de sesión correcto en el portal con un usuario del dominio.
<img width="1254" height="1093" alt="{B74E0D40-91D3-49A6-88CC-0FC0C1CC88C0}" src="https://github.com/user-attachments/assets/5286374a-daef-4579-af44-3cdf2517895c" />

## 12. Conexión del portal con MariaDB
El portal interno de InnovateTech se conecta con la base de datos MariaDB del proyecto para mostrar información real de la empresa.

La base de datos utilizada es:

```text
innovatetech
```

El servidor MariaDB se encuentra en la red privada de AWS:

```text
10.0.142.205
```

Para que el portal pudiera consultar la información de forma segura, se utilizó un usuario específico de base de datos con permisos mínimos de lectura:

```text
web_readonly
```

Este usuario permite que la aplicación web consulte los datos necesarios sin utilizar el usuario `root` de MariaDB.

La conexión desde PHP se realiza mediante PDO, usando el módulo `pdo_mysql`.

Para comprobar que el servidor `web-sftp` podía conectarse a MariaDB se ejecutó una prueba directa con PHP:

```bash
php -r '$pdo=new PDO("mysql:host=10.0.142.205;dbname=innovatetech","web_readonly","WebRead@2025"); echo "Conexión web_readonly OK\n";'
```

También se realizó una consulta de prueba sobre varias tablas de la base de datos:

```bash
php -r '$pdo=new PDO("mysql:host=10.0.142.205;dbname=innovatetech","web_readonly","WebRead@2025"); foreach(["clients","productes","comandes","empleats","videos","trucades","mesures_amplada_banda"] as $t){echo "\n--- $t ---\n"; foreach($pdo->query("SELECT * FROM `$t` LIMIT 3") as $r){print_r($r);}}'
```

El portal utiliza esta conexión para cargar dinámicamente las tablas que se muestran a cada usuario según su perfil.

Esta integración cumple con la parte del proyecto donde la base de datos debe servir como fuente de datos para una aplicación de gestión alojada en el servidor web.

**Evidencia:** captura donde se comprueba la conexión desde `web-sftp` hacia MariaDB mediante PHP/PDO.

<img width="1206" height="1161" alt="{21216820-9E5B-44FB-A9D0-B29E423C325A}" src="https://github.com/user-attachments/assets/5743cba6-5054-49a9-80ee-cc83cf7ad5e6" />

En la captura se comprueba que el servidor `web-sftp` puede conectarse correctamente a MariaDB usando el usuario `web_readonly` y consultar datos reales de la base de datos `innovatetech`. Esto demuestra que el portal web no utiliza datos estáticos, sino información obtenida directamente desde la base de datos del proyecto.

## 13. Control de acceso por usuario en el portal
El portal interno de InnovateTech aplica un control de acceso por usuario. Esto significa que todos los usuarios no ven la misma información al iniciar sesión.

Después de autenticarse contra Samba AD, el portal identifica el usuario que ha iniciado sesión y le asigna un perfil dentro de la aplicación. Según ese perfil, se muestran unas tablas u otras de MariaDB.

La relación aplicada en el portal fue:

| Usuario | Grupo / perfil | Tablas visibles |
|---|---|---|
| `joan.garcia` | `vendes` | `clients`, `productes`, `comandes`, `videos` |
| `maria.lopez` | `administracio` | `empleats`, `nomines`, `departaments`, `videos` |
| `pere.martinez` | `suport` | `avisos`, `backups_control`, `mesures_amplada_banda`, `videos` |
| `anna.puig` | `logistica` | `productes`, `comandes`, `cistell`, `videos` |
| `admin.itb` | `portal_admins` | acceso completo a todas las tablas |

El archivo principal encargado de mostrar el dashboard y aplicar esta lógica es:

```text
/var/www/innovatetech/index.php
```

En este archivo se define qué tablas puede visualizar cada usuario. De esta forma, el portal evita que un usuario pueda consultar información que no corresponde a su área.

Por ejemplo:

- `joan.garcia`, como usuario del área de ventas, puede ver clientes, productos, pedidos y vídeos.
- `maria.lopez`, como usuaria de administración, puede ver empleados, nóminas, departamentos y vídeos.
- `admin.itb`, como usuario administrador, puede ver todas las tablas disponibles.

Para comprobar que el archivo principal del portal no tenía errores de sintaxis se utilizó:

```bash
php -l /var/www/innovatetech/index.php
```

También se comprobó visualmente iniciando sesión con diferentes usuarios del dominio y verificando que cada uno veía secciones distintas.

Esta capa de control de acceso en la aplicación complementa los roles y permisos SQL definidos en MariaDB, documentados en la parte de base de datos.

**Evidencia:** captura del portal iniciado como `joan.garcia`, mostrando sus secciones correspondientes al área de ventas.

<img width="1259" height="523" alt="{04091988-9129-4630-9C68-1E5403961F70}" src="https://github.com/user-attachments/assets/6bc52e2f-04c9-4253-86ee-de281db0ac0d" />

**Evidencia:** captura del portal iniciado como `admin.itb`, mostrando acceso completo a las secciones del portal.

<img width="1268" height="885" alt="{01EFE876-C863-4B0F-BF4E-213EFAA6868B}" src="https://github.com/user-attachments/assets/060b02ea-fd40-473b-b870-1b3528a97956" />

## 14. Descarga CSV/SQL desde el portal
El portal interno permite descargar información de MariaDB en formato CSV o SQL directamente desde la web.

Esta funcionalidad se implementó mediante el archivo:

```text
/var/www/innovatetech/download.php
```

El objetivo de esta parte es que cada usuario pueda exportar únicamente los datos que tiene permitidos según su perfil, sin acceder directamente a MariaDB.

El funcionamiento es el siguiente:

```text
Usuario autenticado en el portal
        ↓
El portal muestra solo sus tablas permitidas
        ↓
El usuario pulsa Descargar CSV o Descargar SQL
        ↓
download.php comprueba sesión y permisos
        ↓
Si la tabla está permitida, genera el archivo
        ↓
Si la tabla no está permitida, bloquea la descarga
```

El control de acceso se aplica en dos niveles:

| Archivo | Función |
|---|---|
| `index.php` | Muestra únicamente las tablas permitidas para cada usuario |
| `download.php` | Valida que el usuario tenga permiso antes de generar CSV o SQL |

Por ejemplo, el usuario `joan.garcia`, perteneciente al perfil de ventas, visualiza en el portal las tablas:

```text
clients
productes
comandes
videos
```

Dentro de estas tablas aparecen los botones:

```text
Descargar CSV
Descargar SQL
Copiar tabla
```

Por tanto, `joan.garcia` puede descargar información de sus tablas permitidas, como `clients`:

En cambio, tablas como `nomines` no aparecen directamente en su portal, ya que no pertenecen a su perfil. Además, si el usuario intenta acceder manualmente a una URL de descarga de una tabla no permitida, `download.php` bloquea la petición:
El resultado esperado en ese caso es:

```text
No tienes permiso para descargar esta tabla.
```

También se permite la descarga en formato SQL para las tablas autorizadas:

```text
https://52.1.67.249/download.php?tabla=clients&formato=sql
```

Para comprobar que el archivo no contiene errores de sintaxis se ejecutó:

```bash
php -l /var/www/innovatetech/download.php
```

Esta funcionalidad permite exportar información desde el portal de forma controlada y evita que los usuarios puedan descargar datos que no pertenecen a su área.

**Evidencia:** captura del portal iniciado como `joan.garcia`, donde solo se muestran sus tablas permitidas y los botones de descarga CSV/SQL.

<img width="1240" height="591" alt="{EBCBE937-15E9-4E00-86B1-CE10617D695B}" src="https://github.com/user-attachments/assets/b935b5c5-94cc-437f-90f2-cd833d65e45d" />

**Evidencia:** captura de una descarga permitida, por ejemplo `clients.csv`.

<img width="1254" height="527" alt="{8C858824-E74D-4DF7-A00B-FB8A1B71D5BC}" src="https://github.com/user-attachments/assets/f2bae121-9481-4f1d-9538-319db062f684" />

**Evidencia :** captura de ejemplo de joan.garcia no puede ver otros apartados, como por ejemplo `nominas`.

<img width="217" height="525" alt="{0CA44619-ECB8-4195-87B7-B5B0AF643CA6}" src="https://github.com/user-attachments/assets/eeabc48c-8c76-4e06-9fb8-d3803dba7e16" />


## 15. Configuración del servicio SFTP
Además del portal web, el servidor `web-sftp` también ofrece acceso SFTP para usuarios del dominio Samba AD.

El objetivo de esta configuración es que los usuarios corporativos puedan subir y descargar ficheros de forma segura utilizando sus credenciales del dominio, sin tener que crear cuentas locales independientes en Linux.

El servicio SFTP utiliza el servidor OpenSSH instalado en la máquina `web-sftp`.

La configuración principal se encuentra en:

```text
/etc/ssh/sshd_config
```

Para validar que la configuración de SSH no contenía errores de sintaxis se utilizó:

```bash
sudo sshd -t
```

Después de aplicar cambios, se reinició el servicio SSH:

```bash
sudo systemctl restart ssh
```

También se comprobó el estado del servicio:

```bash
sudo systemctl status ssh --no-pager
```

El acceso SFTP se realiza con usuarios del dominio utilizando el formato:

```bash
sftp 'BTIS\usuario'@52.1.67.249
```

Por ejemplo, para acceder con el usuario `joan.garcia`:

```bash
sftp 'BTIS\joan.garcia'@52.1.67.249
```

Una vez autenticado, el usuario accede a un entorno limitado donde puede ver las carpetas:

```text
download
upload
```

La carpeta `download` se utiliza para descargar ficheros CSV exportados desde MariaDB, mientras que la carpeta `upload` se utiliza para subir ficheros CSV que posteriormente pueden ser procesados e importados a la base de datos.

Esta configuración permite cumplir el requisito de disponer de un servicio SFTP autenticado mediante usuarios del Directorio Activo.

**Evidencia:** captura del acceso SFTP con el usuario de dominio `BTIS\joan.garcia`.

<img width="1022" height="292" alt="{C8D0EC7B-9C35-4598-B208-6B90D3E8A0EC}" src="https://github.com/user-attachments/assets/0363a4d4-b3e9-4f92-ab1d-172a016f281f" />

## 16. Estructura de carpetas SFTP
Para organizar el intercambio de ficheros por SFTP, se creó una estructura de carpetas separada para cada usuario del dominio.

La ruta base utilizada fue:

```text
/sftp/usuarios/
```

Dentro de esta ruta, cada usuario dispone de su propio directorio. Como los usuarios pertenecen al dominio Samba AD, el nombre utilizado incluye el prefijo del dominio:

```text
BTIS\usuario
```

Por ejemplo, para el usuario `joan.garcia`, la estructura queda así:

```text
/sftp/usuarios/BTIS\joan.garcia/
├── download
└── upload
```

La función de cada carpeta es:

| Carpeta | Función |
|---|---|
| `download` | Contiene los CSV exportados desde MariaDB para que el usuario pueda descargarlos |
| `upload` | Permite subir CSV para que posteriormente sean procesados e importados a MariaDB |
| `upload/procesados` | Guarda los CSV que ya han sido procesados por el script de sincronización |

Para comprobar la estructura de carpetas se utilizaron los siguientes comandos en el servidor `web-sftp`:

```bash
sudo ls -la /sftp/usuarios
sudo ls -la '/sftp/usuarios/BTIS\joan.garcia'
sudo ls -la '/sftp/usuarios/BTIS\joan.garcia/download'
sudo ls -la '/sftp/usuarios/BTIS\joan.garcia/upload'
```

También se comprobó desde el cliente SFTP que el usuario únicamente veía las carpetas esperadas:

```sftp
pwd
ls
cd download
ls
cd ../upload
ls
```

Esta estructura permite separar los ficheros de cada usuario y organizar el flujo de intercambio con MariaDB.

**Evidencia:** captura donde se ve la estructura de carpetas SFTP del usuario `BTIS\joan.garcia`.

<img width="739" height="601" alt="{FA3DFD1E-C93C-4922-A45E-96856FA1DDB8}" src="https://github.com/user-attachments/assets/4e72eefd-d956-4d52-9b60-f83ad94cdbad" />

**Evidencia:** captura desde el cliente SFTP donde se ve las carpetas `download` y `upload`.

<img width="1030" height="242" alt="{CD1FA11C-6AD7-4B0B-AF0E-F7D314AC51E1}" src="https://github.com/user-attachments/assets/9dfc874e-981e-4d07-8c69-a9415e893d8b" />

## 17. Integración SFTP con MariaDB
Para integrar el servicio SFTP con la base de datos MariaDB, se creó un script de sincronización en PHP.

El objetivo de esta integración es permitir que los usuarios del dominio puedan:

- Descargar información exportada desde MariaDB en formato CSV.
- Subir ficheros CSV mediante SFTP.
- Importar datos desde esos CSV hacia MariaDB.
- Mantener un flujo de intercambio entre el portal, SFTP y la base de datos.

El script utilizado se encuentra en:

```text
/usr/local/bin/sftp_db_sync.php
```

Este script realiza dos tareas principales:

| Acción | Descripción |
|---|---|
| Exportación | Lee tablas de MariaDB y genera ficheros CSV en la carpeta `download` de cada usuario |
| Importación | Lee CSV subidos a la carpeta `upload`, los importa a MariaDB y mueve el archivo a `upload/procesados` |

El flujo de funcionamiento es el siguiente:

```text
MariaDB
   ↓
sftp_db_sync.php exporta tablas a CSV
   ↓
/sftp/usuarios/BTIS\usuario/download
   ↓
El usuario descarga los CSV por SFTP
```

Y para la importación:

```text
El usuario sube un CSV por SFTP
   ↓
/sftp/usuarios/BTIS\usuario/upload
   ↓
sftp_db_sync.php procesa el archivo
   ↓
MariaDB recibe los nuevos datos
   ↓
El CSV procesado se mueve a upload/procesados
```

Para que el script pudiera conectarse a MariaDB se creó el usuario específico:

```text
sftp_sync
```

Este usuario se utiliza únicamente para la sincronización SFTP ↔ MariaDB.

Primero se comprobó la conexión con MariaDB desde `web-sftp`:

```bash
php -r '$pdo=new PDO("mysql:host=10.0.142.205;dbname=innovatetech","sftp_sync","@ITB2025"); echo "Conexión sftp_sync OK\n";'
```

Después se ejecutó manualmente el script de sincronización:

```bash
sudo php /usr/local/bin/sftp_db_sync.php
```

En la salida del script se pueden ver las exportaciones hacia la carpeta `download` de cada usuario y las importaciones de CSV subidos a `upload`.

Ejemplo de salida esperada:

```text
[OK] Conexión con MariaDB correcta
[EXPORT] empleats -> /sftp/usuarios/BTIS\joan.garcia/download/empleats.csv
[EXPORT] videos -> /sftp/usuarios/BTIS\joan.garcia/download/videos.csv
[EXPORT] trucades -> /sftp/usuarios/BTIS\joan.garcia/download/trucades.csv
[EXPORT] mesures_amplada_banda -> /sftp/usuarios/BTIS\joan.garcia/download/mesures_amplada_banda.csv
[IMPORT] 1 registros importados desde /sftp/usuarios/BTIS\joan.garcia/upload/import_mesures_amplada_banda.csv
[OK] Archivo movido a upload/procesados
[FIN] Sincronización finalizada correctamente
```

También se comprobó que los CSV exportados existían dentro de la carpeta `download`:

```bash
ls -la '/sftp/usuarios/BTIS\joan.garcia/download'
```

Y se verificó en MariaDB que los datos importados quedaban registrados correctamente:

```bash
php -r '$pdo=new PDO("mysql:host=10.0.142.205;dbname=innovatetech","sftp_sync","@ITB2025"); foreach($pdo->query("SELECT data_mesura, usuari, download_mbps, upload_mbps, latencia_ms, resultat FROM mesures_amplada_banda ORDER BY data_mesura DESC LIMIT 3") as $r){print_r($r);}'
```

Esta integración permite que SFTP funcione como mecanismo de intercambio de datos con MariaDB, conectando la parte de transferencia de ficheros con la base de datos central del proyecto.

**Evidencia:** captura donde se comprueba la conexión del usuario `sftp_sync` con MariaDB.

<img width="1209" height="61" alt="{D8875BFC-395F-475D-B3FA-CAA731F2BBFE}" src="https://github.com/user-attachments/assets/fcfcd82b-5ba0-4e35-9c60-a55eadafeeea" />

**Evidencia:** captura de la ejecución del script `sftp_db_sync.php` mostrando exportaciones e importaciones.

<img width="931" height="350" alt="{B1107961-A627-4542-A6CC-EA232B376A62}" src="https://github.com/user-attachments/assets/460dd4e7-e1bb-45f7-90b1-d99497edbff9" />

**Evidencia:** captura donde se vean los CSV generados en la carpeta `download`.

<img width="642" height="174" alt="{EFF54467-BD2F-4BB1-8E25-F786017F8415}" src="https://github.com/user-attachments/assets/90f3b20b-5588-4502-9f71-537bbf2ffe10" />

## 18. Automatización con cron
Para automatizar la sincronización entre SFTP y MariaDB se configuró una tarea programada con `cron`.

El objetivo es que el script:

```text
/usr/local/bin/sftp_db_sync.php
```

se ejecute de forma periódica sin intervención manual.

De esta manera, los CSV exportados desde MariaDB se actualizan automáticamente en las carpetas `download` de los usuarios, y los CSV subidos a `upload` pueden procesarse de forma periódica.

La tarea se configuró en el `crontab` del usuario `root`, ya que el script necesita permisos para acceder a las carpetas SFTP y mover archivos procesados.

Para editar el `crontab` de root se utilizó:

```bash
sudo crontab -e
```

La línea añadida fue:

```cron
*/2 * * * * /usr/bin/php /usr/local/bin/sftp_db_sync.php >> /var/log/sftp_sync.log 2>&1
```

Esta línea ejecuta el script cada 2 minutos.

Explicación de la tarea:

| Parte | Significado |
|---|---|
| `*/2 * * * *` | Ejecuta la tarea cada 2 minutos |
| `/usr/bin/php` | Intérprete PHP utilizado para ejecutar el script |
| `/usr/local/bin/sftp_db_sync.php` | Script de sincronización SFTP ↔ MariaDB |
| `>> /var/log/sftp_sync.log` | Guarda la salida en un fichero de log |
| `2>&1` | Redirige también los errores al mismo log |

Para comprobar que la tarea quedó guardada correctamente se ejecutó:

```bash
sudo crontab -l
```

También se puede revisar el log generado por el script con:

```bash
sudo tail -n 30 /var/log/sftp_sync.log
```

Con esta automatización, la integración entre SFTP y MariaDB queda programada y no depende de ejecutar manualmente el script cada vez.

**Evidencia:** captura donde se vea la tarea configurada en `cron`.

<img width="802" height="60" alt="{19D8D04B-3FD8-469A-BF9A-BE58A15DAABF}" src="https://github.com/user-attachments/assets/22436a5d-3c6d-4de1-83a6-f7ce54a1fd89" />

**Evidencia** captura del fichero `/var/log/sftp_sync.log` mostrando ejecuciones del script.

<img width="927" height="545" alt="{E5293211-BA6F-41AD-B0EC-6DD321551767}" src="https://github.com/user-attachments/assets/fde0ebe0-258a-448b-a23b-86257521d8a5" />

## 19. Integración con servicios multimedia
El portal interno de InnovateTech también integra accesos a los servicios multimedia del proyecto.

Estos servicios son gestionados por la parte multimedia del grupo, pero desde el servidor `web-sftp` se configuraron rutas en NGINX para que los usuarios puedan acceder a ellos desde el portal interno.

Los servicios integrados son:

| Servicio | Ruta desde el portal | Destino real |
|---|---|---|
| Radio / Icecast | `/radio/` | `http://32.198.236.17:8000/stream` |
| Jitsi Meet | `/jitsi/` | `https://3.219.249.6:8443/prueba-innovatetech` |
| Jellyfin | `/jellyfin/` | `http://32.198.236.17:8096/` |

El archivo de configuración utilizado fue:

```text
/etc/nginx/sites-available/innovatetech
```

### Radio / Icecast

Para el servicio de radio se configuró una redirección desde `/radio/` hacia el punto de montaje real del stream:

```nginx
location = /radio/ {
    return 302 /radio/stream;
}

location /radio/ {
    proxy_pass http://32.198.236.17:8000/;
    proxy_http_version 1.1;

    proxy_set_header Host 32.198.236.17;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_buffering off;
}
```

Con esta configuración, cuando un usuario entra en:

```text
https://52.1.67.249/radio/
```

NGINX lo redirige hacia:

```text
https://52.1.67.249/radio/stream
```

De esta forma, el usuario accede directamente al stream de audio desde el navegador.

### Jitsi Meet

Para la videoconferencia se configuró una redirección hacia la sala de prueba de Jitsi:

```nginx
location = /jitsi {
    return 301 /jitsi/;
}

location /jitsi/ {
    return 302 https://3.219.249.6:8443/prueba-innovatetech;
}
```

Inicialmente se probó un reverse proxy hacia Jitsi, pero Jitsi necesita que su `PUBLIC_URL` coincida con la URL final usada por los usuarios. Por ese motivo, se dejó configurada una redirección directa hacia el servidor real de Jitsi para evitar problemas con recursos, WebSocket o WebRTC.

La ruta del portal es:

```text
https://52.1.67.249/jitsi/
```

y redirige a la sala:

```text
https://3.219.249.6:8443/prueba-innovatetech
```

### Jellyfin

Para el servicio de vídeo se configuró un proxy hacia el servidor Jellyfin:

```nginx
location = /jellyfin {
    return 301 /jellyfin/;
}

location /jellyfin/ {
    proxy_pass http://32.198.236.17:8096/;
    proxy_http_version 1.1;

    proxy_set_header Host 32.198.236.17;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    proxy_buffering off;
    proxy_read_timeout 3600;
    proxy_send_timeout 3600;
}
```

Esta configuración permite acceder al servicio Jellyfin desde el portal usando la ruta:

```text
https://52.1.67.249/jellyfin/
```

### Enlaces desde el portal

Además de la configuración en NGINX, el dashboard interno muestra accesos directos a los servicios:

```text
/radio/
/jitsi/
/jellyfin/
```

De esta forma, los usuarios autenticados en el portal pueden acceder fácilmente a los servicios multimedia del proyecto.

Para comprobar las rutas configuradas se utilizaron los siguientes comandos:

```bash
curl -k -I https://52.1.67.249/radio/
curl -k -I https://52.1.67.249/radio/stream
curl -k -I https://52.1.67.249/jitsi/
curl -k -I https://52.1.67.249/jellyfin/
```

También se comprobó la configuración de NGINX:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Esta integración permite que el portal web actúe como punto central de acceso hacia los servicios multimedia del proyecto, aunque la instalación y configuración interna de Icecast, Jitsi y Jellyfin se documenta en la parte de multimedia.

**Evidencia:** captura donde se ve la configuración de NGINX para las rutas `/radio/`, `/jitsi/` y `/jellyfin/`.

<img width="900" height="223" alt="{275681F3-FCD0-4B1B-8FED-8D6266EEF1BD}" src="https://github.com/user-attachments/assets/3ea85d2e-4f96-4111-833b-27769df90ec2" />

En la captura se muestra la configuración de NGINX utilizada para integrar las rutas `/radio/`, `/jitsi/` y `/jellyfin/` desde el servidor `web-sftp`.

**Evidencia:** captura donde se comprueban las rutas con `curl`.

<img width="594" height="869" alt="{FB15CE69-3FDD-474D-9113-036A5D44B536}" src="https://github.com/user-attachments/assets/df814c79-1f70-4f3a-b4ab-224e3edaea98" />

En la captura se comprueba que las rutas multimedia responden desde el servidor `web-sftp`, permitiendo acceder a radio, videoconferencia y vídeo desde el portal.

**Evidencia:** captura del portal mostrando los botones de acceso a Radio, Jitsi y Jellyfin.

<img width="1082" height="128" alt="{AF632BFC-D56B-4CAB-BC56-959E4BA0E225}" src="https://github.com/user-attachments/assets/3bf49756-608f-435b-9040-831b630405ca" />

En la captura se comprueba que el dashboard interno incluye accesos directos a los servicios multimedia integrados.

## 20. Conclusión
El servidor `web-sftp` quedó configurado como uno de los puntos principales de acceso de la infraestructura de InnovateTech.

En esta máquina se integraron el servicio web, el acceso SFTP, la autenticación con Samba AD y la conexión con MariaDB. De esta forma, los usuarios internos pueden acceder al portal con sus credenciales corporativas, consultar información real de la base de datos y descargar datos en formato CSV o SQL según su perfil.

La configuración final permite:

- Publicar el portal interno mediante NGINX.
- Servir la web mediante HTTPS.
- Redirigir automáticamente HTTP hacia HTTPS.
- Utilizar un certificado SSL con `Subject Alternative Name`.
- Autenticar usuarios contra Samba AD mediante LDAP/TLS.
- Aplicar control de acceso por usuario dentro del portal.
- Conectar el portal con MariaDB usando usuarios con permisos mínimos.
- Descargar información en formato CSV o SQL de forma controlada.
- Permitir acceso SFTP con usuarios del dominio.
- Sincronizar datos entre SFTP y MariaDB mediante un script PHP.
- Automatizar la sincronización mediante `cron`.
- Integrar accesos hacia Radio, Jitsi y Jellyfin desde el dashboard.

Con esta configuración, el servidor `web-sftp` cumple los requisitos principales del proyecto relacionados con el servicio web, transferencia segura de ficheros, autenticación centralizada y conexión con la base de datos.

Además, el portal actúa como una aplicación de gestión interna, ya que no muestra datos estáticos, sino información obtenida directamente desde MariaDB. Esto permite que el sistema sea más realista y esté mejor integrado con el resto de servicios desplegados en AWS.

En conjunto, esta parte aporta una capa de acceso segura y centralizada para los usuarios de InnovateTech, conectando la infraestructura web con el Directorio Activo, la base de datos y los servicios multimedia del proyecto.
