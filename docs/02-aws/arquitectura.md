# Infraestructura AWS — InnovateTech

**Responsable:** Bilal El Younossi  
**Data:** Maig 2026

---

## 1. Visió general

Infraestructura desplegada a AWS regió `us-east-1` amb VPC dedicada, subnets pública i privada, 7 instàncies EC2, ALB, EFS compartit i serveis de xarxa.

---

## 2. Diagrama de l'arquitectura

<img width="607" height="694" alt="imatge" src="https://github.com/user-attachments/assets/3095c451-b0ae-4596-ae93-a213af9b0e2e" />


---

## 3. VPC i Xarxa

| Paràmetre | Valor |
|-----------|-------|
| Nom | vpc-innovatetech-vpc |
| ID | vpc-0837b38c8e1749eeb |
| CIDR | 10.0.0.0/16 |
| Regió | us-east-1 |
| DNS Hostnames | Activat |
| DNS Resolution | Activat |

### Subnets

| Nom | ID | CIDR | Tipus | AZ |
|-----|----|------|-------|----|
| innovatetech-subPublica | subnet-001c139d2c6686a83 | 10.0.0.0/20 | Pública | us-east-1a |
| innovatetech-subPublica2 | subnet-040af8871129f2119 | 10.0.50.0/24 | Pública | us-east-1b |
| innovatetech-subPrivada | subnet-0e35e1475b888ea55 | 10.0.128.0/20 | Privada | us-east-1a |

### Gateways i routing

| Recurs | Funció |
|--------|--------|
| Internet Gateway | Connexió subnet pública a internet |
| NAT Gateway | Connexió subnet privada a internet (sortida) |
| ALB alb-innovatetech | Load Balancer davant de web-sftp, HTTP→HTTPS |

---

## 4. Instàncies EC2

| Màquina | IP Pública | IP Privada | Subnet | Key |
|---------|-----------|------------|--------|-----|
| web-sftp | 52.1.67.249 | 10.0.5.140 | Pública | T.pem |
| multimedia | 32.198.236.17 | 10.0.8.36 | Pública | S.pem |
| jitsi-meet | 3.219.249.6 | 10.0.14.189 | Pública | S.pem |
| ansible-controller | 32.193.193.146 | 10.0.7.201 | Pública | I.pem |
| samba-ad | — | 10.0.141.9 | Privada | T.pem |
| mariadb | — | 10.0.142.205 | Privada | I.pem |
| logs-server-private | — | 10.0.133.107 | Privada | I.pem |

### Configuració comuna

- SO: Ubuntu 24.04 LTS
- Disc: 8 GB gp3 (logs-server-private: 20 GB)
- Usuari administrador: `adminitb`
- Autenticació: clau pública/privada (sense contrasenya)
- Sudo: sense contrasenya

---

## 5. EFS — Sistema de fitxers compartit

| Paràmetre | Valor |
|-----------|-------|
| Nom | efs-innovatetech-logs |
| ID | fs-09f8489813ea7d132 |
| Mount target pública2 | 10.0.50.145 (subnet-040af8871129f2119) |
| Mount target privada | 10.0.142.148 (subnet-0e35e1475b888ea55) |
| Security Group | SG-EFS (sg-002a5243a5fb9a320) |
| Muntat a | /mnt/efs-logs |
| Capacitat | 8.0E (elàstica) |

El EFS és compartit entre `web-sftp` i `logs-server-private`. El servidor de logs escriu els logs de totes les EC2 al EFS via rsyslog, i web-sftp els llegeix directament per mostrar-los al portal web.

---

## 6. Security Groups

### SG-WEB
| Direcció | Protocol | Port | Origen/Destí |
|----------|----------|------|--------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Entrada | TCP | 80 | 0.0.0.0/0 |
| Entrada | TCP | 443 | 0.0.0.0/0 |
| Sortida | TCP | 3306 | 10.0.0.0/16 |
| Sortida | TCP | 389 | 10.0.0.0/16 |
| Sortida | TCP/UDP | 514 | 10.0.0.0/16 |
| Sortida | TCP | 2049 | 10.0.0.0/16 |

