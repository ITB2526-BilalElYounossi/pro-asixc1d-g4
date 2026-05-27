# Multimèdia — Jellyfin (Streaming de Vídeo)

**Responsable:** Serghei  
**Màquina:** EC2-3 — Ubuntu 22.04 LTS  
**IP pública:** 100.31.147.184  
**Port:** 8096 TCP

---

## 1. Descripció del servei

Jellyfin és un servidor de streaming de vídeo de codi obert que permet organitzar, gestionar i distribuir contingut audiovisual a través d'una interfície web intuïtiva. Suporta transcodificació en temps real i és accessible des de qualsevol navegador modern sense plugins addicionals.

S'ha escollit Jellyfin perquè l'enunciat el menciona explícitament com a opció vàlida, té interfície web visual que facilita la demostració i suporta de forma nativa H.264 i MP4.

---

## 2. Requeriments

- Descripció de la funcionalitat del servei de vídeo.
- Instal·lació i configuració de Jellyfin com a servidor de vídeo.
- Pujada d'almenys un vídeo accessible des del servei.
- Ús del format MP4 i el codec H.264.
- Verificació de la reproducció des del navegador.
- Documentació de la instal·lació, configuració i proves.

### Requisits mínims obligatoris

| | Requisit |
|---|---|
| ✅ | Servidor de vídeo instal·lat i operatiu |
| ✅ | Almenys 1 vídeo accessible en streaming |
| ✅ | Reproducció funcional des d'un navegador |
| ✅ | Ús d'almenys 1 format de vídeo digital (MP4 amb H.264) |
| ✅ | Evidències documentades |

---

## 3. Instal·lació

*Servidor: EC2-3 — Ubuntu 22.04 LTS — Port 8096 TCP obert al Security Group d'AWS*

**Pas 1 — Instal·lar usant el script oficial de Jellyfin:**

```bash
curl -fsSL https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

> **Nota:** Si el script falla per espai insuficient a `/tmp`, ampliar primer:
> ```bash
> sudo mount -o remount,size=4G /tmp
> ```

**Pas 2 — Activar i iniciar:**

```bash
sudo systemctl enable jellyfin
sudo systemctl start jellyfin
```

> 📸 **Captura 9** — Terminal mostrant la instal·lació completada de Jellyfin  
> 📸 **Captura 10** — `systemctl status jellyfin` mostrant estat `active (running)`

---

## 4. Configuració del servei

### Pas 1 — Crear directori i descarregar vídeo de prova

```bash
sudo mkdir -p /media/videos
sudo chown jellyfin:jellyfin /media/videos
sudo wget -O /media/videos/video_prueba.mp4 "https://www.w3schools.com/html/mov_bbb.mp4"
sudo chown jellyfin:jellyfin /media/videos/video_prueba.mp4
```

Verificació que l'arxiu s'ha descarregat correctament:

```bash
ls -la /media/videos/
```

### Pas 2 — Configuració inicial via wizard web

Accedir a `http://100.31.147.184:8096` i seguir el wizard:

| Pas | Acció |
|---|---|
| 1. Benvinguda | Fer clic a Següent |
| 2. Idioma | Seleccionar l'idioma i fer clic a Següent |
| 3. Usuari administrador | Crear usuari: `jellyfin` amb contrasenya segura |
| 4. Biblioteca de medis | Fer clic a Afegir biblioteca de medis |
| 5. Tipus de biblioteca | Seleccionar *Vídeos i fotos personals* (accepta qualsevol MP4) |
| 6. Directori | Afegir la ruta: `/media/videos` |
| 7. Finalitzar | Fer clic a Següent i Acabar |

> 📸 **Captura 11** — Wizard de configuració inicial de Jellyfin amb la biblioteca configurada  
> 📸 **Captura 12** — Biblioteca creada amb el vídeo visible al panell de Jellyfin

---

## 5. Formats i codecs utilitzats

| Element | Valor | Descripció |
|---|---|---|
| Contenidor | MP4 (`.mp4`) | Format estàndard universal per a distribució de vídeo al web |
| Codec vídeo | H.264 (AVC) | Codec més compatible. Suportat per tots els navegadors moderns sense plugins |
| Codec àudio | AAC | Àudio comprimit d'alta qualitat inclòs al contenidor MP4 |
| Protocol | HTTP Progressive | Jellyfin serveix el vídeo via HTTP. El navegador el reprodueix progressivament |

---

## 6. Reproducció des del navegador

Una vegada el vídeo és a la biblioteca, s'accedeix des de qualsevol navegador a `http://100.31.147.184:8096`. Jellyfin proporciona un player web integrat amb controls de reproducció, volum i qualitat.

> 📸 **Captura 13** — Vídeo reproduint-se al navegador des de Jellyfin (player amb controls visible)  
> 📸 **Captura 14** — Informació tècnica del vídeo mostrant codec H.264 i format MP4

