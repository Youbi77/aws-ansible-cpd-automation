# Ansible — Automatización de la infraestructura

**Responsable:** Bilal El Younossi e Izan Velázquez Cerrato  
**Máquina:** ansible-controller (`32.193.193.146` / `10.0.7.201`)  
**Fecha:** Mayo 2026

---
## Índice

1. [Descripción](#1-descripción)
2. [Instalación](#2-instalación)
3. [Estructura del proyecto](#3-estructura-del-proyecto)
4. [Inventario — inventory.ini](#4-inventario--inventoryini)
5. [Playbooks](#5-playbooks)
   - [logs_baseline.yml — Configuración del servidor de logs](#logs_baselineyml--configuración-del-servidor-de-logs)
   - [logs_clients.yml — Configuración de los clientes rsyslog](#logs_clientsyml--configuración-de-los-clientes-rsyslog)
   - [mariadb.yml — Configuración de MariaDB](#mariadbyml--configuración-de-mariadb)
6. [Ejecución de los playbooks](#6-ejecución-de-los-playbooks)
7. [Verificación](#7-verificación)
   - [Ping a todas las máquinas](#ping-a-todas-las-máquinas)
   - [Enviar logs de prueba a todas las máquinas](#enviar-logs-de-prueba-a-todas-las-máquinas)
8. [Máquinas automatizadas](#8-máquinas-automatizadas)
9. [Evidencias](#9-evidencias)
   - [Ping a todas las máquinas](#ping-a-todas-las-máquinas-1)
   - [Ejecución del playbook logs_clients.yml](#ejecución-del-playbook-logs_clientsyml)
   - [Inventario configurado](#inventario-configurado)
     
## 1. Descripción

Ansible se utiliza para automatizar la configuración de las máquinas de la infraestructura InnovateTech. Desde el `ansible-controller` se gestionan todas las instancias EC2 mediante SSH con clave pública/privada.

---

## 2. Instalación

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ansible -y
ansible --version
```

---

## 3. Estructura del proyecto

```
~/proyecto-ibdt/
├── inventory/
│   └── inventory.ini
└── playbooks/
    ├── logs_baseline.yml
    ├── logs_clients.yml
    └── mariadb.yml
```

---

## 4. Inventario — inventory.ini

```ini
[ansible_controller]
32.193.193.146 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/I.pem

[web]
10.0.5.140 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/T.pem

[mariadb]
10.0.142.205 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/I.pem

[multimedia]
10.0.8.36 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/S.pem

[jitsi]
10.0.14.189 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/S.pem

[samba]
10.0.141.9 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/T.pem

[logs]
10.0.133.107 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/I.pem

[all:vars]
ansible_become=true
ansible_become_method=sudo
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---

## 5. Playbooks

### logs_baseline.yml — Configuración del servidor de logs

Configura `logs-server-private` como receptor rsyslog y monta el EFS.

```yaml
---
- name: Configuración servidor de logs centralizados
  hosts: logs
  become: yes
  tasks:
    - name: Instalar rsyslog y nfs-common
      apt:
        name:
          - rsyslog
          - nfs-common
        state: present
        update_cache: yes

    - name: Crear directorio de montaje EFS
      file:
        path: /mnt/efs-logs
        state: directory
        mode: '0777'

    - name: Montar EFS
      mount:
        path: /mnt/efs-logs
        src: "10.0.142.148:/"
        fstype: nfs4
        opts: nfsvers=4.1,_netdev
        state: mounted

    - name: Configurar rsyslog para recibir logs remotos
      blockinfile:
        path: /etc/rsyslog.conf
        block: |
          module(load="imudp")
          input(type="imudp" port="514")
          module(load="imtcp")
          input(type="imtcp" port="514")
          $template RemoteLogs,"/mnt/efs-logs/%HOSTNAME%/%PROGRAMNAME%.log"
          *.* ?RemoteLogs

    - name: Reiniciar rsyslog
      systemd:
        name: rsyslog
        state: restarted
        enabled: yes
```

### logs_clients.yml — Configuración de los clientes rsyslog

Configura todas las EC2 para enviar logs a `logs-server-private`.

```yaml
---
- name: Configuración clientes rsyslog
  hosts: all
  become: yes
  tasks:
    - name: Instalar rsyslog
      apt:
        name: rsyslog
        state: present
        update_cache: yes

    - name: Configurar envío de logs al servidor central
      copy:
        content: "*.* @@10.0.133.107:514\n"
        dest: /etc/rsyslog.d/99-remote.conf
        mode: '0644'

    - name: Reiniciar rsyslog
      systemd:
        name: rsyslog
        state: restarted
        enabled: yes
```

### mariadb.yml — Configuración de MariaDB

```yaml
---
- name: Configuración MariaDB
  hosts: mariadb
  become: yes
  tasks:
    - name: Instalar MariaDB
      apt:
        name:
          - mariadb-server
          - python3-pymysql
        state: present
        update_cache: yes

    - name: Habilitar e iniciar MariaDB
      systemd:
        name: mariadb
        state: started
        enabled: yes

    - name: Configurar bind-address a IP privada
      lineinfile:
        path: /etc/mysql/mariadb.conf.d/50-server.cnf
        regexp: '^bind-address'
        line: 'bind-address = 10.0.142.205'

    - name: Reiniciar MariaDB
      systemd:
        name: mariadb
        state: restarted
```

---

## 6. Ejecución de los playbooks

```bash
cd ~/proyecto-ibdt

# Verificar conectividad con todas las máquinas
ansible all -i inventory/inventory.ini -m ping

# Ejecutar configuración del servidor de logs
ansible-playbook -i inventory/inventory.ini playbooks/logs_baseline.yml

# Ejecutar configuración de los clientes
ansible-playbook -i inventory/inventory.ini playbooks/logs_clients.yml

# Ejecutar configuración de MariaDB
ansible-playbook -i inventory/inventory.ini playbooks/mariadb.yml
```

---

## 7. Verificación

### Ping a todas las máquinas

```bash
$ ansible all -i inventory/inventory.ini -m ping
10.0.5.140 | SUCCESS => { "ping": "pong" }
10.0.142.205 | SUCCESS => { "ping": "pong" }
10.0.8.36 | SUCCESS => { "ping": "pong" }
10.0.14.189 | SUCCESS => { "ping": "pong" }
10.0.133.107 | SUCCESS => { "ping": "pong" }
10.0.141.9 | SUCCESS => { "ping": "pong" }
32.193.193.146 | SUCCESS => { "ping": "pong" }
```

### Enviar logs de prueba a todas las máquinas

```bash
ansible all -i inventory/inventory.ini -m command \
  -a "logger 'InnovateTech sistema activo $(date)'"
```

---

## 8. Máquinas automatizadas

| Máquina | Playbook | Configuración aplicada |
|---------|----------|------------------------|
| logs-server-private | logs_baseline.yml | rsyslog receptor + EFS montado |
| web-sftp | logs_clients.yml | rsyslog cliente → logs-server |
| mariadb | logs_clients.yml + mariadb.yml | rsyslog cliente + MariaDB configurado |
| multimedia | logs_clients.yml | rsyslog cliente → logs-server |
| jitsi-meet | logs_clients.yml | rsyslog cliente → logs-server |
| samba-ad | logs_clients.yml | rsyslog cliente → logs-server |
| ansible-controller | logs_clients.yml | rsyslog cliente → logs-server |

---

## 9. Evidencias

### Ping a todas las máquinas

<img width="708" height="942" alt="Ansible ping todas las máquinas" src="https://github.com/user-attachments/assets/43a01b42-7347-4dc3-8a19-7a8b7d9fa545" />

### Ejecución del playbook logs_clients.yml

<img width="714" height="797" alt="Ejecución playbook logs_clients" src="https://github.com/user-attachments/assets/9d3d809c-c895-459e-b74b-5d8d1054ccd8" />

### Inventario configurado

<img width="711" height="474" alt="Inventario Ansible" src="https://github.com/user-attachments/assets/d56caeda-df75-464c-a5c4-649c0f95476b" />
