# Infraestructura AWS — InnovateTech CPD
**Projecte:** pro-asixc1X-gX | **Grup:** ASIXc1 | **Curs:** 25/26

---

## 1. Visió General de l'Arquitectura

```
Internet
    │
    ▼
[Internet Gateway]
    │
    ▼
[Application Load Balancer] ──── HTTPS/HTTP ───► EC2-1 (Web + SFTP)
    │
[VPC: 10.0.0.0/16]
    ├── Subnet Pública  10.0.1.0/24  (eu-west-1a)
    │       ├── EC2-1  Web + SFTP
    │       ├── EC2-3  Jellyfin + Icecast2
    │       ├── EC2-4  Jitsi Meet
    │       └── EC2-6  Ansible Controller
    │
    └── Subnet Privada 10.0.2.0/24  (eu-west-1a)
            ├── EC2-2  Samba AD (Directori Actiu)
            └── EC2-5  MariaDB
```

---

## 2. VPC i Xarxa

### 2.1 VPC Principal

| Paràmetre        | Valor            |
|------------------|------------------|
| Nom              | `vpc-innovatetech` |
| CIDR Block       | `10.0.0.0/16`    |
| Regió            | `eu-west-1` (Irlanda) |
| DNS Hostnames    | Activat          |
| DNS Resolution   | Activat          |

### 2.2 Subnets

| Nom                        | CIDR           | AZ           | Tipus   | Auto-assign IP |
|----------------------------|----------------|--------------|---------|----------------|
| `subnet-publica-innovatetech`  | `10.0.1.0/24`  | eu-west-1a   | Pública | Sí             |
| `subnet-privada-innovatetech`  | `10.0.2.0/24`  | eu-west-1a   | Privada | No             |

### 2.3 Internet Gateway

```
Nom: igw-innovatetech
Associat a: vpc-innovatetech
```

### 2.4 Taules de Rutes

**Route Table Pública:**

| Destí         | Target                 |
|---------------|------------------------|
| `10.0.0.0/16` | local                  |
| `0.0.0.0/0`   | igw-innovatetech        |

**Route Table Privada:**

| Destí         | Target |
|---------------|--------|
| `10.0.0.0/16` | local  |

> Les instàncies privades (MariaDB, Samba AD) no tenen accés a internet. Es gestionen via Ansible des de la subnet pública.

### 2.5 NAT Gateway (opcional per actualizaciones de paquetes en privada)

```
Nom:       nat-innovatetech
Subnet:    subnet-publica-innovatetech
Elastic IP: assignada automàticament
```

---

## 3. Security Groups

### SG-ALB — Application Load Balancer

| Tipus   | Protocol | Port | Origen      | Descripció          |
|---------|----------|------|-------------|---------------------|
| Inbound | TCP      | 80   | 0.0.0.0/0   | HTTP públic         |
| Inbound | TCP      | 443  | 0.0.0.0/0   | HTTPS públic        |
| Outbound| All      | All  | 0.0.0.0/0   | Tot el tràfic       |

### SG-WEB — EC2-1 Web + SFTP

| Tipus   | Protocol | Port  | Origen         | Descripció               |
|---------|----------|-------|----------------|--------------------------|
| Inbound | TCP      | 80    | SG-ALB         | HTTP des del ALB         |
| Inbound | TCP      | 443   | SG-ALB         | HTTPS des del ALB        |
| Inbound | TCP      | 22    | 10.0.1.0/24    | SSH (Ansible)            |
| Inbound | TCP      | 22    | (IP admin)     | SSH administració        |
| Outbound| All      | All   | 0.0.0.0/0      | Tot el tràfic            |

### SG-MULTIMEDIA — EC2-3 Jellyfin + Icecast2

| Tipus   | Protocol | Port  | Origen         | Descripció               |
|---------|----------|-------|----------------|--------------------------|
| Inbound | TCP      | 8096  | 0.0.0.0/0      | Jellyfin Web UI          |
| Inbound | TCP      | 8000  | 0.0.0.0/0      | Icecast2 streaming       |
| Inbound | TCP      | 22    | 10.0.1.0/24    | SSH (Ansible)            |
| Outbound| All      | All   | 0.0.0.0/0      | Tot el tràfic            |

### SG-JITSI — EC2-4 Jitsi Meet

