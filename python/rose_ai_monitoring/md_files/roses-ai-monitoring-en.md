# 🌹 RoseAI Monitor — Intelligent Rose Crop Monitoring

**Edge-AI computer vision system for rose growth and phenological stage monitoring in the field, using 10BASE-T1S MultiDrop bus and Hailo-8L acceleration.**

Hardware: Hellbender VineCam System (1 Hub + 4 Cameras).

---

## 🎯 Project Objective

Automate daily rose crop monitoring using the Hellbender VineCam system — 11.9 MP cameras distributed across the field, connected via 10BASE-T1S daisy-chain bus, with bloom stage inference (bud vs. open flower) running on a Raspberry Pi 5 server with Hailo-8L accelerator.

**Key metrics:**
- Rose count by phenological stage (`rose_small`, `rose_large`)
- Bloom ratio (`large / total`)
- Harvest alert when `rose_large > threshold`
- Optional: disease detection (powdery mildew, botrytis, black spot)

---

## 🏗️ System Architecture

*Implementación técnica — ver anexo técnico.*

**Kit base:** 1 Hub + 4 Cámaras = $1,049–$1,199 USD. Pre-order Spring 2026.

![Cayambe Greenhouse AI Monitoring](./images/cayambe_greenhouse_ai_monitoring.png)

*Render conceptual: Invernadero inteligente de rosas con sistema VineCam + Hailo-8L en Cayambe, Ecuador.*

### Daily Data Flow

*Implementación técnica — ver anexo técnico.*

### VineCam Hub Internal Components

| Componente | Especificación |
|---|---|
| **CPU** | Raspberry Pi CM4 (Quad Cortex-A72 @ 1.5 GHz) |
| **RAM / Storage** | 2 GB / 32 GB eMMC |
| **Bus MCU** | RP2350 Dual Cortex-M33 @ 150 MHz — dedicado a 10BASE-T1S |
| **AI Accelerator** | Hailo-8 (opcional, requiere compra adicional) |
| **Camera ports** | 4× RJ45 10BASE-T1S (⚠️ NON-standard pinout — do not connect to other devices) |
| **External network** | 1× Gigabit Ethernet RJ45 + WiFi 5 (802.11ac) + BLE |
| **Sensores** | IMU (acelerómetro + giroscopio) + ambiental (Temperatura, Presión, Humedad) |
| **Power** | 48V DC (conector Molex 6 pines) |
| **OS / API** | Linux + REST API + Web GUI de configuración |
| **Weight / Dimensions** | 390 g / 121 × 144 × 43 mm (VESA mount 100×100) |

### VineCam Unit Internal Components

| Componente | Especificación |
|---|---|
| **Sensor** | 11.9 MP, autofocus, rolling shutter |
| **FOV** | Opción 66°H×41°V o 102°H×67°V |
| **Iluminación** | 2× programmable LEDs (control remoto desde el Hub) |
| **Conectividad** | 2× RJ45 10BASE-T1S with passthrough (daisy-chain) |
| **Power** | 48V DC received from Hub or previous camera in chain |
| **Weight / Dimensions** | 121 g / 43 × 81 × 39 mm |
| **Montaje** | 1/4-20 thread (tripod) + 2× M3 |
| **Temp range** | -20 a 65°C |

---



---

## ⚡ Why Edge AI? The Five Advantages Over Cloud

This system is designed from the ground up for **edge inference** — all AI processing happens locally on the Raspberry Pi 5 + Hailo-8L, not in the cloud. Here's why that matters in the field:

### 1. No Internet Required

Rose farms in Cayambe sit at 2,800 meters above sea level. 4G coverage is unreliable at best. Edge AI processes every image locally — **zero dependency on connectivity**. The system works whether there's a cell signal or not.

### 2. Real-Time Decisions

A rose ready for harvest has a **24-hour optimal cutting window**. Cloud round-trips add seconds of latency per image. Edge inference on the Hailo-8L runs at **30 FPS** — the decision happens at the camera, not in a data center 5,000 km away.

### 3. Data Sovereignty

Crop images never leave the farm. No cloud storage costs. No privacy concerns. No risk of proprietary agricultural data being exposed or monetized. **The farmer owns the data, always.**

### 4. Cost at Scale

| Approach | Monthly cost (4 cameras, daily) |
|---|---|
| Cloud API (GPT-4V / Replicate) | ~$200–400/month |
| **Edge AI (RPi5 + Hailo-8L)** | **$0/month** after hardware purchase |

The Hailo-8L accelerator costs ~$70. Paired with a Raspberry Pi 5 (~$80), the entire inference server is **under $200 one-time**, with zero recurring fees.