### SG-MULTIMEDIA
| Direcció | Protocol | Port | Origen/Destí |
|----------|----------|------|--------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 8096 | 0.0.0.0/0 |
| Sortida | Tot | Tot | 0.0.0.0/0 |

### SG-JITSI
| Direcció | Protocol | Port | Origen/Destí |
|----------|----------|------|--------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 8443 | 0.0.0.0/0 |
| Entrada | UDP | 10000 | 0.0.0.0/0 |
| Sortida | TCP/UDP | 514 | 10.0.0.0/16 |

### SG-AD (Samba AD)
| Direcció | Protocol | Port | Origen/Destí |
|----------|----------|------|--------------|
| Entrada | TCP | 22 | 10.0.0.0/16 |
| Entrada | TCP/UDP | 88 | 10.0.0.0/16 |
| Entrada | TCP/UDP | 389 | 10.0.0.0/16 |
| Entrada | TCP | 445 | 10.0.0.0/16 |
| Entrada | TCP | 636 | 10.0.0.0/16 |
| Sortida | TCP | 80/443 | 0.0.0.0/0 |

### SG-DB (MariaDB)
| Direcció | Protocol | Port | Origen/Destí |
|----------|----------|------|--------------|
| Entrada | TCP | 22 | 10.0.0.0/16 |
| Entrada | TCP | 3306 | 10.0.0.0/16 |
| Sortida | TCP | 80/443 | 0.0.0.0/0 |

### SG-ANSIBLE
| Direcció | Protocol | Port | Origen/Destí |
|----------|----------|------|--------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Sortida | TCP | 22 | 10.0.0.0/16 |
| Sortida | TCP | 80/443 | 0.0.0.0/0 |

### SG-LOGS
| Direcció | Protocol | Port | Origen/Destí |
|----------|----------|------|--------------|
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Entrada | TCP | 514 | 10.0.0.0/16 |
| Entrada | UDP | 514 | 10.0.0.0/16 |
| Entrada | ICMP | Tot | 10.0.0.0/16 |
| Sortida | TCP | 80/443 | 0.0.0.0/0 |
| Sortida | TCP | 2049 | 10.0.0.0/16 |

### SG-EFS
| Direcció | Protocol | Port | Origen/Destí |
|----------|----------|------|--------------|
| Entrada | TCP | 2049 | 0.0.0.0/0 |
| Entrada | Tot | Tot | 0.0.0.0/0 |
| Sortida | TCP | 2049 | 0.0.0.0/0 |
| Sortida | Tot | Tot | 0.0.0.0/0 |

---

## 7. Justificació de l'arquitectura

### Per què subnet pública i privada?

Les màquines que han de ser accessibles des d'internet (web, multimedia, jitsi, ansible) estan a la subnet pública. Les màquines que només han de ser accessibles internament (samba-ad, mariadb, logs-server) estan a la subnet privada amb accés a internet via NAT Gateway per a actualitzacions.

### Per què EFS en lloc de disc local per als logs?

El EFS permet que `logs-server-private` escrigui els logs i que `web-sftp` els llegeixi simultàniament sense necessitat d'una API intermèdia. És elàstic, no es pot omplir i té alta disponibilitat multi-AZ.

### Per què ALB davant de web-sftp?

L'ALB permet afegir futures instàncies web en alta disponibilitat sense canviar la URL pública. També gestiona la terminació SSL i redirigeix HTTP a HTTPS.

---

## 8. Evidències

### Instàncies EC2 en execució
> Afegir captura de pantalla de la consola AWS amb les 7 instàncies actives.

### VPC i Subnets
> Afegir captura de pantalla de la configuració de VPC.

### EFS muntat
```bash
$ df -h | grep mnt
10.0.50.145:/   8.0E     0  8.0E   0% /mnt/efs-logs   # web-sftp
10.0.142.148:/  8.0E  623M  8.0E   1% /mnt/efs-logs   # logs-server-private
```
