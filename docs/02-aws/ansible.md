# Ansible — Automatització de la infraestructura

**Responsable:** Bilal El Younossi  
**Màquina:** ansible-controller (`32.193.193.146` / `10.0.7.201`)  
**Data:** Maig 2026

---

## 1. Descripció

Ansible s'utilitza per automatitzar la configuració de les màquines de la infraestructura InnovateTech. Des del `ansible-controller` es gestionen totes les instàncies EC2 mitjançant SSH amb clau pública/privada.

---

## 2. Instal·lació

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ansible -y
ansible --version
```

---

## 3. Estructura del projecte

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

## 4. Inventari — inventory.ini

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

### logs_baseline.yml — Configuració del servidor de logs

Configura `logs-server-private` com a receptor rsyslog i munta el EFS.

```yaml
---
- name: Configuració servidor de logs centralitzats
  hosts: logs
  become: yes
  tasks:
    - name: Instal·lar rsyslog i nfs-common
      apt:
        name:
          - rsyslog
          - nfs-common
        state: present
        update_cache: yes

    - name: Crear directori de muntatge EFS
      file:
        path: /mnt/efs-logs
        state: directory
        mode: '0777'

    - name: Muntar EFS
      mount:
        path: /mnt/efs-logs
        src: "10.0.142.148:/"
        fstype: nfs4
        opts: nfsvers=4.1,_netdev
        state: mounted

    - name: Configurar rsyslog per rebre logs remots
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

### logs_clients.yml — Configuració dels clients rsyslog

Configura totes les EC2 per enviar logs a `logs-server-private`.

```yaml
---
- name: Configuració clients rsyslog
  hosts: all
  become: yes
  tasks:
    - name: Instal·lar rsyslog
      apt:
        name: rsyslog
        state: present
        update_cache: yes

    - name: Configurar enviament de logs al servidor central
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

### mariadb.yml — Configuració de MariaDB

```yaml
---
- name: Configuració MariaDB
  hosts: mariadb
  become: yes
  tasks:
    - name: Instal·lar MariaDB
      apt:
        name:
          - mariadb-server
          - python3-pymysql
        state: present
        update_cache: yes

    - name: Habilitar i iniciar MariaDB
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

## 6. Execució dels playbooks

```bash
cd ~/proyecto-ibdt

# Verificar connectivitat amb totes les màquines
ansible all -i inventory/inventory.ini -m ping

# Executar configuració del servidor de logs
ansible-playbook -i inventory/inventory.ini playbooks/logs_baseline.yml

# Executar configuració dels clients
ansible-playbook -i inventory/inventory.ini playbooks/logs_clients.yml

# Executar configuració de MariaDB
ansible-playbook -i inventory/inventory.ini playbooks/mariadb.yml
```

---

## 7. Verificació

### Ping a totes les màquines

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

### Enviar logs de prova a totes les màquines

```bash
ansible all -i inventory/inventory.ini -m command \
  -a "logger 'InnovateTech sistema actiu $(date)'"
```

---

## 8. Màquines automatitzades

| Màquina | Playbook | Configuració aplicada |
|---------|----------|-----------------------|
| logs-server-private | logs_baseline.yml | rsyslog receptor + EFS muntat |
| web-sftp | logs_clients.yml | rsyslog client → logs-server |
| mariadb | logs_clients.yml + mariadb.yml | rsyslog client + MariaDB configurat |
| multimedia | logs_clients.yml | rsyslog client → logs-server |
| jitsi-meet | logs_clients.yml | rsyslog client → logs-server |
| samba-ad | logs_clients.yml | rsyslog client → logs-server |
| ansible-controller | logs_clients.yml | rsyslog client → logs-server |

---

## 9. Evidències

### Ping a totes les màquines
> Afegir captura de `ansible all -i inventory/inventory.ini -m ping`

### Execució del playbook logs_clients.yml
> Afegir captura de l'execució del playbook amb resultat OK

### Inventari configurat
> Afegir captura del fitxer `inventory.ini`
