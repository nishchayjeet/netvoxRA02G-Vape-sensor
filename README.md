# 🚦 Netvox RA02G Sensitivity Configuration Cheat Sheet

# RA02G Reporting Interval Configuration Reference

## 🔧 Shortest Possible Reporting Interval (for diagnostics)

The **shortest possible reporting interval** you can configure for the RA02G is:

- **MinTime = 0x0001 (1 second)**
- **MaxTime = 0x0001 (1 second)**

> ⚠️ **WARNING:** The manual explicitly does **not recommend** setting such a low interval due to **battery drain risk** — even with external power. However, if you're troubleshooting why a device has stopped reporting, it's acceptable to use for **brief diagnostics** only.

---

### ✅ Downlink Payload for Shortest Report Interval

To configure the minimum report interval via downlink, use:

- **Hex:** `01D7000100010000000000`
- **Base64:** `AdcAAQABAAAAAAA=`
- **FPort:** `7`

This sets:

- `MinTime = 0x0001` (1 second)
- `MaxTime = 0x0001` (1 second)

---

### ⚙️ How to Send This

Use your LNS (e.g., ChirpStack or IoTCommunity.Space) to enqueue this payload to the device on **FPort 7** as a **confirmed downlink**.

---

### ❗ If Still Not Reporting After This

- ✅ Verify the device is **still joined** to the network
- ✅ Use the **Rejoin Command** (FPort `0x20`):

    - **Hex:** `010000003C03`
    - **Base64:** `AQAAADwD`
    - **FPort:** `32` (hex 0x20)

Or press **Function Key ①** to manually force a rejoin (green LED should flash once).

---

Let us know if you want a **full diagnostic downlink bundle** (e.g., rejoin + interval + smoke debounce override).

---

## 🟢 Recommended Configuration (Manufacturer Default)

The **best recommended reporting interval** for the RA02G—according to the manufacturer’s balance of reliability and power consumption—is:

### ✅ Recommended (Default) Interval

- **MinTime = 0x0384 (900 seconds = 15 minutes)**
- **MaxTime = 0x0384 (900 seconds = 15 minutes)**

This means:

- The device **wakes up every 15 minutes**
- Reports are sent **even if no change occurs**

---

### 🟢 Recommended Downlink Payload

- **Hex:** `01D7038403840000000000`
- **Base64:** `AdcDhAOEAAAAAAA=`
- **FPort:** `7`

---

### 📋 Why This Setting Is Recommended

| Parameter   | Value        | Purpose                                         |
|-------------|--------------|--------------------------------------------------|
| **MinTime** | 900 seconds  | Minimum interval between change-triggered reports |
| **MaxTime** | 900 seconds  | Maximum interval for sending heartbeat reports   |
| **Battery Impact** | Low  | Suitable for 24/7 operation, even on backup battery |

---

## 🛠️ Additional Recommended Downlinks

To fully apply manufacturer defaults and ensure stable behavior:

### 1. 🕒 Set MinTime & MaxTime to 15 min
- **Hex:** `01D7038403840000000000`
- **Base64:** `AdcDhAOEAAAAAAA=`
- **FPort:** `7`

### 2. 🚬 Set SmokeDebounceTime = 300s, ResumeTime = 10s
- **Hex:** `0CD7012C0A000000000000`
- **Base64:** `DNcBLAoAAAAAAA==`
- **FPort:** `7`

### 3. 🔊 Set Smoke Sensitivity to Level 1 (Highest)
- **Hex:** `03D7010000000000000000`
- **Base64:** `A9cBAAAAAAAAAA==`
- **FPort:** `7`  
(Optional: Use `0x02`, `0x03`, etc., for lower sensitivity)

---

### 🔄 Rejoin (Optional - if device not responding)

- **Hex:** `0100000E1003`
- **Base64:** `AQAAChAD`
- **FPort:** `32` (0x20)