### 5. Power Efficiency

| Device | Performance | Power draw |
|---|---|---|
| Hailo-8L | 13 TOPS | **5W** |
| NVIDIA Jetson Nano | 0.5 TOPS | 5–10W |
| Cloud GPU (A100) | 312 TOPS | 300W (per request) |

The Hailo-8L delivers **26× the TOPS-per-watt** of a Jetson Nano. It can run 24/7 on a small solar panel — critical for remote agricultural deployments.

### Bonus: The 10BASE-T1S Advantage

The VineCam system uses a single twisted-pair bus for both data and power. One cable daisy-chains across the field. No Ethernet switches. No Wi-Fi access points. No repeaters. This is infrastructure designed for **real-world agricultural conditions**, not climate-controlled server rooms.


## 🔌 10BASE-T1S Connection

### Why 10BASE-T1S?

El VineCam Hub de Hellbender utiliza 10BASE-T1S como bus de campo. Ventajas sobre alternativas:

| Criterio | 10BASE-T1S | Alternativas |
|---|---|---|
| **Velocidad** | 10 Mbps | WiFi: inestable en campo, requiere APs |
| **Topología** | Bus MultiDrop / Daisy-chain | Ethernet 100BASE: requiere switch ($) |
| **Cableado** | single twisted pair | CAN/RS-485: no tienen TCP/IP |
| **Power** | 48V DC sobre el mismo cable | LoRaWAN: 50 kbps, sin TCP/IP |
| **Latencia** | Determinística (PLCA) | WiFi: alta variabilidad |
| **Alcance** | 25 m total del bus | Suficiente para invernadero/cama de cultivo |

**Ventaja decisiva:** Un solo cable lleva datos + alimentación. Sin switches, sin repetidores. Ideal para campo agrícola.

### Estándar

IEEE 802.3cg-2019, capa física 10BASE-T1S:
- 10 Mbps half-duplex sobre par trenzado único
- Topología bus MultiDrop: hasta 8 nodos, ≤25 m
- PLCA (Physical Layer Collision Avoidance): acceso determinístico al medio
- SPoE vía IEEE 802.3bu (Power over Data Lines, Clase 5 hasta 13W)

### How VineCam Implements It

Hellbender usa un **RP2350** (Dual Cortex-M33 @ 150 MHz) como co-procesador dedicado exclusivamente al bus 10BASE-T1S. Esto libera al CM4 de la gestión en tiempo real de PLCA y timing del bus.

*Implementación técnica — ver anexo técnico.*

**Arquitectura de conexión de cámaras (daisy-chain con passthrough):**

*Implementación técnica — ver anexo técnico.*

Cada cámara tiene 2 puertos RJ45 10BASE-T1S y retransmite datos + 48V a la siguiente. No requiere switch ni cableado en estrella.

⚠️ **El pinout RJ45 de VineCam NO es estándar.** Solo conectar entre Hub y VineCam Units — nunca a otros dispositivos 10BASE-T1S.

---

## 📷 VineCam Cameras — Operation

### Image Capture (Hub REST API control)

*15 líneas de código de implementación — ver anexo técnico.*

### Resolution and Detection

Sensor 11.9 MP. A 1 m de altura sobre las rosas: ~0.28 mm/píxel.

| Objeto | Tamaño real | Píxeles | Detectable |
|---|---|---|---|
| Rose bud (rose_small) | 15-30 mm | 54-107 px | ✅ Excelente |
| Open rose (rose_large) | 50-100 mm | 179-357 px | ✅ Excelente |
| Aphids (pest) | 2-3 mm | 7-11 px | ⚠️ Marginal |
| Powdery mildew | 2-5 mm | 7-18 px | ⚠️ Requires high resolution |
| Botrytis (lesiones) | 5+ mm | 18+ px | ✅ Posible |

---

## 🧠 Modelo de IA

**Dataset:** RoseBlooming (Shinoda et al. 2023) — 519 imágenes aéreas, 2 clases COCO: `rose_small` (botón), `rose_large` (flor abierta).

**Arquitectura:** YOLOv8n entrenado en GPU → exportado a ONNX → compilado a Hailo `.hef` para el acelerador Hailo-8L. Inferencia en edge con DeGirum PySDK.

**Performance esperada:** mAP@0.5: 0.85–0.92, ~30 FPS en Hailo-8L.



---

## 📊 Monitoring and Visualization

### Observability Stack

