# Multimedia - Jellyfin (Streaming de Vídeo)

**Máquina:** EC2-3 - Ubuntu 22.04 LTS  
**IP pública:** 100.31.147.184  
**Puerto:** 8096 TCP

---

## 1. Descripción del servicio

Jellyfin es un servidor de streaming de vídeo de código abierto que permite organizar, gestionar y distribuir contenido audiovisual a través de una interfaz web intuitiva. Soporta transcodificación en tiempo real y es accesible desde cualquier navegador moderno sin plugins adicionales.

Se ha elegido Jellyfin porque el enunciado lo menciona explícitamente como opción válida, tiene interfaz web visual que facilita la demostración y soporta de forma nativa H.264 y MP4.

---

## 2. Requerimientos

- Descripción de la funcionalidad del servicio de vídeo.
- Instalación y configuración de Jellyfin como servidor de vídeo.
- Subida de al menos un vídeo accesible desde el servicio.
- Uso del formato MP4 y el codec H.264.
- Verificación de la reproducción desde navegador.
- Documentación de la instalación, configuración y pruebas.
---

## 3. Instalación

*Servidor: EC2-3 - Ubuntu 22.04 LTS - Puerto 8096 TCP abierto en el Security Group de AWS*

**Paso 1 - Instalar usando el script oficial de Jellyfin:**

```bash
curl -fsSL https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

> **Nota:** Si el script falla por espacio insuficiente en `/tmp`, ampliar primero:
> ```bash
> sudo mount -o remount,size=4G /tmp
> ```

**Paso 2 — Activar e iniciar:**

```bash
sudo systemctl enable jellyfin
sudo systemctl start jellyfin
```

<img width="734" height="476" alt="unnamed" src="https://github.com/user-attachments/assets/514273f0-aa61-4e6a-b34c-2b00fc0331b4" />
<img width="734" height="476" alt="unnamed" src="https://github.com/user-attachments/assets/d8f4b2b0-6b9c-44ac-a706-979eb81ed230" />

---

## 4. Configuración del servicio

### Paso 1 — Crear directorio y descargar vídeo de prueba

```bash
sudo mkdir -p /media/videos
sudo chown jellyfin:jellyfin /media/videos
sudo wget -O /media/videos/video_prueba.mp4 "https://www.w3schools.com/html/mov_bbb.mp4"
sudo chown jellyfin:jellyfin /media/videos/video_prueba.mp4
```

Verificación de que el archivo se descargó correctamente:

```bash
ls -la /media/videos/
```

### Paso 2 — Configuración inicial via wizard web

Acceder a `http://100.31.147.184:8096` y seguir el wizard:

| Paso | Acción |
|---|---|
| 1. Bienvenida | Hacer clic en Siguiente |
| 2. Idioma | Seleccionar el idioma y hacer clic en Siguiente |
| 3. Usuario administrador | Crear usuario: `jellyfin` con contraseña segura |
| 4. Biblioteca de medios | Hacer clic en Añadir biblioteca de medios |
| 5. Tipo de biblioteca | Seleccionar *Vídeos y fotos personales* (acepta cualquier MP4) |
| 6. Directorio | Añadir la ruta: `/media/videos` |
| 7. Finalizar | Hacer clic en Siguiente y Terminar |

<img width="904" height="542" alt="unnamed" src="https://github.com/user-attachments/assets/fb304ec5-356b-41c8-8b15-c57c0eab0ec6" />

> <img width="1082" height="588" alt="unnamed" src="https://github.com/user-attachments/assets/71cd22c3-1dc4-4336-8f5b-b24eee1f64ee" />


---

## 5. Formatos y codecs utilizados

| Elemento | Valor | Descripción |
|---|---|---|
| Contenedor | MP4 (`.mp4`) | Formato estándar universal para distribución de vídeo en web |
| Codec vídeo | H.264 (AVC) | Codec más compatible. Soportado por todos los navegadores modernos sin plugins |
| Codec audio | AAC | Audio comprimido de alta calidad incluido en el contenedor MP4 |
| Protocolo | HTTP Progressive | Jellyfin sirve el vídeo via HTTP. El navegador lo reproduce progresivamente |

---

## 6. Reproducción desde el navegador

Una vez el vídeo está en la biblioteca, se accede desde cualquier navegador en `http://100.31.147.184:8096`. Jellyfin proporciona un player web integrado con controles de reproducción, volumen y calidad.

> 📸 **Captura 13** — Vídeo reproduciéndose en el navegador desde Jellyfin (player con controles visible)  
> 📸 **Captura 14** — Información técnica del vídeo mostrando codec H.264 y formato MP4

---

## 7. Validación del servicio

- Reproducción correcta del vídeo desde el navegador web.
- Acceso funcional desde diferentes dispositivos.
- El vídeo muestra la información correcta de codec (H.264) y formato (MP4).

