# 📦 Smart QR & LoRa-Based Inventory Management and Secure Delivery System

> A complete solution to manage, track, and verify the movement of inventory with **QR-based product lifecycle mapping**, **LoRa-GNSS real-time tracking**, and **OTP-verified delivery confirmation**. Designed for high-security logistics pipelines and smart warehouses.

---

## 📖 Table of Contents

- [🔍 Project Overview](#-project-overview)
- [📦 Workflow Across Logistics Stages](#-workflow-across-logistics-stages)
- [🔐 Role-Based Access & Interfaces](#-role-based-access--interfaces)
- [📡 IoT Integration: LoRa + GNSS](#-iot-integration-lora--gnss)
- [🛠️ Tech Stack & Methodologies](#️-tech-stack--methodologies)
- [⚙️ System Modules & Flow](#️-system-modules--flow)
- [🔒 Security Architecture](#-security-architecture)
- [💾 Database Schema Overview](#-database-schema-overview)
- [📲 Mobile App Overview](#-mobile-app-overview)
- [📈 Scalability & Deployment](#-scalability--deployment)
- [🔮 Future Enhancements](#-future-enhancements)
- [📄 License](#-license)

---

## 🔍 Project Overview

This project is designed to solve core issues in logistics: lack of transparency, tamperable deliveries, manual errors, and loss of goods. Using **static QR codes for product identity**, **GNSS with LoRa for low-power real-time tracking**, and **OTP verification for delivery**, this system brings automation and security across warehouse and delivery operations.

---

## 📦 Workflow Across Logistics Stages

### 1. **Central Warehouse Stage**
- Unique **QR Code** generated and mapped with:
  - UUID
  - Product info
  - Source/Destination warehouse IDs
  - Timestamp
- Access restricted to **Central Manager** only
- Status: `Dispatched from Central`

### 2. **Local Warehouse Receipt**
- Validated by **Local Warehouse Incharge**
- Role and QR scan history checked before status change
- Status: `Received at Local Warehouse`

### 3. **Out for Delivery**
- Assigned to delivery agent
- On scan: 
  - Status = `Out for Delivery`
  - **LoRa-E5 + Quectel L89 GNSS** powered up
  - Starts sending GNSS location data periodically

### 4. **Final Delivery**
- QR scanned again by delivery agent
- Triggers **OTP** to customer’s registered number
- Delivery marked as completed only after OTP verification

### 5. **Post-Delivery**
- LoRa GNSS deactivates
- All records logged: time, GPS, delivery agent, OTP verification
- Status: `Delivered`

---

## 🔐 Role-Based Access & Interfaces

| Role               | Permissions                                                                 |
|--------------------|------------------------------------------------------------------------------|
| 🧑‍💼 Manager         | Approves dispatches, monitors logs, full audit access                       |
| 🏢 Warehouse Incharge | Scans incoming/outgoing items, manages inventory at a local level          |
| 🚚 Delivery Agent   | Sees assigned deliveries, scans QR, handles OTP delivery                    |
| 👤 Customer         | Gets OTP, confirms delivery, reports issues                                 |

All actions are governed via **JWT + RBAC (Role-Based Access Control)**.

---

## 📡 IoT Integration: LoRa + GNSS

### 📍 LoRa Module
- **STM32WLE5JC** microcontroller
- LoRaWAN support for long-range, low-power comms

### 🌐 GNSS Module
- **Quectel L89 R2.0**
- Parses NMEA sentences
- Location logged and sent via LoRa to gateway

### 🔋 Power Management
- Deep sleep modes
- Optional PMIC + 3.6V LiPo battery
- Wakes on scan or motion (LIS3DH support optional)

### ⛓️ Payload Format Example
  13.189707,80.106296,45.60,1,0 (lat, lon, alt, emergency_button, anomaly_status)


---

## 🛠️ Tech Stack & Methodologies

### 🔧 Backend
- **Node.js + Express**
- REST API for QR, status updates, role validation, OTP
- JWT Auth, Bcrypt, HTTPS, CORS

### 🗃️ Database
- **MongoDB (recommended)** for flexibility  
- Tables/Collections:
  - Users
  - Products
  - Delivery Logs
  - Audit Logs

### 🖥️ Frontend Dashboard
- **React.js / Next.js**
- Admin views, product flows, delivery reports
- Authentication, role-based views

### 📲 Mobile App
- **Flutter / React Native**
- For delivery agents:
  - QR scan
  - LoRa device activation
  - OTP entry
- Offline-first logic with sync

---

## ⚙️ System Modules & Flow

### `backend/`

Built using **Node.js + Express**, connected to **MongoDB**.

- **models/**: Mongoose schemas for users, products, deliveries, and logs
- **routes/**: Endpoints like `/api/qr`, `/api/status`, `/api/otp`
- **middleware/**: 
  - `auth.js` → JWT validation  
  - `roles.js` → Role-based access filtering  
  - `errorHandler.js` → Unified error responses
- **server.js**: Loads middleware, connects DB, runs Express server

🔒 Features:
- JWT authentication
- Bcrypt for password hashing
- HTTPS, Helmet, CORS, and CSRF protection
- Role-Based Access Control (RBAC)

---

### `frontend/pages/`

Web dashboard using **React** (or Next.js if using SSR).

Key Pages:
- **Login & Auth**
- **Product Registration (QR Generation)**
- **Warehouse Scan Interface**
- **Delivery Logs Viewer**
- **Role-Based Dashboards**

💡 Frontend communicates with backend via secure REST APIs.

---

### `firmware/`

**Controller**: STM32WLE5JC  
**Tools**: STM32CubeIDE + HAL Drivers  
**Peripherals**:
- GNSS (Quectel L89 R2.0 via UART)
- LoRaWAN (SX126x onboard with Semtech stack)
- LIS3DH Accelerometer (via I2C/SPI)

#### `NMEA_GNSS/`
- Parses `$GNGGA`, `$GNRMC` NMEA sentences
- Extracts latitude, longitude, altitude
- Sends data on successful GPS fix

#### `LoRaWAN_Stack/`
- Uplink payload: `lat, lon, alt, anomaly_flag, emergency_flag`
- Uses ABP/OTAA to connect to LoRaWAN network
- Supports low-power modes

#### `LIS3DH_Motion/`
- Detects abnormal movement
- Triggers emergency uplink flag if threshold exceeded

### `mobile-app/`

Delivery agent app built in **Flutter** or **React Native**

Key Features:
- Login with role check
- QR scanning using phone camera
- Trigger LoRa/GNSS tracker
- Send and verify OTP on delivery
- Offline QR caching + sync on reconnect

### `docs/`

Contains all architectural and planning material:

- 🗺️ System architecture diagrams
- 🔁 Project workflow charts
- 🔐 Security and access model documents
- 🔧 Hardware interfacing details
- 📝 API Documentation (Swagger/OpenAPI optional)

---

## 🔐 Security Architecture

- **JWT Tokens** for stateless API access  
- **Bcrypt**-hashed passwords  
- **Encrypted QR Codes** with UUID and metadata  
- **CORS + HTTPS + Helmet.js** for API protection  
- **OTP Confirmation** using secure SMS gateway  
- **Audit Logs** for all status changes & role actions  

---

## 💾 Database Schema Overview

The database structure is optimized for high traceability, role-based operations, and secure logging.

### 📘 Collections Overview

#### 🧑‍💼 Users
Stores registered users with their roles and credentials.
```json
{
  "userId": "uuid",
  "name": "John Doe",
  "role": "manager | warehouse | agent | customer",
  "contact": "email or phone",
  "passwordHash": "bcrypt-hashed-string"
}
```

##### 📦 Products

Holds static QR info, status stages, and origin-destination data.

```json
{
  "productId": "uuid",
  "qrCodeData": "base64 or plain QR string",
  "source": "Central-001",
  "destination": "Local-104",
  "status": "Out for Delivery",
  "timestamps": {
    "created": "2025-04-01T10:30:00Z",
    "dispatched": "2025-04-01T12:00:00Z",
    "received": "2025-04-02T09:45:00Z",
    "delivered": "2025-04-02T15:30:00Z"
  }
}
```

##### 🚚 DeliveryLogs

Logs of GNSS points and OTP verification for each product.

```json
{
  "logId": "uuid",
  "productId": "linked to Products",
  "agentId": "linked to Users",
  "path": [
    {
      "lat": "13.189707",
      "lon": "80.106296",
      "alt": "45.60",
      "timestamp": "ISO8601"
    }
  ],
  "otpVerified": true,
  "deliveredAt": "ISO8601"
}
```

##### 📋 AuditLogs

Tracks every critical user or system action.

```json
{
  "logId": "uuid",
  "action": "DISPATCH_CONFIRMED | OTP_VERIFIED | ...",
  "performedBy": "userId",
  "timestamp": "ISO8601",
  "meta": {}
}
```

#### 📲 Mobile App Overview

##### 🎯 Purpose
Provides delivery agents with an intuitive app to:

- **Scan QR codes**
- **Trigger LoRa-based GNSS tracker**
- **Manage OTP-based delivery confirmation**

##### 📱 Core Features

- 📷 **QR Scanning** for metadata
- 🔗 **LoRa device activation** (via BLE or UART trigger)
- 🔐 **OTP entry interface** (customer receives OTP)
- 📴 **Offline caching** (important for poor network zones)
- 🔄 **Sync when reconnected to network**

##### 🛠️ Tech Stack
- **Flutter** (preferred) or **React Native**
- Uses **backend REST APIs**
- **Local storage** with secure caching (e.g., **Hive**, **SQLite**)

---

#### 📈 Scalability & Deployment

##### ⚙️ Scalability Strategy
- **Horizontal scaling** for backend using containerized microservices
- Separate **database clusters** for each region or client
- **MQTT/LoRaWAN** integration with gateways for sensor scalability
- **Offline-first mobile design** + sync improves field operability

##### ☁️ Deployment Architecture
- **Backend**: Node.js + Express hosted on Heroku/AWS EC2
- **Frontend**: Vercel or Netlify for React-based dashboard
- **Database**: MongoDB Atlas (Cloud)
- **LoRa Gateway**: WM1302 with Raspberry Pi
- **OTA Firmware Update**: Over USB-C/Serial port during sync
- **Mobile App**: APK via internal store / Play Store (optional)

---

#### 🔮 Future Enhancements
- 📦 **Product Categorization** via smart labels (RFID + QR Hybrid)
- 🌐 **Geo-Fencing Alerts** based on LoRa GNSS boundary logic
- 🔊 **Buzzer/Vibration Alerts** for anomaly or tampering
- 📊 **Predictive Delivery Analytics** using machine learning
- 🔁 **Return Logistics Flow** for failed deliveries
- ☁️ **Real-time Map View** with GNSS trails for admins

---

#### 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---
