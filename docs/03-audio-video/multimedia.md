# Icecast2 - Streaming de Audio

**Máquina:** EC2-3 - Ubuntu 22.04 LTS  
**IP pública:** 100.31.147.184  
**Puerto:** 8000 TCP

---

## 1. Descripción del servicio

Icecast2 es un servidor de streaming de audio de código abierto que permite transmitir audio en tiempo real a múltiples clientes simultáneamente. Actúa como punto central de distribución: recibe el audio de un cliente emisor (source client) y lo redistribuye a todos los oyentes conectados.

Soporta los formatos MP3, OGG Vorbis y AAC, y dispone de un panel web integrado accesible desde el navegador para monitorizar el estado del servicio, los canales activos y el número de oyentes en tiempo real.

En este proyecto se utiliza para distribuir audio en directo dentro de la infraestructura de InnovateTech, accesible tanto desde navegadores web como desde clientes multimedia como VLC.

---

## 2. Requerimientos

- Descripción de la funcionalidad del servicio de audio.
- Instalación y configuración de un servidor de streaming de audio (Icecast2).
- Uso del formato de audio digital MP3.
- Configuración del cliente de acceso (ffmpeg como source client).
- Acceso al servicio via navegador web.
- Documentación de la instalación, configuración y pruebas del servicio.

## 3. Instalación

*Servidor: EC2-3 - Ubuntu 22.04 LTS - IP pública: 100.31.147.184 - Puerto 8000 TCP abierto en el Security Group de AWS*

**Paso 1 - Corregir dependencias rotas (necesario en este servidor):**

```bash
sudo dpkg --configure -a
sudo apt --fix-broken install -y
```

**Paso 2 - Actualizar e instalar Icecast2:**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install icecast2 -y
```

Durante la instalación el asistente pregunta:
- Hostname
- Source password
- Relay password
- Admin password

**Paso 3 - Activar e iniciar el servicio:**

```bash
sudo systemctl enable icecast2
sudo systemctl start icecast2
```

**Paso 4 - Verificar que funciona:**

```bash
sudo systemctl status icecast2
```
<img width="730" height="345" alt="unnamed" src="https://github.com/user-attachments/assets/1414b292-e400-498c-aa8e-32a72929282f" />


---

## 4. Configuración del servicio

El archivo de configuración principal es `/etc/icecast2/icecast.xml`:

```bash
sudo nano /etc/icecast2/icecast.xml
```

Parámetros modificados:

| Parámetro | Valor configurado | Descripción |
|---|---|---|
| `<hostname>` | 100.31.147.184 | IP pública del servidor EC2-3 en AWS |
| `<port>` | 8000 | Puerto de escucha. Abierto en el Security Group de AWS |
| `<source-password>` | [password] | Contraseña que usa ffmpeg para enviar audio al servidor |
| `<relay-password>` | [password] | Contraseña para servidores relay |
| `<admin-user>` | admin | Usuario del panel web de administración |
| `<admin-password>` | [password] | Contraseña del panel web de administración |
| `<mount-name>` | /stream | Ruta del canal de audio. URL: `http://IP:8000/stream` |

Después de modificar el archivo, reiniciar el servicio:

```bash
sudo systemctl restart icecast2
```
<img width="723" height="200" alt="unnamed" src="https://github.com/user-attachments/assets/cfd360ea-93a3-4aec-81fc-5fdac719ede9" />
<img width="735" height="306" alt="unnamed" src="https://github.com/user-attachments/assets/4c676f75-c7c2-434c-a325-33ae28d91605" />


---

## 5. Configuración del emisor de audio - ffmpeg

En lugar de usar BUTT desde un PC local, se emite audio directamente desde el servidor EC2-3 usando ffmpeg. Se han descargado 10 canciones en formato MP3 en el servidor que se emiten en bucle continuo a Icecast2 sin necesitar ningún PC externo ni micrófono.

**Paso 1 - Descargar canciones en el servidor:**

```bash
sudo mkdir -p /media/mu<img width="1778" height="515" alt="unnamed" src="https://github.com/user-attachments/assets/e7e53a55-1f96-4c0f-98ad-8e78a6566756" />
sica && cd /media/musica
wget https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3
```

**Paso 2 - Emitir las canciones en bucle a Icecast2 con ffmpeg:**

| Parámetro | Valor introducido | Por qué |
|---|---|---|
| Type | Icecast | Tipo de servidor al que nos conectamos |
| Name | InnovateTech Radio | Nombre descriptivo de la conexión |
| Address | 100.31.147.184 | IP pública del servidor EC2-3 en AWS |
| Port | 8000 | Puerto donde escucha Icecast2 |
| Password | [source-password] | La configurada en icecast.xml |
| Mount | stream | Canal configurado en icecast.xml (sin barra inicial) |
| Format | MP3 — 128 kbps — 44100 Hz | Formato y calidad del audio emitido |
<img width="1776" height="632" alt="unnamed" src="https://github.com/user-attachments/assets/9ead34e2-0a60-4c02-9cb5-4e8b0c5b07bf" />
<img width="1778" height="515" alt="unnamed" src="https://github.com/user-attachments/assets/6b12cd28-427f-4f4e-8c4c-ef90e08b3ee2" />


---

## 6. Acceso via navegador web

Una vez ffmpeg está emitiendo las canciones, el servicio es accesible desde cualquier navegador:

- **Panel de administración:** http://100.31.147.184:8000
- **URL directa del stream:** http://100.31.147.184:8000/stream

<img width="1852" height="973" alt="unnamed" src="https://github.com/user-attachments/assets/400b420b-7aca-488b-9778-f136ed7c467b" />


> 📸 **Capt![Uploading unnamed.png…]()

---

## 7. Formatos de audio utilizados

| Formato | Descripción técnica | Uso en el proyecto |
|---|---|---|
| **MP3** | MPEG-1 Audio Layer III. Compresión con pérdida. Bitrate: 128 kbps. Compatible con todos los navegadores sin plugins. | Formato principal del stream. Las canciones están en MP3 y se emiten via ffmpeg desde el servidor. |
| **OGG Vorbis** | Formato libre y abierto. Mejor calidad que MP3 al mismo bitrate. | Soportado por Icecast2. Alternativa libre al MP3. |
| **AAC** | Advanced Audio Coding. Sucesor del MP3 con mejor calidad a menor bitrate. | Soportado por Icecast2. Usado en streaming móvil. |

---

## 8. Incidencias y soluciones

| Incidencia | Solución aplicada |
|---|---|
| `E: Unable to locate package icecast2` | Ejecutar `sudo apt update` antes de instalar |
| `E: dpkg was interrupted` | Ejecutar `sudo dpkg --configure -a` y `sudo apt --fix-broken install -y` |
| `ERR_CONNECTION_TIMED_OUT` al acceder al puerto 8000 | Puerto 8000 no estaba abierto en el Security Group de AWS. Se abrió en Inbound Rules. |
| BUTT no funciona en los PCs del instituto (sin tarjeta de sonido) | Se sustituyó BUTT por ffmpeg ejecutado directamente en el servidor. |
| El canal desaparece al cerrar el terminal | Usar `nohup` y `&` para ejecutar ffmpeg en segundo plano |

---

## 9. Security Group - SG-MULTIMEDIA

| Dirección | Protocolo | Puerto | Origen |
|---|---|---|---|
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Salida | TCP/UDP | 514 | 10.0.0.0/16 |