| Tipus   | Protocol | Port      | Origen      | Descripció                  |
|---------|----------|-----------|-------------|-----------------------------|
| Inbound | TCP      | 80        | 0.0.0.0/0   | HTTP                        |
| Inbound | TCP      | 443       | 0.0.0.0/0   | HTTPS                       |
| Inbound | UDP      | 10000     | 0.0.0.0/0   | WebRTC media                |
| Inbound | TCP      | 4443      | 0.0.0.0/0   | JVB fallback TCP            |
| Inbound | TCP      | 22        | 10.0.1.0/24 | SSH (Ansible)               |
| Outbound| All      | All       | 0.0.0.0/0   | Tot el tràfic               |

### SG-DB — EC2-5 MariaDB (Privada)

| Tipus   | Protocol | Port  | Origen         | Descripció               |
|---------|----------|-------|----------------|--------------------------|
| Inbound | TCP      | 3306  | 10.0.1.0/24    | MySQL des de subnet públ |
| Inbound | TCP      | 22    | 10.0.1.0/24    | SSH (Ansible)            |
| Outbound| All      | All   | 10.0.0.0/16    | Tràfic intern VPC        |

### SG-AD — EC2-2 Samba AD (Privada)

| Tipus   | Protocol | Port  | Origen         | Descripció               |
|---------|----------|-------|----------------|--------------------------|
| Inbound | TCP      | 389   | 10.0.1.0/24    | LDAP                     |
| Inbound | TCP      | 636   | 10.0.1.0/24    | LDAPS                    |
| Inbound | TCP/UDP  | 445   | 10.0.1.0/24    | SMB                      |
| Inbound | TCP/UDP  | 88    | 10.0.1.0/24    | Kerberos                 |
| Inbound | TCP      | 22    | 10.0.1.0/24    | SSH (Ansible)            |
| Outbound| All      | All   | 10.0.0.0/16    | Tràfic intern VPC        |

### SG-ANSIBLE — EC2-6 Ansible Controller

| Tipus   | Protocol | Port | Origen         | Descripció               |
|---------|----------|------|----------------|--------------------------|
| Inbound | TCP      | 22   | (IP admin)     | SSH administrador        |
| Outbound| TCP      | 22   | 10.0.0.0/16    | SSH a totes les EC2      |
| Outbound| All      | All  | 0.0.0.0/0      | Accés internet           |

---

## 4. Instàncies EC2

| ID    | Nom                  | AMI                    | Tipus       | Subnet   | IP Privada  | IP Pública | SG          |
|-------|----------------------|------------------------|-------------|----------|-------------|------------|-------------|
| EC2-1 | web-sftp             | Ubuntu 22.04 LTS       | t3.small    | Pública  | 10.0.1.10   | Elastic IP | SG-WEB      |
| EC2-2 | samba-ad             | Ubuntu 22.04 LTS       | t3.medium   | Privada  | 10.0.2.10   | —          | SG-AD       |
| EC2-3 | multimedia           | Ubuntu 22.04 LTS       | t3.medium   | Pública  | 10.0.1.20   | Elastic IP | SG-MULTIMEDIA |
| EC2-4 | jitsi-meet           | Ubuntu 22.04 LTS       | t3.medium   | Pública  | 10.0.1.30   | Elastic IP | SG-JITSI    |
| EC2-5 | mariadb              | Ubuntu 22.04 LTS       | t3.small    | Privada  | 10.0.2.20   | —          | SG-DB       |
| EC2-6 | ansible-controller   | Ubuntu 22.04 LTS       | t3.micro    | Pública  | 10.0.1.40   | Elastic IP | SG-ANSIBLE  |

### Configuració comuna a totes les instàncies

```bash
# Usuari d'administració (NO usar ec2-user ni ubuntu per defecte)
Usuari: adminitb
Autenticació: Clau pública/privada (sense contrasenya)

# Key Pair
Nom: keypair-innovatetech
Tipus: RSA 4096 bits
Format: .pem
```

### Volums EBS per instància

