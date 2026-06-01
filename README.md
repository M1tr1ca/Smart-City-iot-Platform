# 🏙️ Smart City IoT Ecosystem

[![Tech Stack](https://img.shields.io/badge/Stack-Java%20%7C%20C%2B%2B%20%7C%20Docker%20%7C%20Android-blue.svg)](#-tecnologías-y-herramientas)
[![Architecture](https://img.shields.io/badge/Architecture-Event--Driven%20%7C%20Microservices-orange.svg)](#-arquitectura-del-sistema)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Ecosistema completo e industrial para la gestión y monitorización en tiempo real de una **Smart City** (Ciudad Inteligente). El sistema integra nodos sensores físicos (IoT Edge), un bróker de mensajería dirigido por eventos (MQTT), un servidor backend corporativo contenerizado (Java EE + Docker) y un cliente móvil nativo (Android) con mapas interactivos de Madrid.

---

## 🏗️ Arquitectura del Sistema

El ecosistema sigue un modelo de comunicación híbrido de alta disponibilidad: **Publish-Subscribe (MQTT)** para la ingesta rápida y comandos en tiempo real, junto con una **API REST (HTTP)** para las consultas de datos históricos y configuración del sistema.

```
┌────────────────────────────────────────────────────────┐
│                   DISPOSITIVOS DE RED                  │
│  ┌───────────────────────┐   ┌──────────────────────┐  │
│  │   Estación IoT Edge   │   │  Pantallas/Semáforos │  │
│  │   (ESP32 + BME280)    │   │ (Simulador/Actuador) │  │
│  └───────────┬───────────┘   └──────────▲───────────┘  │
└──────────────┼──────────────────────────┼──────────────┘
               │ (Publica lecturas)       │ (Recibe comandos)
               │ MQTT (Puerto 3000)       │ MQTT (Puerto 3000)
               ▼                          │
 ┌────────────────────────────────────────┴──────────────┐
 │                  DOCKER INFRASTRUCTURE                │
 │  ┌──────────────────────────────────────────────────┐ │
 │  │        Mosquitto MQTT Broker (mqtt-broker)       │ │
 │  └──────────────────────────┬───────────────────────┘ │
 │                             │                         │
 │                             ▼ (Ingesta asíncrona)     │
 │  ┌──────────────────────────────────────────────────┐ │
 │  │      Tomcat Application Server (tomcat-server)   │ │
 │  │                [ Backend Java EE ]               │ │
 │  └──────┬───────────────────────────────────▲───────┘ │
 │         │ (Persistencia)                    │         │
 │         ▼                                   │         │
 │  ┌──────────────────────────┐               │         │
 │  │   PostgreSQL (base-datos)│               │         │
 │  └──────────────────────────┘               │         │
 └─────────────────────────────────────────────┼─────────┘
                                               │ API REST / JSON
                                               │ (Puerto 3002)
 ┌─────────────────────────────────────────────┴─────────┐
 │                   INTERFACES DE USUARIO               │
 │        ┌──────────────────────────────────────┐       │
 │        │    Aplicación Móvil Dashboard        │       │
 │        │  (Android Nativo + OpenStreetMap)    │       │
 │        └──────────────────────────────────────┘       │
 └───────────────────────────────────────────────────────┘
```

---

## 🗂️ Estructura del Repositorio

El proyecto se divide en tres componentes principales que aíslan cada capa tecnológica:

*   **[`iot-sensor-node`](./iot-sensor-node/)**: Código fuente en C++ para el microcontrolador **ESP32**.
    *   Lecturas en tiempo real con sensor meteorológico **BME280** (Temperatura, Humedad, Presión).
    *   Visualización de estados local mediante display de 7 segmentos de control directo por GPIO.
    *   Planificación de tareas concurrentes mediante **FreeRTOS** para la gestión de hilos.
    *   Publicación periódica y suscripción a tópicos de control mediante protocolo MQTT.
*   **[`smart-city-backend`](./smart-city-backend/)**: Servidor central y contenedor de infraestructura.
    *   Orquestado con **Docker Compose** que levanta en segundos:
        *   Un bróker **Eclipse Mosquitto MQTT** (puerto `3000`).
        *   Una base de datos **PostgreSQL 16** con esquemas de calles, sensores y lecturas relacionales.
        *   Un servidor de aplicaciones **Apache Tomcat 10.1** corriendo una app corporativa **Java EE 11 / Servlet**.
    *   Inclusión de un agente oyente (`MQTTSuscriber` vía Eclipse Paho) que captura lecturas persistentemente en la BD.
*   **[`android-client-app`](./android-client-app/)**: Cuadro de mando móvil (Dashboard).
    *   Desarrollado en Android Nativo (Kotlin/Java).
    *   Integración de mapas interactivos de Madrid usando **OSMDroid (OpenStreetMap)**.
    *   Conexión de red híbrida: suscripción MQTT directa al bróker para actualizaciones en tiempo real y consumo de la API REST mediante **Retrofit** para obtener datos históricos detallados.

---

## 🛠️ Tecnologías y Herramientas

*   **Lenguajes:** Java 21, C++ (Arduino/ESP-IDF), XML.
*   **IoT & Hardware:** ESP32 DevKit, Sensor BME280 (I2C), Display 7-Segmentos, LEDs de Estado, Protocolo MQTT, FreeRTOS.
*   **Backend & Servidores:** Jakarta EE 11 (Servlets, JNDI DataSources), Apache Tomcat 10.1, Maven, Eclipse Paho MQTT.
*   **Base de Datos & DevOps:** PostgreSQL 16, Docker, Docker Compose, mosquitto-clients.
*   **Mobile:** Android SDK, OSMDroid (Mapas), Retrofit 2 (Peticiones HTTP/REST), Gson.

---

## ⚡ Guía de Inicio Rápido

### 1. Levantar la Infraestructura (Backend y DB)
Asegúrate de tener instalado **Docker** y **Docker Desktop** iniciado (en Windows).

```bash
# Entra al directorio del backend
cd smart-city-backend/ServerUbicua-master/ServerUbicua-master

# Levanta los contenedores en segundo plano
docker-compose up -d
```
Esto inicializará:
1. El Bróker MQTT en `localhost:3000`.
2. La Base de datos PostgreSQL en `localhost:5432`.
3. El Servidor Tomcat en `localhost:3002/Server/`.

### 2. Configurar y Desplegar el Nodo IoT (ESP32)
1. Abre el directorio `iot-sensor-node` con tu IDE favorito (Arduino IDE o VS Code + PlatformIO).
2. Ajusta las credenciales de tu WiFi local y la IP del Broker MQTT en `config.h`.
3. Compila y flashea el código al **ESP32**.

### 3. Ejecutar la Aplicación Android
1. Abre la carpeta `android-client-app` en **Android Studio**.
2. Modifica la dirección IP de tu servidor backend en la configuración de la conexión (cambia `10.0.2.2` si usas dispositivo físico real).
3. Compila e instala la app en tu dispositivo o emulador.

---

## 👥 Contribuidores
*   **Juan Pérez Resa** - [GitHub](https://github.com/juanpresaa)
*   **David Nicolás Mitrica** - [GitHub](https://github.com/M1tr1ca)

---
*Proyecto académico desarrollado para la asignatura de **Computación Ubicua** - Universidad de Alcalá (UAH).*
