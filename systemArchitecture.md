# This explains the technical layout for development
(System Architecture — Meter Monie)


## Overview
Meter Monie is a mobile-first electricity audit platform built to analyze prepaid power usage and forecast energy depletion over time.  
It combines user-entered meter data, real-time calculations, and optional smart-meter APIs all together as a product.

---

## Frontend
- **Technology:** React Native  
- **Purpose:** Provide users with a simple dashboard to:
  - Enter meter tokens or import data from connected meters  
  - View current balance, usage rate, and forecast duration  
  - Access analytics charts and cost predictions  
  - Receive notifications when power units are low  
- **Communication:** Uses HTTPS REST APIs to talk with the backend.

---

## Backend
- **Technology:** Node.js + Express.js  
- **Purpose:** Handle data processing, storage, and logic.  
- **Key Modules:**
  - **Auth Module:** Sign-up, login, and role (tenant, landlord, admin) management  
  - **Meter Data Module:** Accepts and validates token data, calculates kWh totals  
  - **Usage Forecast Module:** Uses consumption rate + purchased units to estimate remaining time  
  - **Analytics Module:** Generates reports for daily, weekly, and monthly usage  
  - **Notification Module:** Pushes reminders or alerts via Firebase Cloud Messaging  

---

## Database
- **Primary DB:** MongoDB Atlas (cloud)  
- **Collections:**
  - `users` – user profiles and meter links  
  - `meters` – meter numbers, DISCO details, and token logs  
  - `usage_logs` – timestamped consumption data  
  - `alerts` – scheduled or sent notifications  

---

## API Flow
1. User logs in on the app → Backend validates credentials (JWT) 
2. App requests linked meter data (manual or automatic)
3. Backend retrieves real-time usage from cache (Redis) or MongoDB  
4. Forecast module (Python) computes depletion predictions
5. API returns updated metrics → displayed on user dashboard
6. Notification module triggers alerts if threshold is crossed

----------------------  overview  ----------------------------
                ┌───────────────────────────────┐
                │       Mobile App(React native)│
                │   iOS / Android               │
                └──────────────┬────────────────┘
                               │ REST / WebSocket
                               ▼
                 ┌─────────────────────────────┐
                 │       API Gateway           │
                 │  (FastAPI / Node.js)        │
                 └──────────────┬──────────────┘
                                │
                                ▼
      ┌──────────────────────────────────────────────────────┐
      │            Hardware Layer (IoT Device)               │
      │  - Microcontroller (ESP32 or Raspberry Pi Pico W)    │
      │  - Power sensor(optical pulse or current transformer)|
      │  - Wi-Fi or GSM module for internet connectivity     │
      └──────────────┬───────────────────────────────────────┘
                     │  Publishes meter readings over MQTT to backend broker
                     ▼
      ┌──────────────────────────────────────────────────────┐
      │                       Backend Core                   │
      │  - Auth Service (JWT / OTP)                          │
      │  - User & Meter Management                           │
      │  - Device Registration                               │
      │  - Token Conversion Engine (₦ → kWh)                 │
      │  - Usage Analytics / Forecast Engine                 │
      │  - Notification Service (Mail + SMS)                 │
      └──────────────┬───────────────────────────────────────┘
                     │
                     ▼
          ┌────────────────────────┐
          │  MQTT Broker (EMQX)    │
          │  + Telemetry Ingestor  │
          └────────────┬───────────┘
                       │
                       ▼
              ┌────────────────┐
              │  Database Layer│
              │ (PostgreSQL +  │
              │   Redis Cache) │
              └──────┬─────────┘
                     │
                     |
                     ▼
    * A Separate Python microservice (connected to backend via REST or internal API)*
          ┌────────────────────────┐
          │ Forecasting Engine     │
          │ (Python / Scikit-learn)│
          └────────────────────────┘
    
---
## Feasibility & Scalability
- Node.js ensures fast, event-driven handling of meter and forecast calculations. 
- IOT device captures real-time consumption data from prepaid meters via a connected sensor or smart module while broadcasting it to users or DB.
  Security: Each device is registered with a unique device ID + JWT (JSON Web Token) for authentication.
            Encrypted device-to-server communication (HTTPS + MQTT over TLS)
- Relational DBs handle time-series data and multi-table joins(like users ↔ meters ↔ usage_logs)cleanly.  
- React Native enables cross-platform deployment (Android/iOS).  
- Firebase integration supports real-time alerts and authentication.

---

## Future Extensions
- Integration with payment gateways (e.g., Flutterwave, Paystack) for in-app top-ups
- Smart home integration for energy automation and IoT control  
- AI-driven forecasting models using historical and behavioral data  
- Multi-meter and multi-property management dashboard for landlords  
- Offline mode for token entry and deferred synchronization
- Dark mode and multi-language support for accessibility  