| Instància | Volum Root | Volum Addicional | Motiu                        |
|-----------|-----------|------------------|------------------------------|
| EC2-1     | 20 GB gp3 | —                | Web + SFTP                   |
| EC2-2     | 20 GB gp3 | —                | Samba AD                     |
| EC2-3     | 30 GB gp3 | —                | Jellyfin + Icecast2 + vídeos |
| EC2-4     | 20 GB gp3 | —                | Jitsi Meet                   |
| EC2-5     | 20 GB gp3 | —                | MariaDB dades                |
| EC2-6     | 10 GB gp3 | —                | Ansible (lleuger)            |

---

## 5. Application Load Balancer (ALB)

```
Nom:      alb-innovatetech
Tipus:    Application Load Balancer
Scheme:   Internet-facing
Subnet:   subnet-publica-innovatetech
SG:       SG-ALB
```

### Listeners i Regles

| Listener | Port | Protocol | Acció                                    |
|----------|------|----------|------------------------------------------|
| HTTP     | 80   | HTTP     | Redirigir a HTTPS (301)                  |
| HTTPS    | 443  | HTTPS    | Forward → Target Group `tg-web-innovatetech` |

### Target Group

```
Nom:               tg-web-innovatetech
Protocol:          HTTP
Port:              80
Target type:       instance
Health check path: /
Threshold:         2 checks consecutius OK
Interval:          30 s
```

### Targets registrats

| Instància | Port | Estat esperat |
|-----------|------|---------------|
| EC2-1     | 80   | healthy        |

---

## 6. Emmagatzematge S3

### Bucket Principal

```
Nom:    s3-innovatetech-media
Regió:  eu-west-1
Accés:  Privat (sense accés públic)
Versioning: Activat
```

### Estructura de carpetes

```
s3-innovatetech-media/
    ├── videos/          → Vídeos de Jellyfin
    ├── backups/
    │   ├── mariadb/     → Backups de MariaDB (mysqldump)
    │   └── configs/     → Backups de configuració dels servidors
    └── logs/            → Arxius de logs centralitzats
```

### Bucket per Backups

```
Nom:           s3-innovatetech-backups
Versioning:    Activat
Lifecycle:     Eliminar versions antigues > 30 dies
```

### IAM Role per accés S3 des de les EC2

```
Nom Role:   role-ec2-s3-innovatetech
Política:   AmazonS3FullAccess (restringida al bucket)
Associat a: EC2-3 (multimedia), EC2-5 (mariadb backups)
```

---

## 7. EFS — Elastic File System

```
Nom:           efs-innovatetech-shared
Tipus:         Regional (alta disponibilitat)
Performance:   General Purpose
Throughput:    Bursting
Xifrat:        Activat (AES-256)
```

### Mount Targets

| Subnet              | IP              |
|---------------------|-----------------|
| subnet-publica-innovatetech | 10.0.1.x |

### Instàncies amb EFS muntat

| Instància | Punt de muntatge          | Ús                          |
|-----------|---------------------------|-----------------------------|
| EC2-1     | `/mnt/efs/sftp`           | Fitxers compartits SFTP     |
| EC2-3     | `/mnt/efs/media`          | Vídeos Jellyfin compartits  |
| EC2-6     | `/mnt/efs/ansible`        | Playbooks i inventaris      |

### Comanda de muntatge (a posar al /etc/fstab)

```bash
# Instal·lar client EFS
sudo apt install nfs-common amazon-efs-utils -y

# Muntar (substituir fs-XXXXXXXX pel FileSystem ID real)
sudo mount -t efs -o tls fs-XXXXXXXX:/ /mnt/efs/sftp

# Afegir a /etc/fstab per persistència
fs-XXXXXXXX:/ /mnt/efs/sftp efs _netdev,tls 0 0
```

---

## 8. CloudWatch — Monitoratge

### Log Groups

| Log Group                          | Origen               | Retencio |
|------------------------------------|----------------------|----------|
| `/innovatetech/web/nginx`           | EC2-1 nginx access   | 30 dies  |
| `/innovatetech/web/sftp`            | EC2-1 sftp logs      | 30 dies  |
| `/innovatetech/multimedia/jellyfin` | EC2-3 jellyfin logs  | 14 dies  |
| `/innovatetech/multimedia/icecast`  | EC2-3 icecast logs   | 14 dies  |
| `/innovatetech/jitsi`               | EC2-4 jitsi logs     | 14 dies  |
| `/innovatetech/db/mariadb`          | EC2-5 mariadb logs   | 30 dies  |
| `/innovatetech/syslog`              | Tots els servidors   | 7 dies   |

