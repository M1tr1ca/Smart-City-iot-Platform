# Smart City IoT Ecosystem

[![Tech Stack](https://img.shields.io/badge/Stack-Java%20%7C%20C%2B%2B%20%7C%20Docker%20%7C%20Android-blue.svg)](#technologies-and-tools)
[![Architecture](https://img.shields.io/badge/Architecture-Event--Driven%20%7C%20Microservices-orange.svg)](#system-architecture)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A complete, industrial-grade ecosystem for real-time management and monitoring of a **Smart City**. The system integrates physical sensor nodes (IoT Edge), an event-driven message broker (MQTT), a containerized enterprise backend server (Java EE + Docker), and a native mobile client (Android) featuring interactive maps of Madrid.

---

## System Architecture

The ecosystem follows a high-availability hybrid communication model: **Publish-Subscribe (MQTT)** for fast ingestion and real-time commands, alongside a **REST API (HTTP)** for historical data queries and system configuration.

```mermaid
flowchart TD
    subgraph Devices["Network Devices"]
<<<<<<< HEAD
        edge_node["IoT Edge Station<br/>(ESP32 + BME280)"]
        actuators["Screens / Traffic Lights<br/>(Simulator/Actuator)"]
    end

    subgraph Infra["Docker Infrastructure"]
        broker["Mosquitto MQTT Broker<br/>(mqtt-broker)"]
        tomcat["Tomcat Application Server<br/>(tomcat-server)<br/>[Backend Java EE]"]
        postgres[("PostgreSQL Database<br/>(base-datos)")]
    end

    subgraph UIs["User Interfaces"]
        android["Mobile Dashboard App<br/>(Native Android + OpenStreetMap)"]
    end

    %% Flows
    edge_node -->|"Publish readings - MQTT Port 3000"| broker
    broker -->|"Receive commands - MQTT Port 3000"| actuators
    broker -->|"Async ingestion"| tomcat
    tomcat -->|"Persistence"| postgres
    android -->|"API REST / JSON - HTTP Port 3002"| tomcat
    android -.->|"Real-time update - MQTT Port 3000"| broker
=======
        edge_node["IoT Edge Station<br>(ESP32 + BME280)"]
        actuators["Screens / Traffic Lights<br>(Simulator/Actuator)"]
    end

    subgraph Infra["Docker Infrastructure"]
        broker["Mosquitto MQTT Broker<br>(mqtt-broker)"]
        tomcat["Tomcat Application Server<br>(tomcat-server)<br>[Backend Java EE]"]
        postgres[("PostgreSQL Database<br>(base-datos)")]
    end

    subgraph UIs["User Interfaces"]
        android["Mobile Dashboard App<br>(Native Android + OpenStreetMap)"]
    end

    %% Flows
    edge_node -->|Publish readings<br>MQTT (Port 3000)| broker
    broker -->|Receive commands<br>MQTT (Port 3000)| actuators
    broker -->|Async ingestion| tomcat
    tomcat -->|Persistence| postgres
    android -->|API REST / JSON<br>HTTP (Port 3002)| tomcat
    android -.->|Real-time update<br>MQTT (Port 3000)| broker
>>>>>>> 8f4a145a7fc5126bfc41dee692d5379738ada970
```

---

## Repository Structure

The project is divided into three main components that isolate each technological layer:

*   **[`iot-sensor-node`](./iot-sensor-node/)**: C++ source code for the **ESP32** microcontroller.
    *   Real-time readings using the **BME280** weather sensor (Temperature, Humidity, Pressure).
    *   Local status visualization via a 7-segment display directly controlled by GPIO.
    *   Concurrent task scheduling using **FreeRTOS** for thread management.
    *   Periodic publication and subscription to control topics using the MQTT protocol.
*   **[`smart-city-backend`](./smart-city-backend/)**: Central server and infrastructure container.
    *   Orchestrated with **Docker Compose** to spin up in seconds:
        *   An **Eclipse Mosquitto MQTT** broker (port `3000`).
        *   A **PostgreSQL 16** database with relational schemas for streets, sensors, and readings.
        *   An **Apache Tomcat 10.1** application server running an enterprise **Java EE 11 / Servlet** app.
    *   Includes a listener agent (`MQTTSubscriber` via Eclipse Paho) that persistently captures readings into the database.
*   **[`android-client-app`](./android-client-app/)**: Mobile dashboard.
    *   Developed in Native Android (Kotlin/Java).
    *   Integration of interactive maps of Madrid using **OSMDroid (OpenStreetMap)**.
    *   Hybrid network connection: direct MQTT subscription to the broker for real-time updates, and REST API consumption via **Retrofit** to retrieve detailed historical data.

---

## Technologies and Tools

*   **Languages:** Java 21, C++ (Arduino/ESP-IDF), XML.
*   **IoT & Hardware:** ESP32 DevKit, BME280 Sensor (I2C), 7-Segment Display, Status LEDs, MQTT Protocol, FreeRTOS.
*   **Backend & Servers:** Jakarta EE 11 (Servlets, JNDI DataSources), Apache Tomcat 10.1, Maven, Eclipse Paho MQTT.
*   **Database & DevOps:** PostgreSQL 16, Docker, Docker Compose, mosquitto-clients.
*   **Mobile:** Android SDK, OSMDroid (Maps), Retrofit 2 (HTTP/REST Requests), Gson.

---

## Quick Start Guide

### 1. Spin up the Infrastructure (Backend and DB)
Make sure you have **Docker** installed and **Docker Desktop** running (on Windows).

```bash
# Enter the backend directory
cd smart-city-backend/ServerUbicua-master/ServerUbicua-master

# Spin up the containers in the background
docker-compose up -d
```
This will initialize:
1. The MQTT Broker at `localhost:3000`.
2. The PostgreSQL database at `localhost:5432`.
3. The Tomcat server at `localhost:3002/Server/`.

### 2. Configure and Deploy the IoT Node (ESP32)
1. Open the `iot-sensor-node` directory with your favorite IDE (Arduino IDE or VS Code + PlatformIO).
2. Adjust your local WiFi credentials and the MQTT Broker IP in `config.h`.
3. Compile and flash the code to the **ESP32**.

### 3. Run the Android Application
1. Open the `android-client-app` folder in **Android Studio**.
2. Modify the IP address of your backend server in the connection configuration (change `10.0.2.2` if using a real physical device).
3. Compile and install the app on your device or emulator.

---

## Contributors
<table>
  <tr>
    <td align="center">
      <a href="https://github.com/juanprzzz">
        <img src="https://github.com/juanprzzz.png" width="100px;" alt="Juan Pérez Resa"/><br />
        <sub><b>Juan Pérez Resa</b></sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/M1tr1ca">
        <img src="https://github.com/M1tr1ca.png" width="100px;" alt="David Nicolás Mitrica"/><br />
        <sub><b>David Nicolás Mitrica</b></sub>
      </a>
    </td>
  </tr>
</table>

---
*Academic project developed for the **Ubiquitous Computing** course - University of Alcalá (UAH).*
