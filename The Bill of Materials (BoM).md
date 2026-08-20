## The Bill of Materials (BoM)

### Core Node (per unit)

| **Component**                                  | **Purpose in the Node**                                                                           | **Est. Cost** |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------| ------------- |
| **ESP32-S3 Dev Board**                          | Dual-core MCU with vector instructions for onboard mathematical transforms (Wavelets) and TinyML.   | ₹450          |
| **BF350 Strain Gauge with (ADC) + Amp Module**  | Measures deformation. Provides high-resolution input for the Sequential Changepoint Detection.      | ₹250          |
| **Piezo Disk + LM358 Module (w/ comparator)**   | Captures high-frequency acoustic emissions. Comparator enables low-power interrupt wake-on-threshold to trigger the Wavelet Transform. | ₹180      |
| **MPU6050 / BNO085**                            | 6-axis IMU to track macro-vibration baselines and permanent structural tilt. BNO085 preferred for lower long-term bias drift. | ₹150 – ₹900 |
| **BME280**                                      | Measures ambient temperature for the self-calibrated, structure-specific thermal regression model.   | ₹300          |
| **SX1278 (LoRa)**                               | Low-power radio transceiver for transmitting confirmed anomaly alerts.                            | ₹400          |
| **Small Solar Panel (5–6V, 1–2W) + MPPT/charge controller** | Off-grid power for continuous outdoor operation.                                    | ₹500          |
| **18650 Cell + TP4056 (with solar-compatible charge path)** | Battery buffer and protection module for night/overcast operation.                  | ₹250          |
| **Prototyping Consumables**                     | Cyanoacrylate (superglue) for the foil gauge, breadboard, weatherproof enclosure, jumper wires.    | ₹250          |

**Total Estimated Cost (single node): ~₹2,730**

> **Pro-Tip for the Build:** When you buy the Strain Gauge Module, ensure it specifically mentions having an onboard ADC or digital output pin. If it only outputs an analog voltage, the ESP32-S3's internal ADC can read it, but an external 24-bit ADC (like the one built into an HX711-based module) will give you much cleaner data for the Sequential Changepoint Detection (CUSUM) algorithm, which is sensitive to noise.

### Second Node (Optional — for Multi-Node Cross-Validation)

Adding a second, identical node on the same structure (a different mounting point) enables the distributed consensus innovation. Cost is approximately the same as the core node above (~₹2,700), for a combined two-node deployment of **~₹5,400**. 

## The Data-Flow Architecture (Updated for 2.0)

The ESP32-S3 has two cores. To prevent data bottlenecks, hardware polling is separated from intensive mathematical inference using FreeRTOS. The pipeline below reflects the rigorously updated, mathematically-focused architecture.

**1. Low-Power Listening State:**
The piezo channel sits in a low-power comparator mode, and Core 0 polls the BME280 and IMU via I2C on a relaxed schedule (e.g., every 2–5s) — no continuous high-rate sampling. This is the default, battery/solar-conserving state.

**2. Event-Triggered Wake & Wavelet Transform:**
When the piezo comparator detects a transient crossing the acoustic threshold, an interrupt wakes Core 1. Core 1 immediately runs a **Continuous or Discrete Wavelet Transform (CWT/DWT)** on the acoustic burst to mathematically isolate the time and frequency of a micro-fracture snap. Simultaneously, it opens a high-resolution sampling window for strain and tilt.

**3. Thermal Compensation (Self-Calibrated):**
Before the strain reading is used, the firmware applies the *learned*, structure-specific thermal coefficient (derived via regression during the initial calibration window) rather than a fixed textbook offset formula.

**4. Causal Correlation via Changepoint Detection:**
Instead of a simple timing rule, the system runs a **Sequential Changepoint Detection algorithm (e.g., CUSUM)** on the compensated strain data. It mathematically confirms whether the strain baseline has experienced a permanent regime shift following the wavelet-confirmed acoustic transient.

**5. Physics-Informed Residual Check:**
The device infers the load magnitude from the IMU/vibration data and feeds it into a lightweight on-device physical model (beam-deflection). It compares the *predicted* expected strain against the *actual* changepoint-confirmed strain to calculate the **Residual**. A large residual confirms the shift is structurally dangerous.

**6. Sensor-Fault Discrimination (Mahalanobis Distance):**
Before escalating an alert, the node calculates the **Mahalanobis Distance** of the current sensor readings as a single multivariate point against its learned "normal" distribution. If the sensors are behaving inconsistently with one another (e.g., strain jumps but acoustics/tilt are silent), a high Mahalanobis distance flags a **sensor-hardware fault** rather than a structural alarm. **Dempster-Shafer theory** merges the confidence scores from conflicting sensors.

**7. (Optional) Multi-Node Consensus:**
If a second node is deployed, a locally confirmed anomaly is broadcast as *provisional* over the LoRa mesh. The alert is only finalized once a neighboring node reports a correlated anomaly.

**8. State Machine & Transmission:**
If the final, mathematically confirmed anomaly score exceeds the defined threshold, the ESP32 wakes the SX1278 LoRa module from deep sleep, transmits an encrypted anomaly alert to the gateway, and logs the event to flash memory.

> **Key insight:** The integration of Wavelet Transforms and Physics-Informed Residuals significantly increases the burst compute load on Core 1, but the *event-triggered* architecture ensures Core 1 still spends the vast majority of its time in deep sleep. This prevents the mathematical upgrades from draining the solar/battery budget.
