# Projecte Transversal ASIXc1 — InnovateTech CPD

> Disseny i implementació d'un Centre de Processament de Dades (CPD) al núvol AWS per a l'empresa **InnovateTech**, incloent serveis multimèdia, base de dades, directori actiu i gestió centralitzada de logs.

**Grup:** TIBS (Taylor · Izan · Bilal · Serghei)  
**Cicle:** CFGS Administració de Sistemes Informàtics en Xarxa  
**Centre:** Institut Tecnològic de Barcelona  
**Curs:** 2025–2026  
**Lliurament:** 28 de maig de 2026

---

## Descripció del projecte

InnovateTech és una empresa dedicada a la provisió de serveis tecnològics que necessita modernitzar la seva capacitat operativa. Aquest projecte dissenya i implementa un CPD eficient al núvol AWS que integra:

- Distribució de continguts d'àudio i vídeo en streaming (ràdio en directe + vídeo sota demanda)
- Canal de vídeo en directe (live streaming)
- Sistema de videoconferències per a comunicació interna
- Base de dades per gestionar personal, comunicacions i activitat
- Directori Actiu per centralitzar la gestió d'usuaris
- Servidor web (intranet) autenticat contra el Directori Actiu
- Gestió centralitzada de logs de tots els servidors via rsyslog + EFS
- Configuració automatitzada amb Ansible

---

## Arquitectura AWS

