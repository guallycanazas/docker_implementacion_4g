# 🚀 Red 4G LTE/VoLTE con Open5GS, Kamailio & srsRAN (Dockerizada) - GuallyTel

![Docker](https://img.shields.io/badge/Docker-25.0+-blue.svg)
![srsRAN](https://img.shields.io/badge/srsRAN-4G-green.svg)
![Hardware](https://img.shields.io/badge/SDR-USRP_X310-orange.svg)
![License](https://img.shields.io/badge/License-MIT-lightgrey.svg)

## 📖 Descripción General

Este proyecto ("GuallyTel") implementa una red móvil completa **4G LTE/VoLTE** utilizando herramientas de software libre y Radio Definida por Software (SDR).

A diferencia de despliegues estándar, esta arquitectura integra **Wowza Streaming Engine** para demostrar capacidades reales de **IPTV y streaming** sobre el plano de usuario LTE, además de servicios de VoLTE y SMS .

El núcleo de la red (EPC) y el subsistema IMS están contenerizados usando **Docker**, proporcionando un entorno modular y reproducible, mientras que la Red de Acceso (RAN) está impulsada por **srsRAN** conectado a un **Ettus USRP X310**.

### 🏗️ Arquitectura de la Red

El sistema se divide en dos bloques principales:
1. **Acceso de Radio (Nativo/SDR):** USRP X310 + srsRAN.
2. **Core de Red & IMS (Docker):** Contenedores gestionando el control y datos.

![Arquitectura de la red](docs/img/arquitectura.png)

**Componentes Clave:**
* **Core:** Open5GS (MME, HSS, SGW, PGW, PCRF).
* **IMS:** Kamailio (P-CSCF, I/S-CSCF) para VoLTE y señalización SIP.
* **RAN:** srsRAN 4G actuando como eNodeB.
* **Servicios:** PyHSS para gestión avanzada de suscriptores y Wowza para Streaming.

---

## 🛠️ Requisitos de Hardware

[cite_start]Para replicar el laboratorio, se utilizó la siguiente configuración [cite: 207-208]:

| Componente | Especificación | Función |
|------------|----------------|---------|
| **Host PC** | Ryzen 7 7435HS, 16GB RAM, Ubuntu 24.04 LTS | Ejecuta Docker y srsRAN. |
| **SDR** | **Ettus USRP X310** (Conexión 1GbE/10GbE) | Interfaz de radio LTE (eNodeB). |
| **Tarjetas SIM**| **Oyeitimes** USIM Programables (LTE/VoLTE) | Autenticación de usuarios. |
| **Programador**| Lector Smart Card SCR35xx USB | Personalización de SIMs. |
| **UE** | Samsung Galaxy S24 ULTRA/ Poco F7 | Dispositivos de usuario final. |

---

## ⚙️ Configuración de Red

[cite_start]Se utilizan subredes específicas para separar el tráfico de gestión de Docker del enlace físico de radio [cite: 217-218]:

* **Red Docker (Bridge):** `172.22.0.0/24` (Comunicación interna EPC/IMS).
* **Enlace Físico SDR:** `192.168.10.0/24` (Host <-> USRP).

### 📌 Tabla de IPs y Puertos

| Servicio | IP del Contenedor | Puerto Principal |
|----------|-------------------|------------------|
| **MME** | `172.22.0.9` | 36412 (SCTP) |
| **HSS** | `172.22.0.3` | Diameter |
| **UPF/PGW**| `172.22.0.8` | 2152 (GTP-U) |
| **Kamailio**| `172.22.0.20` | 5060 (SIP) |
| **Wowza** | `172.22.0.50` | 1935 (RTMP/HLS) |

---

## 🚀 Instalación y Uso

Sigue estos pasos secuenciales para levantar toda la infraestructura.

### 1️⃣ Prerrequisitos
Instalar los drivers UHD para el USRP X310 y Docker en Ubuntu 24.04 :

```bash
# Actualizar sistema e instalar Docker
sudo apt update && sudo apt install docker-ce docker-compose -y

# Instalar Drivers UHD (Ettus) y herramientas de compilación
sudo add-apt-repository ppa:ettusresearch/uhd
sudo apt update && sudo apt install libuhd-dev uhd-host git -y

# Descargar imágenes FPGA para el X310
sudo /usr/lib/uhd/utils/uhd_images_downloader.py
