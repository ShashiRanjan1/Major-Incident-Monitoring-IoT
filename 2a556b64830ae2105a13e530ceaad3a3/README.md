# 🚨 Smart Incident Monitoring System (ServiceNow + Tuya IoT)

This project integrates **ServiceNow** with **Tuya Smart Devices** (like smart bulbs) to create a **visual alert system for major incidents**. Ideal for **war room scenarios**, the bulb color indicates the incident's lifecycle:

- 🔴 **Red Blinking** – New Critical Incident
- 🟢 **Green Solid** – Incident Acknowledged
- ⚪ **White Steady** – Incident Resolved

---

## 🧠 Project Overview

- Real-time smart alerts using Tuya smart bulb via **Tuya Cloud API**
- Hosted in **ServiceNow** with custom Script Includes and REST integration
- Generates **HMAC-SHA256** signatures using a custom `PureCryptoUtil` since `GlideCrypto` lacks advanced crypto capabilities
- Fully mimics **Postman CryptoJS** logic to ensure request integrity

---

## 🛠 ServiceNow Configuration

### 1. Script Includes

- **PureCryptoUtil**  
  Custom implementation of SHA256 and HMAC-SHA256 using `Packages.javax.crypto`.

- **SignatureUtil**
  - Builds canonical request string based on HTTP method, body hash, headers
  - Generates signature string required by Tuya Cloud

### 2. REST Message

- **Create REST Message** in ServiceNow:
  - Method: `POST`
  - URL: `https://openapi.tuya{region}.com/v1.0/devices/{device_id}/commands`
  - Authentication: None (we're signing manually)
  - Set required headers:
    - `client_id`
    - `access_token`
    - `sign`
    - `t` (timestamp)
    - `sign_method: HMAC-SHA256`
    - `Signature-Headers: nonce` (if needed)
    - `Content-Type: application/json`
  
### 3. System Properties (Maintain Easily)

Add the following properties to ServiceNow (or store securely via other means):

| Property Name                     | Description                        |
|----------------------------------|------------------------------------|
| `x_tuya.client_id`               | Tuya client ID                     |
| `x_tuya.client_secret`           | Tuya client secret                 |
| `x_tuya.device_id`               | Tuya smart device ID (bulb)        |
| `x_tuya.region`                  | API region (e.g. `us`, `eu`, `in`) |
| `x_tuya.access_token`            | Stored access token                |

---

## ⚙️ Tuya Configuration

### 1. Create a Tuya Developer Account

- Go to [https://iot.tuya.com](https://iot.tuya.com) and sign up
- Choose **Cloud > Create Cloud Project**
  - Enable **Device Control**, **Authorization Management**, and **Smart Home**
  - Select correct **Data Center** based on your region

### 2. Link Your Tuya App Devices

- Install the **Tuya Smart App** on your phone
- Add your smart bulb/device and test its on/off and color capabilities
- Go to the **Cloud Project** > **Link Tuya App Account**
  - Use the QR Code in Tuya Developer Console
  - Login via your app account

### 3. Obtain Credentials

- Copy **Client ID** and **Client Secret**
- Use them in ServiceNow properties
- Also fetch your **Device ID** from the "Device List"

---

## 🔐 Signature Logic Summary

Tuya requires custom HMAC-SHA256 signatures for all secure API calls.

**Signature Format:**

```text
METHOD\n
BODY_SHA256\n
HEADERS (optional)\n
REQUEST_PATH_WITH_QUERY