![Arquitectura AWS InnovateTech](https://github.com/user-attachments/assets/4db7655e-f0b4-493a-bdd5-90f19d86825d)

---

## Equip i responsabilitats

| Membre | Rol | Màquines |
|--------|-----|----------|
| **Bilal** | Infraestructura AWS + Logs + Ansible | logs-server-private |
| **Izan** | Base de dades + Ansible | mariadb + ansible-controller |
| **Taylor** | Web + SFTP + Directori Actiu | web-sftp + samba-ad |
| **Serghei** | Multimèdia + Videoconferències | multimedia + jitsi-meet |

---

## Màquines i serveis

| Màquina | IP Pública | IP Privada | Serveis | Subnet |
|---------|-----------|------------|---------|--------|
| web-sftp | 52.1.67.249 | 10.0.5.140 | NGINX + PHP + SFTP + Portal web | Pública |
| multimedia | 32.198.236.17 | 10.0.8.36 | Jellyfin + Icecast2 + Live stream | Pública |
| jitsi-meet | 3.219.249.6 | 10.0.14.189 | Jitsi Meet (Docker) | Pública |
| ansible-controller | 32.193.193.146 | 10.0.7.201 | Ansible | Pública |
| samba-ad | — | 10.0.141.9 | Samba AD (domini BTIS) | Privada |
| mariadb | — | 10.0.142.205 | MariaDB 10.11 | Privada |
| logs-server-private | — | 10.0.133.107 | Rsyslog + EFS | Privada |

---

## Stack tecnològic

| Component | Tecnologia | Funció |
|-----------|-----------|--------|
| Cloud | AWS (VPC, EC2, ALB, EFS, S3) | Infraestructura |
| SO | Ubuntu 24.04 LTS | Totes les instàncies |
| Directori Actiu | Samba AD | Gestió d'usuaris (domini BTIS) |
| Servidor web | NGINX + PHP 8.3 | Intranet InnovateTech |
| Transferència fitxers | SFTP + AD auth | Accés fitxers per usuaris |
| Vídeo sota demanda | Jellyfin | Distribució vídeo |
| Vídeo en directe | ffmpeg + HLS + NGINX | Canal live streaming |
| Ràdio en directe | Icecast2 + ffmpeg | Retransmissió RNE Radio 1 |
| Videoconferències | Jitsi Meet (Docker) | Comunicació interna |
| Base de dades | MariaDB 10.11 | Gestió dades empresa |
| Automatització | Ansible | Configuració servidors |
| Logs centralitzats | Rsyslog + EFS | Monitoratge en temps real |
| Control de versions | Git + GitHub | Documentació i codi |

---

## Seguretat

| Capa | Mesures aplicades |
|------|------------------|
| Xarxa | Subnet privada per BD, AD i logs. Security Groups per servei |
| Accés SSH | Clau pública/privada, usuari `adminitb` dedicat, sense contrasenya |
| Web | HTTPS, X-Frame-Options, X-XSS-Protection, X-Content-Type-Options |
| BD | Rols diferenciats, bind-address privat, prepared statements, triggers |
| Logs | Rsyslog centralitzat → EFS, logrotate diari, dashboard admin |
| AWS | ALB davant web, NAT Gateway per subnet privada |

---

## Estructura del repositori

```
Proyecto-Transversal-TIBS/
├── README.md
├── docs/
│   ├── 01-cpd/
│   │   └── cpd.md                  → Proposta CPD físic (Bilal)
│   ├── 02-aws/
│   │   ├── arquitectura.md         → VPC, subnets, EC2, SG (Bilal)
│   │   ├── ansible.md              → Playbooks, inventari (Bilal)
│   │   ├── logs.md                 → Rsyslog + EFS (Bilal)
│   │   ├── samba-ad.md             → Directori Actiu (Taylor)
│   │   └── web-sftp.md             → NGINX, PHP, SFTP (Taylor)
│   ├── 03-audio-video/
│   │   ├── icecast-radio.md        → Ràdio en directe (Serghei)
│   │   ├── jellyfin.md             → Vídeo sota demanda (Serghei)
│   │   ├── live-streaming.md       → Canal vídeo en directe (Serghei)
│   │   └── jitsi.md                → Videoconferències (Serghei)
│   └── 04-BaseDeDades/
│       ├── disseny-er.md           → Model E/R (Izan)
│       ├── usuaris-rols.md         → Rols i permisos (Izan)
│       └── triggers-events.md      → Triggers i backups (Izan)
├── infrastructure/
│   └── aws/
│       └── vpc.md
├── media/
│   ├── aws/                        → Captures infraestructura
│   └── cpd/                        → Imatges CPD físic
├── tests/
│   └── test-plan.md
└── infrastructure/aws/vpc.md
```

---

## Índex de documentació

| Document | Responsable | Estat |
|----------|-------------|-------|
| [Proposta CPD](docs/01-cpd/cpd.md) | Bilal | ✅ |
| [Infraestructura AWS](docs/02-aws/arquitectura.md) | Bilal | ✅ |
| [Ansible](docs/02-aws/ansible.md) | Bilal | ✅ |
| [Logs centralitzats](docs/02-aws/logs.md) | Bilal | ✅ |
| [Samba AD](docs/02-aws/samba-ad.md) | Taylor | 🔄 |
| [Web + SFTP](docs/02-aws/web-sftp.md) | Taylor | 🔄 |
| [Ràdio Icecast2](docs/03-audio-video/icecast-radio.md) | Serghei | 🔄 |
| [Jellyfin VOD](docs/03-audio-video/jellyfin.md) | Serghei | 🔄 |
| [Live Streaming](docs/03-audio-video/live-streaming.md) | Serghei | 🔄 |
| [Jitsi Meet](docs/03-audio-video/jitsi.md) | Serghei | 🔄 |
| [Base de dades](docs/04-BaseDeDades/disseny-er.md) | Izan | 🔄 |
| [Usuaris i rols](docs/04-BaseDeDades/usuaris-rols.md) | Izan | 🔄 |
| [Triggers i events](docs/04-BaseDeDades/triggers-events.md) | Izan | 🔄 |

---

## Proves de funcionament

| Prova | Resultat |
|-------|---------|
| SSH amb clau pública a totes les màquines | ✅ |
| Connectivitat entre subnets | ✅ |
| NAT Gateway subnet privada | ✅ |
| Logs centralitzats (7 màquines → EFS) | ✅ |
| Dashboard logs al portal web | ✅ |
| Ansible ping totes les màquines | ✅ |
| Ansible playbooks (logs_baseline, logs_clients, mariadb) | ✅ |
| Portal web amb autenticació Samba AD | ✅ |
| HTTPS + certificat SSL | ✅ |
| Ràdio en directe (RNE Radio 1 via Icecast2) | ✅ |
| Jellyfin vídeo sota demanda | ✅ |
| Jitsi Meet videoconferència | ✅ |
| MariaDB rols i permisos | ✅ |
| SFTP autenticat contra AD | ✅ |
| Live streaming vídeo en directe | 🔄 En curs |