> Forces device to check and rejoin the network every 1 hour, after 3 failed attempts.

---

### ⏱️ Recommended Send Order

1. MinTime/MaxTime (Heartbeat)
2. Smoke Debounce + Resume Time
3. Smoke Sensitivity (if needed)
4. Rejoin command (only for disconnected devices)

---

Let us know if you'd like a **batch-downlink file** or **server-side script** to enqueue these automatically.


------------------------------------------------------


## 1️⃣ SMOKE Sensitivity

### **Set SMOKE Sensitivity**
| Sensitivity Level        | HEX to Send              | BASE64 to Send         |
|-------------------------|--------------------------|------------------------|
| 1 (most sensitive)      | 03D7010000000000000000   | A9cBAAAAAAAAAAA=       |
| 2                       | 03D7020000000000000000   | A9cCAAAAAAAAAAA=       |
| 3                       | 03D7030000000000000000   | A9cwAAAAAAAAAA=        |
| 4 (least sensitive)     | 03D7040000000000000000   | A9dAAAAAAAAAAA=        |

- **FPort:** `7`
- Paste into BASE64 field, click **Enqueue**.

### **Get/Verify SMOKE Sensitivity**
| Command                 | HEX                      | BASE64                |
|-------------------------|--------------------------|-----------------------|
| Get Smoke Sensitivity   | 04D7000000000000000000   | BNcAAAAAAAAAAA==      |

- Device responds with (example):  
    - Hex: `84D7030000000000000000` → **Level 3**
    - Hex: `84D7040000000000000000` → **Level 4**

#### **How to Interpret**
| Response (HEX)          | Level      | Meaning                         |
|-------------------------|------------|---------------------------------|
| 84D7010000000000000000  | 1          | Most sensitive (lots of alarms) |
| 84D7040000000000000000  | 4          | Least sensitive (few alarms)    |

---

## 2️⃣ NOISE Sensitivity

> Noise requires two values: **Threshold** (higher = less sensitive) and **Duration**.

### **Set NOISE Sensitivity**
| Threshold (dec/hex) | Duration (dec/hex) | HEX to Send             | BASE64                 |
|---------------------|--------------------|-------------------------|------------------------|
| 30 / 1E             | 10 / 0A            | 07D7001E000A0000000000  | B9cwHgAKAAAAAAAA      |
| 50 / 32             | 10 / 0A            | 07D70032000A0000000000  | B9cwMgAKAAAAAAAA      |
| 100 / 64            | 10 / 0A            | 07D70064000A0000000000  | B9cwZAAKAAAAAAAA      |

*(You can adjust “Threshold” as needed for your environment; higher numbers mean less sensitive to noise!)*

- **FPort:** `7`
- Paste into BASE64 field, click **Enqueue**.

### **Get/Verify NOISE Sensitivity**
| Command                 | HEX                      | BASE64                |
|-------------------------|--------------------------|-----------------------|
| Get Noise Sensitivity   | 08D7000000000000000000   | CdcAAAAAAAAAAA==      |

- Device responds with (example):  
    - Hex: `88D7001E000A0000000000` → Threshold = 30, Duration = 10  
    - Hex: `88D70064000A0000000000` → Threshold = 100, Duration = 10

#### **How to Interpret**
- **Byte 2-3:** Threshold (combine to get decimal, e.g., `0x001E = 30`)
- **Byte 4-5:** Duration (e.g., `0x000A = 10 seconds`)

---

## 3️⃣ SHOCK/TAMPER Sensitivity

### **Set SHOCK Sensitivity**
| Sensitivity Level        | HEX to Send              | BASE64 to Send         |
|-------------------------|--------------------------|------------------------|
| 1 (most sensitive)      | 05D7010000000000000000   | BdcBAAAAAAAAAAA=       |
| 10 (factory default)    | 05D70A0000000000000000   | BdcKAAAAAAAAAAA=       |
| 20 (least sensitive)    | 05D7140000000000000000   | BdcUAAAAAAAAAAA=       |