---

## 7. Validació del servei

- Reproducció correcta del vídeo des del navegador web.
- Accés funcional des de diferents dispositius.
- El vídeo mostra la informació correcta de codec (H.264) i format (MP4).

---

## 8. Incidències i solucions

| Incidència | Solució aplicada |
|---|---|
| `Insufficient free space for /tmp` durant instal·lació | Executar: `sudo mount -o remount,size=4G /tmp` abans d'instal·lar |
| El vídeo no apareix a la biblioteca *Pel·lícules* | Canviar tipus de biblioteca de *Pel·lícules* a *Vídeos i fotos personals* (més permissiu) |
| `Permission denied` en fer scp del vídeo | Descarregar el vídeo directament al servidor amb `wget` en lloc de `scp` |
| La biblioteca no escaneja el vídeo nou | Fer clic a *Escaneja totes les mediatecques* des del panell d'administració de Jellyfin |

---

## 9. Security Group — SG-MULTIMEDIA

| Direcció | Protocol | Port | Origen |
|---|---|---|---|
| Entrada | TCP | 8096 | 0.0.0.0/0 |
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Sortida | TCP/UDP | 514 | 10.0.0.0/16 |

---

## 10. Comprovacions d'Ample de Banda

### 10.1 Objectiu

Garantir que la infraestructura desplegada és capaç de suportar els serveis d'àudio, vídeo i videoconferència sense degradació del servei.

### 10.2 Requisits mínims obligatoris

| | Requisit |
|---|---|
| ✅ | Realització d'almenys 2 proves d'ample de banda |
| ✅ | Mostrar download, upload i latència a cada prova |
| ✅ | Relacionar els resultats amb els tres serveis multimèdia |
| ✅ | Classificar el sistema com Acceptable o No acceptable |
| ✅ | Incloure evidències (captures) i conclusió tècnica |

### 10.3 Eines utilitzades

```bash
sudo apt install iperf3 -y
pip3 install speedtest-cli --break-system-packages
```

| Eina | Funció |
|---|---|
| `speedtest-cli` | Mesura velocitat de baixada, pujada i latència contra servidors d'internet |
| `iperf3` | Mesura rendiment de xarxa entre dos equips de la infraestructura |
| `ping` | Mesura la latència cap a un destí específic |

### 10.4 Prova 1 — Sense serveis actius (línia base)

*Condicions: cap servei multimèdia actiu. Mesura de referència.*

```bash
speedtest-cli --simple
ping -c 20 8.8.8.8
```

> 📸 **Captura 20** — Resultat de `speedtest-cli` sense serveis actius mostrant download, upload i latència

| Mesura | Valor obtingut |
|---|---|
| Download | 47 Mbps |
| Upload | 8,5 Mbps |
| Latència (ping) | 1,97 ms |

### 10.5 Prova 2 — Amb tots els serveis actius

*Condicions: Icecast2 emetent + Jellyfin reproduint vídeo + Jitsi Meet en videotrucada activa.*

```bash
speedtest-cli --simple
ping -c 20 8.8.8.8
```

> 📸 **Captura 21** — Els tres serveis actius simultàniament en pantalla  
> 📸 **Captura 22** — `speedtest-cli` amb serveis actius mostrant download, upload i latència

| Mesura | Valor obtingut |
|---|---|
| Download | 191 Mbps |
| Upload | 6,7 Mbps |
| Latència (ping) | 1,95 ms |

### 10.6 Anàlisi de resultats

| Servei | Consum estimat de xarxa | Tipus de tràfic |
|---|---|---|
| Icecast2 (MP3 128kbps) | ~0,13 Mbps per oient | Upload des del servidor |
| Jellyfin (H.264 720p) | ~2–4 Mbps per stream | Download cap al client |
| Jitsi Meet (720p, 2 usuaris) | ~1,5–3 Mbps per participant | Upload i download bidireccional |
| **Total estimat** | **~4–8 Mbps** | Combinat |

### 10.7 Classificació del sistema

✅ **ACCEPTABLE** — La infraestructura suporta els tres serveis sense degradació significativa. La latència es manté per sota dels 2 ms en ambdues proves i la velocitat de descàrrega és molt superior al consum estimat dels serveis.

### 10.8 Propostes d'optimització

| Proposta | Benefici esperat |
|---|---|
| Augmentar tipus d'instància EC2 (`t3.micro` → `t3.medium`) | Més CPU i RAM per processar streams simultanis |
| Reduir bitrate Icecast2 de 128kbps a 96kbps | Menor consum d'ample de banda mantenint qualitat acceptable |
| Separar Jellyfin en el seu propi EC2 | Cada servei té recursos dedicats sense competència |
| Configurar QoS a la VPC d'AWS | Prioritzar tràfic de videoconferència sobre la resta |
