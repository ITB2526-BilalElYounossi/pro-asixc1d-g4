# Automatización con Ansible

---

# 1. Objetivo

El objetivo de este apartado es automatizar la configuración de la infraestructura de InnovateTech mediante Ansible.

Se utiliza una máquina dedicada como controlador Ansible desde la cual se administran el servidor MariaDB y el servidor de logs.

<img width="594" height="203" alt="imatge" src="https://github.com/user-attachments/assets/9a980426-4baf-4918-b022-b69bd16f3f8b" />

---

# 2. Arquitectura Ansible

| Máquina | IP | Función |
|---|---|---|
| ansible-controller | 32.193.193.146 | Controlador Ansible |
| mariadb | 10.0.142.205 | Servidor de base de datos |
| logs-server | 100.49.230.2 | Servidor de logs |

---

# 3. Instalación de Ansible

Comandos utilizados:

```bash
sudo apt update
sudo apt install ansible git -y
ansible --version
```

📸 Captura:

<img width="829" height="223" alt="imatge" src="https://github.com/user-attachments/assets/5a998997-5d39-4d19-8336-70a8a1418be9" />

---

# 4. Configuración del inventario

Archivo utilizado:

```bash
inventory/inventory.ini
```

Contenido:

```ini
[ansible_controller]
32.193.193.146 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/I.pem

[mariadb]
10.0.142.205 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/I.pem

[logs]
100.49.230.2 ansible_user=adminitb ansible_ssh_private_key_file=~/.ssh/I.pem

[all:vars]
ansible_become=true
ansible_become_method=sudo

```

📸 Captura:

<img width="715" height="241" alt="imatge" src="https://github.com/user-attachments/assets/52eda657-3d7a-4aaf-a65d-74e977300fda" />

---

# 5. Prueba de conectividad

Comando utilizado:

```bash
ansible all -i inventory/inventory.ini -m ping
```

Resultado esperado:
- Todas las máquinas deben responder con `SUCCESS`.

📸 Captura:

<img width="827" height="403" alt="imatge" src="https://github.com/user-attachments/assets/7ab1eb0f-02a9-422e-900a-c2e8c83499ea" />

---

# 6. Automatización del servidor MariaDB

Playbook utilizado:

```bash
playbooks/mariadb.yml
```

Este playbook automatiza:

- instalación de MariaDB,
- configuración del servicio,
- configuración del `bind-address` privado,
- copia de los archivos SQL al servidor,
- creación de la base de datos,
- creación de tablas,
- inserción de datos,
- creación de roles,
- creación de triggers,
- creación de eventos automáticos.

📸 Captura:

<img width="671" height="905" alt="imatge" src="https://github.com/user-attachments/assets/486fae39-c7bb-4fd5-b2bd-c19839284ee7" />

---

# 7. Ejecución del playbook MariaDB

Comando ejecutado:

```bash
ansible-playbook -i inventory/inventory.ini playbooks/mariadb.yml
```

📸 Captura:

<img width="1036" height="906" alt="imatge" src="https://github.com/user-attachments/assets/91368929-6df0-4965-bdc0-4e88f5047608" />

---

# 8. Automatización del servidor de logs

Playbook utilizado:

```bash
playbooks/logs_baseline.yml
```

Este playbook automatiza:

- actualización de repositorios,
- instalación de `rsyslog`, `logrotate`, `htop` y `curl`,
- activación del servicio `rsyslog`,
- creación de la carpeta `/opt/innovatetech-logs`,
- creación de un archivo README de validación.

📸 Captura:

<img width="773" height="636" alt="imatge" src="https://github.com/user-attachments/assets/287757d0-f894-4c18-8852-2cd8d2cc5fb2" />

---

# 9. Ejecución del playbook Logs

Comando ejecutado:

```bash
ansible-playbook -i inventory/inventory.ini playbooks/logs_baseline.yml
```

📸 Captura:

<img width="1091" height="439" alt="imatge" src="https://github.com/user-attachments/assets/53b62852-92ba-4682-8312-d6534c3b01be" />

---

# 10. Validación final

Comandos utilizados:

```bash
ansible all -i inventory/inventory.ini -m ping
```

📸 Captura:

<img width="825" height="401" alt="imatge" src="https://github.com/user-attachments/assets/945fdf6a-473e-479f-a941-32ea097dd9a8" />

---

# 11. Conclusión

La infraestructura de InnovateTech ha sido automatizada correctamente mediante Ansible utilizando una máquina controladora centralizada.

Se han desplegado y configurado automáticamente los servicios de MariaDB y del servidor de logs, permitiendo una administración más eficiente, reproducible y segura.

La automatización reduce errores manuales y facilita futuras ampliaciones y mantenimientos de la infraestructura.