- **FPort:** `7`
- Paste into BASE64 field, click **Enqueue**.

### **Get/Verify SHOCK Sensitivity**
| Command                  | HEX                      | BASE64                |
|--------------------------|--------------------------|-----------------------|
| Get Shock Sensitivity    | 06D7000000000000000000   | BtcAAAAAAAAAAA==      |

- Device responds with (example):  
    - Hex: `86D7140000000000000000` → Level 20 (least sensitive)  
    - Hex: `86D7010000000000000000` → Level 1 (most sensitive)

#### **How to Interpret**
| Response (HEX)           | Level      | Meaning              |
|--------------------------|------------|----------------------|
| 86D7010000000000000000   | 1          | Most sensitive       |
| 86D7140000000000000000   | 20         | Least sensitive      |

---

## 📋 Step-by-Step (ChirpStack UI)

1. Go to **Queue** tab in your device.
2. **Set FPort = 7**
3. **Select BASE64**
4. **Paste your payload** (from the tables above)
5. **Click Enqueue**
6. Wait for the device to **reply with an uplink (FPort 7)**
7. **Check the Events/Frames for the new uplink:**
    - For smoke: look for `84D7XX...`
    - For noise: `88D7XXXXYYYY...`
    - For shock: `86D7XX...`
8. **Interpret using the tables above.**

---

## 📝 Sample Output & Their Meanings

| Uplink Payload           | Means                                        |
|-------------------------|----------------------------------------------|
| 84D7030000000000000000  | Smoke sensitivity now Level 3 (less sensitive) |
| 84D7000000000000000000  | Smoke uses hardware knob                     |
| 88D70032000A0000000000  | Noise threshold=50, duration=10s             |
| 86D7140000000000000000  | Shock sensitivity Level 20 (least sensitive)  |

---

## 🗃️ Summary Table: All Sensitivity Settings (Set and Get)

| Type    | Set Command (BASE64)     | Get Command (BASE64)   | Sample Response (HEX)         | Meaning                        |
|---------|--------------------------|------------------------|-------------------------------|--------------------------------|
| Smoke   | A9cwAAAAAAAAAA= (L3)     | BNcAAAAAAAAAAA==       | 84D7030000000000000000        | Level 3 (set remotely)         |
| Noise   | B9cwHgAKAAAAAAAA         | CdcAAAAAAAAAAA==       | 88D7001E000A0000000000        | Threshold 30, Duration 10s     |
| Shock   | BdcUAAAAAAAAAAA= (L20)   | BtcAAAAAAAAAAA==       | 86D7140000000000000000        | Level 20 (least sensitive)     |

---

## ⚙️ Other Options & Tuning Tips

- **To make it even less sensitive:**  
    - Increase the threshold (e.g., 200 = hex `00C8`)
    - Increase the duration (e.g., 20 seconds = hex `0014`)
- **To make it more sensitive:**  
    - Lower the threshold (e.g., 30 = hex `001E`)
    - Lower the duration (e.g., 5 seconds = hex `0005`)
- **To disable the noise alarm entirely:**  
    - Set threshold to `0xFFFF` (65535)

### **Example: Other HEX/BASE64 Commands**

| Setting             | Threshold | Duration | HEX to Send             | BASE64 to Send         |
|---------------------|-----------|----------|-------------------------|------------------------|
| Default             | 5         | 10       | 07D70005000A0000000000  | B9cwBQAIAAAAAAAAAA==   |
| Less sensitive      | 200       | 10       | 07D700C8000A0000000000  | B9cwygAKAAAAAAAA       |
| Even less sensitive | 300       | 10       | 07D7012C000A0000000000  | B9cxLAoAAAAAAAA=       |
| Disable noise alarm | 65535     | 10       | 07D7FFFF000A0000000000  | B9c/////////wAAAAAAAA= |

---
