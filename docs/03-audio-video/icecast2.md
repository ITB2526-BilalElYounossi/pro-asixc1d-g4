# Icecast2 — Streaming d'Àudio

**Responsable:** Serghei  
**Màquina:** EC2-3 — Ubuntu 22.04 LTS  
**IP pública:** 100.31.147.184  
**Port:** 8000 TCP

---

## 1. Descripció del servei

Icecast2 és un servidor de streaming d'àudio de codi obert que permet transmetre àudio en temps real a múltiples clients simultàniament. Actua com a punt central de distribució: rep l'àudio d'un client emissor (source client) i el redistribueix a tots els oients connectats.

Suporta els formats MP3, OGG Vorbis i AAC, i disposa d'un panell web integrat accessible des del navegador per monitoritzar l'estat del servei, els canals actius i el nombre d'oients en temps real.

En aquest projecte s'utilitza per distribuir àudio en directe dins la infraestructura d'InnovateTech, accessible tant des de navegadors web com des de clients multimèdia com VLC.

---

## 2. Requeriments

- Descripció de la funcionalitat del servei d'àudio.
- Instal·lació i configuració d'un servidor de streaming d'àudio (Icecast2).
- Ús del format d'àudio digital MP3.
- Configuració del client d'accés (ffmpeg com a source client).
- Accés al servei via navegador web.
- Documentació de la instal·lació, configuració i proves del servei.

### Requisits mínims obligatoris

| | Requisit |
|---|---|
| ✅ | Servidor d'àudio instal·lat i operatiu |
| ✅ | Almenys 1 canal d'àudio funcional |
| ✅ | Reproducció correcta des d'un client |
| ✅ | Accés funcional via navegador |
| ✅ | Ús d'almenys 1 format d'àudio digital |
| ✅ | Evidència de funcionament mitjançant captures |

---

## 3. Instal·lació

*Servidor: EC2-3 — Ubuntu 22.04 LTS — IP pública: 100.31.147.184 — Port 8000 TCP obert al Security Group d'AWS*

**Pas 1 — Corregir dependències trencades (necessari en aquest servidor):**

```bash
sudo dpkg --configure -a
sudo apt --fix-broken install -y
```

**Pas 2 — Actualitzar i instal·lar Icecast2:**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install icecast2 -y
```

Durant la instal·lació l'assistent pregunta:
- Hostname
- Source password
- Relay password
- Admin password

**Pas 3 — Activar i iniciar el servei:**

```bash
sudo systemctl enable icecast2
sudo systemctl start icecast2
```

**Pas 4 — Verificar que funciona:**

```bash
sudo systemctl status icecast2
```

> 📸 **Captura 1** — Instal·lació d'Icecast2 completada en terminal  
> 📸 **Captura 2** — `systemctl status icecast2` mostrant estat `active (running)` en verd

---

## 4. Configuració del servei

L'arxiu de configuració principal és `/etc/icecast2/icecast.xml`:

```bash
sudo nano /etc/icecast2/icecast.xml
```

Paràmetres modificats:

| Paràmetre | Valor configurat | Descripció |
|---|---|---|
| `<hostname>` | 100.31.147.184 | IP pública del servidor EC2-3 a AWS |
| `<port>` | 8000 | Port d'escolta. Obert al Security Group d'AWS |
| `<source-password>` | [password] | Contrasenya que usa ffmpeg per enviar àudio al servidor |
| `<relay-password>` | [password] | Contrasenya per a servidors relay |
| `<admin-user>` | admin | Usuari del panell web d'administració |
| `<admin-password>` | [password] | Contrasenya del panell web d'administració |
| `<mount-name>` | /stream | Ruta del canal d'àudio. URL: `http://IP:8000/stream` |

Després de modificar l'arxiu, reiniciar el servei:

```bash
sudo systemctl restart icecast2
```

> 📸 **Captura 3** — Arxiu `/etc/icecast2/icecast.xml` amb els paràmetres hostname i passwords configurats

---

## 5. Configuració de l'emissor d'àudio — ffmpeg

En lloc d'usar BUTT des d'un PC local, s'emet àudio directament des del servidor EC2-3 usant ffmpeg. S'han descarregat 10 cançons en format MP3 al servidor que s'emeten en bucle continu a Icecast2 sense necessitar cap PC extern ni micròfon.

**Pas 1 — Descarregar cançons al servidor:**

```bash
sudo mkdir -p /media/musica && cd /media/musica
wget https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3
```

**Pas 2 — Emetre les cançons en bucle a Icecast2 amb ffmpeg:**

| Paràmetre | Valor introduït | Per què |
|---|---|---|
| Type | Icecast | Tipus de servidor al qual ens connectem |
| Name | InnovateTech Radio | Nom descriptiu de la connexió |
| Address | 100.31.147.184 | IP pública del servidor EC2-3 a AWS |
| Port | 8000 | Port on escolta Icecast2 |
| Password | [source-password] | La configurada a icecast.xml |
| Mount | stream | Canal configurat a icecast.xml (sense barra inicial) |
| Format | MP3 — 128 kbps — 44100 Hz | Format i qualitat de l'àudio emès |

> 📸 **Captura 4** — Panell web d'Icecast2 amb Mountpoint `/stream` actiu (ffmpeg emetent cançons)  
> 📸 **Captura 5** — Navegador o VLC reproduint el stream d'àudio amb les cançons

---

## 6. Accés via navegador web

Una vegada ffmpeg està emetent les cançons, el servei és accessible des de qualsevol navegador:

- **Panell d'administració:** http://100.31.147.184:8000
- **URL directa del stream:** http://100.31.147.184:8000/stream

> 📸 **Captura 6** — Panell web d'Icecast2 mostrant Mountpoint `/stream` actiu amb bitrate 128 i listeners  
> 📸 **Captura 7** — Navegador reproduint el stream d'àudio  
> 📸 **Captura 8** — Panell Icecast2 amb listeners connectats

---

## 7. Formats d'àudio utilitzats

| Format | Descripció tècnica | Ús en el projecte |
|---|---|---|
| **MP3** | MPEG-1 Audio Layer III. Compressió amb pèrdua. Bitrate: 128 kbps. Compatible amb tots els navegadors sense plugins. | Format principal del stream. Les cançons estan en MP3 i s'emeten via ffmpeg des del servidor. |
| **OGG Vorbis** | Format lliure i obert. Millor qualitat que MP3 al mateix bitrate. | Suportat per Icecast2. Alternativa lliure al MP3. |
| **AAC** | Advanced Audio Coding. Successor del MP3 amb millor qualitat a menor bitrate. | Suportat per Icecast2. Usat en streaming mòbil. |

---

## 8. Incidències i solucions

| Incidència | Solució aplicada |
|---|---|
| `E: Unable to locate package icecast2` | Executar `sudo apt update` abans d'instal·lar |
| `E: dpkg was interrupted` | Executar `sudo dpkg --configure -a` i `sudo apt --fix-broken install -y` |
| `ERR_CONNECTION_TIMED_OUT` al accedir al port 8000 | Port 8000 no estava obert al Security Group d'AWS. S'ha obert a Inbound Rules. |
| BUTT no funciona als PCs de l'institut (sense targeta de so) | S'ha substituït BUTT per ffmpeg executat directament al servidor. |
| El canal desapareix en tancar el terminal | Usar `nohup` i `&` per executar ffmpeg en segon pla |

---

## 9. Security Group — SG-MULTIMEDIA

| Direcció | Protocol | Port | Origen |
|---|---|---|---|
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Sortida | TCP/UDP | 514 | 10.0.0.0/16 |