### CloudWatch Agent — instal·lació a cada EC2

```bash
# Descarregar i instal·lar agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb

# Iniciar amb la configuració
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
    -a fetch-config -m ec2 -s \
    -c ssm:/AmazonCloudWatch-Config
```

### Alarmes CloudWatch

| Alarma                        | Mètrica               | Threshold | Acció              |
|-------------------------------|-----------------------|-----------|--------------------|
| `alm-ec2-cpu-high`            | CPUUtilization        | > 80%     | Notificació SNS    |
| `alm-ec2-disk-low`            | disk_used_percent     | > 85%     | Notificació SNS    |
| `alm-alb-5xx-errors`          | HTTPCode_ELB_5XX      | > 10/min  | Notificació SNS    |
| `alm-alb-latency`             | TargetResponseTime    | > 2s      | Notificació SNS    |
| `alm-db-connections`          | DatabaseConnections   | > 80      | Notificació SNS    |

### SNS Topic per notificacions

```
Nom:    sns-innovatetech-alerts
Tipus:  Email
Email:  admin@innovatetech.com
```

---

## 9. Elastic IPs

| Nom                    | Associada a | Descripció                    |
|------------------------|-------------|-------------------------------|
| `eip-web-innovatetech` | EC2-1       | IP fixa pel servidor web/SFTP |
| `eip-multimedia`       | EC2-3       | IP fixa Jellyfin/Icecast2     |
| `eip-jitsi`            | EC2-4       | IP fixa Jitsi Meet            |
| `eip-ansible`          | EC2-6       | IP fixa gestió Ansible        |

---

## 10. IAM — Gestió d'Accés

### Rols IAM per EC2

| Rol                          | Permisos                          | Associat a       |
|------------------------------|-----------------------------------|------------------|
| `role-ec2-cloudwatch`        | CloudWatchAgentServerPolicy       | Totes les EC2    |
| `role-ec2-s3-media`          | S3 read/write s3-innovatetech-media | EC2-3          |
| `role-ec2-s3-backups`        | S3 write s3-innovatetech-backups  | EC2-5            |
| `role-ec2-ssm`               | AmazonSSMManagedInstanceCore      | Totes les EC2    |

---

## 11. Resum de Costos Estimats (Free Tier / t3.micro→small)

| Recurs              | Tipus       | Cost aprox/mes  |
|---------------------|-------------|-----------------|
| 6× EC2 t3.small/med | Compute     | ~50-80 €        |
| 6× EBS gp3          | Storage     | ~8 €            |
| S3 (< 5 GB)         | Storage     | ~0.10 €         |
| EFS (< 1 GB)        | Storage     | ~0.05 €         |
| ALB                 | Networking  | ~18 €           |
| NAT Gateway         | Networking  | ~35 €           |
| CloudWatch Logs     | Monitoring  | ~2 €            |
| **Total estimat**   |             | **~115-145 €/mes** |

> ⚠️ Aturar les instàncies quan no s'usin per reduir costos durant el projecte.

---

## 12. Procediment de Desplegament (Ordre recomanat)

```
1. Crear VPC + subnets + IGW + Route Tables
2. Crear Security Groups (tots)
3. Crear Key Pair keypair-innovatetech
4. Llançar EC2-6 (Ansible) → subnet pública
5. Llançar EC2-1 (Web/SFTP) → subnet pública
6. Llançar EC2-2 (Samba AD) → subnet privada
7. Llançar EC2-3 (Multimedia) → subnet pública
8. Llançar EC2-4 (Jitsi) → subnet pública
9. Llançar EC2-5 (MariaDB) → subnet privada
10. Assignar Elastic IPs a EC2-1, EC2-3, EC2-4, EC2-6
11. Crear i configurar ALB + Target Group + Listener
12. Crear bucket S3 + configurar polítiques
13. Crear EFS + muntar a EC2-1, EC2-3, EC2-6
14. Instal·lar CloudWatch Agent a totes les EC2
15. Configurar alarmes CloudWatch + SNS
16. Verificar connectivitat entre subnets
17. Executar playbooks Ansible des de EC2-6
```

---

*Documentació generada per al Projecte Transversal ASIXc1 — InnovateTech*
*Institut Tecnològic de Barcelona — Curs 25/26*
