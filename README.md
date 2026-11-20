# Smart Parcel Guard · Pure Java & Live Blynk Cloud Monitor

Smart Parcel Guard is a **zero-SQL**, Java 21 / Spring Boot parcel security monitoring application integrated directly with **Blynk Cloud** (`https://blr1.blynk.cloud`, template `TMPL3bxs0Yi_f` — `"parcel monitering system"`) and paired with ESP32 + ESP32-CAM Arduino firmware.

## Architecture (Zero SQL)

- **`backend/` (Pure Java 21 + Spring Boot 3.5.16)**:
  - Connects directly to Blynk Cloud REST endpoints (`/external/api/isHardwareConnected`, `/external/api/getAll`, `/external/api/update`) and polls virtual pins `V0`–`V8`.
  - Provides manual in-memory JWT authentication (`POST /api/auth/login`, `POST /api/auth/register`, `GET /api/auth/me`, `POST /api/auth/logout`). No database or SQL server is required.
  - Streams live Blynk readings, distance history, security alerts (`THEFT DETECTED`), and events over WebSocket (`/ws?access_token=<jwt>`).
  - Serves `frontend/` directly on `http://localhost:8081/` and accepts ESP32-CAM photo uploads (`POST /api/evidence/upload`) stored under `storage/images/`.
- **`frontend/` (Blynk-Matched Live Dashboard + Manual Login)**:
  - Manual Sign In & Create Account tabs (no hardcoded/auto-filled passwords).
  - Live datastream dashboard matching your Blynk Cloud view: `Door (V0)`, `Parcel (V1)`, `Distance (V2)` SVG gauge (`0–500 cm`), `RFID (V3)`, `Access (V4)`, `Camera (V5)`, `Theft (V6)`, `URL (V7)` + live photo preview, `Object (V8)`, ultrasonic distance history chart, security alert acknowledgment, event timeline, and runtime Blynk settings modal.
- **`firmware/` (Arduino IDE Sketches)**:
  - `firmware/esp32-tracker/esp32_tracker.ino`: ESP32 controller (MFRC522 RFID, Servo lock, HC-SR04 ultrasonic sensor, I2C LCD, buzzer, enrollment button, UART2 link to ESP32-CAM, Blynk Cloud `V0–V8`).
  - `firmware/esp32-cam-node/esp32_cam_node.ino`: AI-Thinker ESP32-CAM sketch capturing JPEG frames on UART command and uploading to `POST /api/evidence/upload`.

## Requirements

- **JDK 21** (Eclipse Temurin / Adoptium 21)
- **Maven 3.6.3+**

## Running the Application

1. Copy `.env.example` to `.env` if you don't already have one, and set `BLYNK_AUTH_TOKEN`, `SPG_JWT_SECRET`, and `SPG_ADMIN_USERNAME` / `SPG_ADMIN_PASSWORD` (the Java application automatically loads `.env` at startup).
2. Start the Java backend from `backend/` (ensure `JAVA_HOME` points to JDK 21):

   ```powershell
   $env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-21.0.13.9-hotspot"
   $env:Path = "$env:JAVA_HOME\bin;$env:Path"
   Set-Location backend
   mvn spring-boot:run
   ```

3. Open **`http://localhost:8081/`** in your browser (or open `frontend/index.html` with VS Code Live Server on port `5500`).
4. On the **Manual Sign In** screen, type the operator credentials configured in `.env` (or switch to **Create Account** to register a new operator account in memory).

## Verification

Run the Java backend test suite from `backend/`:

```powershell
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-21.0.13.9-hotspot"
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
Set-Location backend
mvn clean test
```