---

## 8. Incidencias y soluciones

| Incidencia | Solución aplicada |
|---|---|
| `Insufficient free space for /tmp` durante instalación | Ejecutar: `sudo mount -o remount,size=4G /tmp` antes de instalar |
| El vídeo no aparece en la biblioteca *Películas* | Cambiar tipo de biblioteca de *Películas* a *Vídeos y fotos personales* (más permisivo) |
| `Permission denied` al hacer scp del vídeo | Descargar el vídeo directamente en el servidor con `wget` en lugar de `scp` |
| La biblioteca no escanea el vídeo nuevo | Hacer clic en *Escanear todas las mediatecas* desde el panel de administración de Jellyfin |

---

## 9. Security Group — SG-MULTIMEDIA

| Dirección | Protocolo | Puerto | Origen |
|---|---|---|---|
| Entrada | TCP | 8096 | 0.0.0.0/0 |
| Entrada | TCP | 8000 | 0.0.0.0/0 |
| Entrada | TCP | 22 | 0.0.0.0/0 |
| Salida | TCP/UDP | 514 | 10.0.0.0/16 |

---

## 10. Comprobaciones de Ancho de Banda

### 10.1 Objetivo

Garantizar que la infraestructura desplegada es capaz de soportar los servicios de audio, vídeo y videoconferencia sin degradación del servicio.

### 10.2 Requisitos mínimos obligatorios

| | Requisito |
|---|---|
| ✅ | Realización de al menos 2 pruebas de ancho de banda |
| ✅ | Mostrar download, upload y latencia en cada prueba |
| ✅ | Relacionar los resultados con los tres servicios multimedia |
| ✅ | Clasificar el sistema como Aceptable o No aceptable |
| ✅ | Incluir evidencias (capturas) y conclusión técnica |

### 10.3 Herramientas utilizadas

```bash
sudo apt install iperf3 -y
pip3 install speedtest-cli --break-system-packages
```

| Herramienta | Función |
|---|---|
| `speedtest-cli` | Mide velocidad de bajada, subida y latencia contra servidores de internet |
| `iperf3` | Mide rendimiento de red entre dos equipos de la infraestructura |
| `ping` | Mide la latencia hacia un destino específico |

### 10.4 Prueba 1 — Sin servicios activos (línea base)

*Condiciones: ningún servicio multimedia activo. Medida de referencia.*

```bash
speedtest-cli --simple
ping -c 20 8.8.8.8
```

> 📸 **Captura 20** — Resultado de `speedtest-cli` sin servicios activos mostrando download, upload y latencia

| Medida | Valor obtenido |
|---|---|
| Download | 47 Mbps |
| Upload | 8,5 Mbps |
| Latencia (ping) | 1,97 ms |

### 10.5 Prueba 2 — Con todos los servicios activos

*Condiciones: Icecast2 emitiendo + Jellyfin reproduciendo vídeo + Jitsi Meet en videollamada activa.*

```bash
speedtest-cli --simple
ping -c 20 8.8.8.8
```

> 📸 **Captura 21** — Los tres servicios activos simultáneamente en pantalla  
> 📸 **Captura 22** — `speedtest-cli` con servicios activos mostrando download, upload y latencia

| Medida | Valor obtenido |
|---|---|
| Download | 191 Mbps |
| Upload | 6,7 Mbps |
| Latencia (ping) | 1,95 ms |

### 10.6 Análisis de resultados

| Servicio | Consumo estimado de red | Tipo de tráfico |
|---|---|---|
| Icecast2 (MP3 128kbps) | ~0,13 Mbps por oyente | Upload desde el servidor |
| Jellyfin (H.264 720p) | ~2–4 Mbps por stream | Download hacia el cliente |
| Jitsi Meet (720p, 2 usuarios) | ~1,5–3 Mbps por participante | Upload y download bidireccional |
| **Total estimado** | **~4–8 Mbps** | Combinado |

### 10.7 Clasificación del sistema

✅ **ACEPTABLE** — La infraestructura soporta los tres servicios sin degradación significativa. La latencia se mantiene por debajo de los 2 ms en ambas pruebas y la velocidad de descarga es muy superior al consumo estimado de los servicios.

### 10.8 Propuestas de optimización

| Propuesta | Beneficio esperado |
|---|---|
| Aumentar tipo de instancia EC2 (`t3.micro` → `t3.medium`) | Más CPU y RAM para procesar streams simultáneos |
| Reducir bitrate Icecast2 de 128kbps a 96kbps | Menor consumo de ancho de banda manteniendo calidad aceptable |
| Separar Jellyfin en su propio EC2 | Cada servicio tiene recursos dedicados sin competencia |
| Configurar QoS en la VPC de AWS | Priorizar tráfico de videoconferencia sobre el resto |
