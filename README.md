# Fall Detection Alert System  

###  **Project Overview**  
This IoT-based **Fall Detection System** uses an **ESP32 microcontroller** with an **MPU6050 accelerometer** to detect sudden falls and instantly alert caregivers via **Telegram notifications**. The system consists of three main components:  
1. **ESP32 Wearable Device** – Detects falls and triggers alerts.  
2. **Flask Web Server** – Receives and processes alerts.  
3. **Telegram Bot** – Sends real-time notifications to caregivers.  

---

##  **Code Structure & Explanation**  

### 1. **ESP32 MicroPython Code (`esp32_fall_detector.py`)**  
 **Purpose**: Continuously monitors acceleration data and sends alerts when a fall is detected.  

🔹 **Key Features**:  
- **MPU6050 Sensor**: Measures acceleration in 3D space (`X, Y, Z`).  
- **Fall Detection Logic**: Triggers if total acceleration falls below **2.5g** (free-fall condition).  
- **Local Alerts**: Activates **buzzer (GPIO25)** and **LED (GPIO26)**.  
- **WiFi & HTTP POST**: Sends alerts to the Flask server via `urequests`.  

 **How It Works**:  
```python
while True:
    accel = mpu.acceleration
    total_accel = (accel[0]**2 + accel[1]**2 + accel[2]**2)**0.5  # Calculate magnitude
    if total_accel < FALL_THRESHOLD:  # Fall detected!
        send_web_alert()  # Send to Flask server
        activate_buzzer_led()  # Local alarm
```

---

### 2. **Flask Web Server (`fall_detection_server.py`)**  
 **Purpose**: Receives fall alerts from ESP32 and forwards them to Telegram.  

 **Key Features**:  
- **REST API Endpoint (`/fall-alert`)** – Accepts JSON payload from ESP32.  
- **Telegram Integration** – Formats alerts into readable messages and forwards them via the Telegram Bot API.  
- **Logging** – Saves all alerts in `fall_alerts.log`.  

 **How It Works**:  
```python
@app.route('/fall-alert', methods=['POST'])
def handle_alert():
    data = request.json  # Get ESP32 data
    log_alert(data['device_id'], data['timestamp'])  # Save to log
    send_telegram_alert(data)  # Forward to Telegram
    return "Alert processed!"
```

---

### 3. **Telegram Bot (`fall_detection_bot.py`)**  
**Purpose**: Sends emergency alerts to caregivers and provides system status checks.  

 **Key Features**:  
- **Instant Notifications** – Sends formatted fall alerts to Telegram.  
- **User Commands** (`/start`, `/status`, `/help`) – Helps caregivers interact with the system.  
- **Admin Log Access** (`/log`) – Shows recent alerts (admin-only).  

 **How It Works**:  
```python
def start(update, context):
    update.message.reply_text("🆘 Fall Detection Bot Active!")  

updater = Updater(BOT_TOKEN)  # Start bot
updater.start_polling()  # Listen for messages
```

---

##  **Setup & Deployment**  

###  **Hardware Setup**  
1. **ESP32-WROOM** + **MPU6050** (I2C: SDA=GPIO21, SCL=GPIO22)  
2. **Buzzer** (GPIO25) & **LED** (GPIO26)  

###  **Software Setup**  
1. **ESP32**: Upload `esp32_fall_detector.py` via Thonny/MicroPython.  
2. **Flask Server**: Run `fall_detection_server.py` (Python 3.7+).  
3. **Telegram Bot**: Deploy `fall_detection_bot.py` (requires `python-telegram-bot`).  

##  **Key Benefits**  
 **Real-time fall detection** with instant alerts  
 **Two-stage notification** (local buzzer + Telegram)  
 **Easy-to-use Telegram interface** for caregivers  
 **Persistent logging** for incident tracking  

---

