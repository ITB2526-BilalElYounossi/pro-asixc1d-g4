# Projecte Transversal ASIXc1 — InnovateTech

**Grup:** TIBS  
**Curs:** 2025/2026  
**Institut:** Institut Tecnològic de Barcelona  
**Mòduls:** 0371, 0375, 0377, 1665

---

## Equip

| Nom | Responsabilitat |
|-----|----------------|
| Bilal | Infraestructura AWS + Git |
| Izan | Ansible + MariaDB |
| Serghei | Samba AD + Jitsi Meet |
| Taylor | Web/SFTP + Multimedia |

---

## Descripció

Disseny i implementació d'un Centre de Processament de Dades (CPD) al núvol AWS per a l'empresa **InnovateTech**, incloent serveis multimèdia, base de dades, directori actiu i gestió centralitzada de logs.

---

## Arquitectura AWS

```
Internet
    │
    ▼
[Internet Gateway]
    │
    ▼
[Application Load Balancer]
    │
[VPC: 10.0.0.0/16]
    ├── Subnet Pública 10.0.1.0/24
    │       ├── EC2-1  web-sftp        52.1.67.249
    │       ├── EC2-3  multimedia      32.198.236.17
    │       ├── EC2-4  jitsi-meet      3.219.249.6
    │       ├── EC2-6  ansible         32.193.193.146
    │       └── EC2-7  logs-server     100.49.230.2
    │
    └── Subnet Privada 10.0.2.0/24
            ├── EC2-2  samba-ad        (IP privada)
            └── EC2-5  mariadb         (IP privada)
```

---

## Estructura del repositori

```
├── docs/
│   ├── 01-CPD/          → Proposta CPD físic
│   ├── 02-AWS/          → Infraestructura al núvol
│   ├── 03-Audio-Video/  → Serveis multimèdia
│   └── 04-BaseDeDades/  → Disseny i implementació BD
└── ansible/
    └── playbooks/       → Playbooks de configuració
```

---

## Índex de documentació

- [Proposta CPD](docs/01-CPD/ubicacion-fisica.md)
- [Infraestructura AWS](docs/02-AWS/arquitectura.md)
- [Serveis d'àudio i vídeo](docs/03-Audio-Video/icecast2.md)
- [Base de dades](docs/04-BaseDeDades/disseny-er.md)

---

## Dates clau

| Data | Esdeveniment |
|------|-------------|
| 28 de maig 2026 | Lliurament final |
| 29 de maig 2026 | Presentació i defensa |
