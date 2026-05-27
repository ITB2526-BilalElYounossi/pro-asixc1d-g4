# Jitsi Meet — Videoconferència

**Responsable:** Serghei  
**Màquina:** EC2-4 — Ubuntu 22.04 LTS  
**IP pública:** 3.219.249.6  
**Ports:** 80, 443 TCP — 10000 UDP

---

## 1. Descripció del servei

Jitsi Meet és una plataforma de videoconferència de codi obert basada en WebRTC que permet realitzar videotrucades directament des del navegador sense instal·lar programari addicional. S'ha desplegat en un servidor EC2 separat (EC2-4) usant Docker Compose, que és el mètode oficial recomanat.

S'utilitza a InnovateTech per a la comunicació interna entre departaments, reunions d'equip i formació corporativa.

---

## 2. Protocol WebRTC

WebRTC (Web Real-Time Communication) és un estàndard obert que permet comunicació d'àudio, vídeo i dades en temps real directament entre navegadors.

| Component | Funció |
|---|---|
| ICE / STUN / TURN | Troba la millor ruta de xarxa entre els clients i gestiona connexions darrere de NAT |
| DTLS | Xifra totes les connexions d'extrem a extrem |
| SRTP | Protocol segur per transmetre àudio i vídeo en temps real |
| SDP | Negocia els paràmetres i capacitats de cada client |

---

## 3. Requeriments

- Descripció del servei Jitsi Meet i del protocol WebRTC.
- Instal·lació i configuració de Jitsi Meet en servidor separat (EC2-4).
- Accés al servei des del navegador.
- Realització d'almenys una videotrucada funcional entre dos usuaris.

### Requisits mínims obligatoris

| | Requisit |
|---|---|
| ✅ | Servei de videoconferència instal·lat i operatiu a EC2-4 |
| ✅ | Accés funcional des del navegador web |
| ✅ | Almenys 1 videotrucada funcional entre dos usuaris |
| ✅ | Protocol WebRTC documentat |

---

## 4. Instal·lació amb Docker a EC2-4

*Servidor: EC2-4 — Ubuntu 22.04 LTS — Servidor separat del EC2-3. Ports: 80, 443 TCP i 10000 UDP oberts al Security Group.*

**Pas 1 — Actualitzar i instal·lar Docker:**

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://get.docker.com | sudo bash
sudo apt install docker-compose-plugin -y
sudo systemctl enable docker && sudo systemctl start docker
```

**Pas 2 — Descarregar i configurar Jitsi Meet:**

```bash
cd /opt && sudo git clone https://github.com/jitsi/docker-jitsi-meet
cd docker-jitsi-meet && sudo cp env.example .env
sudo bash gen-passwords.sh
mkdir -p ~/.jitsi-meet-cfg/{web,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}
```

**Pas 3 — Arrencar els contenidors:**

```bash
sudo docker-compose up -d
```

**Pas 4 — Verificar que tots els contenidors estan actius:**

```bash
sudo docker-compose ps
```

> 📸 **Captura 15** — Terminal mostrant els contenidors de Jitsi arrancant correctament  
> 📸 **Captura 16** — `docker-compose ps` mostrant els 4 contenidors en estat `Up`

---

## 5. Configuració de l'arxiu .env

L'arxiu `.env` conté la configuració principal. S'edita amb:

```bash
sudo nano /opt/docker-jitsi-meet/.env
```

| Paràmetre | Valor configurat | Descripció |
|---|---|---|
| `HTTP_PORT` | 8000 | Port HTTP d'accés a Jitsi Meet |
| `HTTPS_PORT` | 8443 | Port HTTPS d'accés a Jitsi Meet |
| `TZ` | UTC | Zona horària del servidor |
| `PUBLIC_URL` | https://3.219.249.6:8443 | URL pública completa amb port. Necessari per WebRTC. |
| `ENABLE_AUTH` | 0 | Sense autenticació (qualsevol pot crear sala) |
| `ENABLE_GUESTS` | 1 | Permet convidats sense compte |
| `JVB_PORT` | 10000 | Port UDP per a tràfic de vídeo. OBLIGATORI obrir-lo a AWS |
| `JITSI_IMAGE_VERSION` | stable-9646 | Versió estable de Jitsi. Necessari per evitar error SCRAM-SHA-1. |

> 📸 **Captura 17** — Arxiu `.env` amb els paràmetres de configuració de Jitsi Meet

---

## 6. Prova de videotrucada

Accedir des del navegador a `https://3.219.249.6:8443` (acceptar el certificat autosignat: *Avançat > Continuar*) i:

1. **Usuari 1:** crear sala amb nom `prueba-innovatetech` i entrar.
2. **Usuari 2:** obrir la mateixa URL en mode incògnit (`Ctrl+Shift+N`) o en un altre dispositiu.
3. Verificar que ambdós usuaris es veuen i s'escolten correctament.

> ⚠️ WebRTC requereix HTTPS per accedir a càmera i micròfon. Si el navegador bloqueja el micròfon, acceptar el certificat autosignat correctament.

> 📸 **Captura 18** — Jitsi Meet obert al navegador amb la pantalla d'inici  
> 📸 **Captura 19** — Videotrucada activa entre dos usuaris

---

## 7. Validació del servei

- Videotrucada funcional en temps real entre dos usuaris.
- Àudio i vídeo bidireccional correctes.
- Accés des del navegador sense instal·lar programari addicional.

---

## 8. Incidències i solucions

| Incidència | Solució aplicada |
|---|---|
| `E: Unable to locate package docker-compose-plugin` | Instal·lar Docker modern: `curl -fsSL https://get.docker.com \| sudo bash` |
| `unknown shorthand flag: d` / `docker: unknown command` | Versió antiga de Docker. Instal·lar versió moderna amb el script oficial de Docker. |
| `KeyError: ContainerConfig` en executar `docker-compose up` | Incompatibilitat entre docker-compose 1.29.2 i les imatges noves de Jitsi. Instal·lar Docker modern. |
| `SASLError SCRAM-SHA-1: not-authorized` als logs del jvb | Usar versió estable de Jitsi. Afegir a `.env`: `JITSI_IMAGE_VERSION=stable-9646`. Esborrar `~/.jitsi-meet-cfg` i regenerar amb `gen-passwords.sh` |
| *Ha estat desconnectat* en entrar a la reunió | `PUBLIC_URL` no incloïa el port. Canviat a: `PUBLIC_URL=https://3.219.249.6:8443` |
| `ERR_CONNECTION_REFUSED` en accedir a Jitsi | Ports 8000 i 8443 no estaven oberts al Security Group del EC2-4 a AWS. |

---

## 9. Security Group — SG-JITSI

| Direcció | Protocol | Port | Origen |
|---|---|---|---|
| Entrada | TCP | 80 | 0.0.0.0/0 |
| Entrada | TCP | 443 | 0.0.0.0/0 |
| Entrada | UDP | 10000 | 0.0.0.0/0 |
| Entrada | TCP | 4443 | 0.0.0.0/0 |
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Sortida | TCP/UDP | 514 | 10.0.0.0/16 |
