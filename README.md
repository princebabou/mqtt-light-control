# LuminaSync: Intelligent Lighting Control System ⚡

**System Architect**: Manzi Prince Babou  
**Version**: 1.0.0

You can schedule your **relay/light** to **turn ON or OFF at specific times** from a web browser UI!

Supports:
- ✅ Real Arduino hardware
- ✅ Fake Arduino simulation (no real hardware needed)

---

## 🚀 Project Structure

| File                | Purpose                                              |
|:--------------------|:-----------------------------------------------------|
| `light-scheduler.py` | Main MQTT subscriber + serial writer to Arduino      |
| `web-socket.py`      | WebSocket server to receive browser UI schedules and forward to MQTT |
| `fake-simulation.py` | Simulated Arduino using virtual serial (`loop://`)    |
| `frontend.html`      | Browser UI                                             |

---

## 🧩 System Matrix

### Component Interoperability
| Module | Protocol | Data Format | Security Level |
|--------|----------|-------------|----------------|
| Web UI | WebSocket | JSON | TLS 1.3 (Simulated) |
| Network Core | MQTT 3.1.1 | Binary | Basic Auth |
| Device Interface | Serial UART | ASCII | Physical Layer |

## 🛠 Requirements

- Python 3.7+
- `paho-mqtt`
- `pyserial`
- `websockets`
- An MQTT broker (this project uses public IP: **157.173.101.159**, port **1883**)

Install dependencies:

```bash
pip install paho-mqtt pyserial websockets
```

---

# Simulation Mode (Hardware-Free)

```bash
# Terminal 1: Virtual Device Interface
python fake-simulation.py

# Terminal 2: Control Plane Core
python light-scheduler.py

# Terminal 3: Communication Bridge
python web-socket.py

# Access UI: http://localhost:8000/frontend.html
```
---

## ⚡ How To Run with hardware

1. **Change port** in `light-scheduler.py`:

   From:
   ```python
   arduino = serial.serial_for_url('loop://', baudrate=9600, timeout=1)
   ```

   To (example for Windows COM5 or Linux /dev/ttyUSB0):
   ```python
   arduino = serial.Serial('COM5', baudrate=9600, timeout=1)
   ```

2. **Flash your Arduino with this sketch: also provided as 'sched_on_off.ino'**

   ```cpp
    int relayPin = 2;

    void setup() {
    Serial.begin(9600);
    pinMode(relayPin, OUTPUT);
    digitalWrite(relayPin, HIGH); // Relay OFF initially
    }

    void loop() {
    if (Serial.available() > 0) {
        String cmd = Serial.readStringUntil('\n');
        cmd.trim();
        if (cmd == "ON") {
        digitalWrite(relayPin, LOW); // Relay ON
        } else if (cmd == "OFF") {
        digitalWrite(relayPin, HIGH); // Relay OFF
        }
    }
    }
   ```

3. **Then repeat the same steps:**
   - Run `light-scheduler.py`
   - Run `web-socket.py`
   - Open `frontend.html`
   - Schedule ON/OFF times

✅ Now it controls the **real** relay/light!

---

## 🖥 Frontend (Browser UI)

Your frontend must:
- Connect to WebSocket: `ws://localhost:8765`
- Send schedules like: `12:30 ON` or `19:45 OFF`

**Run provided file 'frontend.html':**

---

## 🧐 How it All Works (Simple)

| Step | Description |
|:-----|:------------|
| 1 | UI sends schedule (e.g., `10:30 ON`) to WebSocket server |
| 2 | WebSocket server forwards the message to MQTT broker |
| 3 | `light-scheduler.py` subscribes and saves the schedule |
| 4 | At the correct time, it sends `ON` or `OFF` over Serial |
| 5 | Arduino receives the command and toggles the relay/light |

---

# Troubleshooting

## Serial Monitor (Physical Device)
cu -l /dev/ttyUSB0 -s 9600

## MQTT Traffic Analysis
mosquitto_sub -h 157.173.101.159 -t "lumina/#" -v

## WebSocket Debug
websocat ws://localhost:8765