| Componente | Propósito |
|---|---|
| **InfluxDB** | Almacenar series temporales: conteos diarios por nodo, ratio de floración |
| **Grafana** | Dashboard con gráficos de evolución fenológica, mapas de calor por cama |
| **Telegram Bot** | Alertas de cosecha lista, anomalías (sin floración, enfermedad) |
| **JSON diario** | Respaldo local en `/opt/data/roses/YYYY-MM-DD/results/` |

### InfluxDB Metrics

*Implementación técnica — ver anexo técnico.*

### Thresholds and Alerts

| Condición | Alerta | Acción |
|---|---|---|
| `rose_large > 20` (por nodo) | 🔴 Cosecha urgente | Telegram + email |
| `rose_large > 5` (por nodo) | 🟡 Cosecha próxima | Telegram |
| `bloom_ratio > 0.7` | 🟢 Pico de floración | Log + dashboard |
| `total == 0` (3 días seguidos) | ⚠️ ¿Cámara caída? | Telegram |
| `rose_small / total < 0.1` | 🔵 Fin de ciclo de floración | Dashboard |

---

## 📅 Cronograma

| Hora | Acción |
|------|--------|
| 03:00 | Trigger de captura en las 4 cámaras + descarga de imágenes |
| 03:02 | Inferencia batch con Hailo-8L → métricas a InfluxDB |
| 03:10 | Limpieza de imágenes > 30 días |
| 08:00 | Reporte matutino vía Telegram |



---

## 🧪 Future Expansions

### Disease Detection

Modelos adicionales ejecutados en paralelo sobre las mismas imágenes:

| Enfermedad | Dataset | Modelo |
|---|---|---|
| Powdery mildew | PlantVillage / anotación manual | YOLOv8n (fine-tune) |
| Botrytis | Imágenes propias de campo | YOLOv8n (fine-tune) |
| Mancha negra | PlantDoc / anotación manual | YOLOv8n clasificación |

**Alternativa:** Un solo modelo multiclase (5 clases: rose_small, rose_large, black_spot, powdery_mildew, botrytis) entrenado con dataset combinado.

### Real-Time Dashboard

- Grafana con consultas a InfluxDB
- Panel de "mapa de camas" con indicadores visuales
- Línea de tiempo de floración por cultivar
- Exportación de reportes PDF semanales

### Irrigation and Climate Control

- Integración con sensores de humedad de suelo (I²C)
- Irrigation valve control via GPIO
- Irrigation ↔ bloom speed correlation
- Possible Home Assistant integration

---

![Cayambe Water Culture Landscape](./images/cayambe_water_culture_landscape.png)

*Render: Cayambe — Agua, Cultura y Paisaje. Integración del invernadero en el paisaje andino.*

## 📚 References

### Paper and Dataset
- [RoseTracker: A system for automated rose growth monitoring (2023)](https://doi.org/10.1016/j.atech.2023.100271)
- [RoseBlooming-Dataset en GitHub](https://github.com/dahlian00/RoseBlooming-Dataset)
- [Google Drive con dataset completo](https://drive.google.com/drive/folders/1I7_3vqDzZNIPwwqqOph1MrEqo8ZxAy0r)

### 10BASE-T1S
- IEEE 802.3cg-2019 — Physical Layer specification
- IEEE 802.3bu — Power over Data Lines (PoDL)

### Hailo y DeGirum
- [DeGirum + Hailo Guide](https://docs.degirum.com/hailo)
- [Hailo DFC (Dataflow Compiler)](https://github.com/hailo-ai/hailo-dfc)
- [Hailo RPi5 Examples](https://github.com/hailo-ai/hailo-rpi5-examples)
- [DeGirum PySDK Examples](https://github.com/DeGirum/PySDKExamples)

### Hellbender VineCam
- [VineCam System — Página de producto](https://hellbender.com/product/vinecam-system/)
  - Hub: CM4 + RP2350 (bus controller dedicado) + Hailo-8 opcional
  - Cámaras: 11.9MP, 2× programmable LEDs, daisy-chain 10BASE-T1S con passthrough
  - 4 puertos 10BASE-T1S (RJ45, pinout no estándar) + Gigabit Ethernet + WiFi 5 + BLE
  - IMU + sensor T/P/RH integrados en el Hub
  - REST API para control y descarga de imágenes
  - 1 Hub + 4 Cámaras = $1,049–$1,199 USD. Pre-order Spring 2026. Designed and assembled in USA.

---

## 📁 Project Structure

*28 líneas de código de implementación — ver anexo técnico.*

---

*Document generated May 16, 2026.*
*Project: Node.ec — Intelligent monitoring services for precision agriculture.*
