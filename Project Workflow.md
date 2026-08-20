## 1. Edge-Processed Structural Integrity Node

This project shifts processing to a dedicated hardware node that monitors critical infrastructure (bridges, overpasses, pipelines), powered by solar + battery for continuous off-grid operation.

- **The Hardware Flow:** An ESP32-S3 is uniquely suited for this due to its vector instructions for neural networks and mathematical transforms. It samples high-frequency data from a piezo sensor only when triggered, cross-references it against strain and tilt using advanced statistical models, and outputs a confirmed binary state (Normal vs. Anomaly) over a low-power protocol (LoRa).

- **The Mechanism:** The device clamps onto infrastructure and listens for acoustic transients in a low-power mode. When a transient crosses a threshold, it wakes to perform a **Wavelet Transform**, mathematically isolating micro-fractures from noise. This triggers a high-resolution sampling window where a **Sequential Changepoint Detection** algorithm (like CUSUM) confirms if the strain baseline has permanently shifted. To ensure this shift is structurally abnormal, it calculates the **Physics-Informed Residual** against an expected load model. Finally, it verifies its own sensor health via **Mahalanobis Distance** to rule out hardware faults. Only if all mathematical criteria are met is an alert transmitted.

- **The Patentable Claim (Updated for 2.0):** The specific method of mathematically confirming a structural anomaly through an **event-triggered Wavelet Transform**, followed by **sequential changepoint detection** of the strain shift, validated against a **physics-informed residual model**, while distinguishing sensor-hardware faults via **Mahalanobis distance** and combining conflicting evidence using **Dempster-Shafer theory**—executed entirely on a low-power microcontroller without cloud connectivity.

- **The Engineering Hurdles:**

    1. **Wavelet & CUSUM Validation:** The hardest part isn't wiring the hardware; it's selecting the correct wavelet basis (e.g., Daubechies) for the fracture snaps and tuning the CUSUM threshold. This requires a controlled environment to record ground truth for the mathematical models.
    2. **Physics-Informed Residual:** Building a lightweight digital twin (beam-deflection model) that can run on an ESP32-S3 is computationally challenging. We must approximate the structure's physical physics without heavy finite-element overhead.
    3. **Thermal Calibration:** The self-calibration routine needs enough natural temperature variation during its 1–2 week learning window to produce a reliable structure-specific coefficient. 
    4. **Multivariate Sensor Health:** Defining what a "physically inconsistent" reading looks like in high-dimensional space requires deliberately inducing sensor faults (e.g., degrading a gauge's bond) to train the Mahalanobis distance thresholds.

Here is how the node's workflow is structured:

## 1. Acoustic Emission (Micro-Fracture Detection) — Wavelet Transformed

Vibration measures macro-movements; Acoustic Emission (AE) measures micro-events — the high-frequency energy released when a rigid material begins to micro-fracture.

- **The Hardware:** A specialized high-frequency piezoelectric contact microphone clamped flush to the material.
- **The Mathematical Role:** The piezo channel sits in a low-power comparator/interrupt mode. Once a transient crosses a threshold, the ESP32-S3 wakes Core 1 to run a **Continuous or Discrete Wavelet Transform (CWT/DWT)**. Unlike an FFT, this resolves the signal in both time and frequency, perfectly isolating the sharp, non-stationary "snap" of a micro-fracture from continuous background noise (like traffic or wind).

## 2. Strain and Deformation (Physical Bending) — Changepoint Detected

A structure might vibrate heavily but snap right back into place (elastic deformation). We need to know if it is permanently bending (plastic deformation) — and specifically, whether that permanent shift *followed* a fracture-like acoustic event.

- **The Hardware:** A foil strain gauge attached via a Wheatstone bridge circuit and a load-cell amplifier.
- **The Mathematical Role:** When a wavelet confirms a fracture snap, the system watches the strain reading. Instead of a blunt timing rule, it runs **Sequential Changepoint Detection (e.g., Page-Hinkley test)** to mathematically detect a true statistical regime shift in the baseline, robustly filtering out elastic noise.

## 3. Physics-Informed Residual Modeling (Digital Twin Cross-Check)

Even if strain shifts, we need to know if the shift is structurally dangerous for the current load.

- **The Innovation:** The device infers the load magnitude (e.g., a truck crossing) from the vibration signature and feeds it into a lightweight, on-device physical model (a simplified beam-deflection equation). 
- **The Mathematical Role:** The system computes the **Residual**: the difference between the *physics-predicted* expected strain and the *actual observed* strain. A large, persistent residual is the primary anomaly signal, providing a principled, explainable metric rather than just an ML "black box."

## 4. Environmental Compensation — Self-Calibrating

Steel expands in summer heat and contracts in winter cold, changing baseline strain readings.

- **The Hardware:** A BME280 measuring ambient temperature and humidity.
- **The Mathematical Role:** During an initial calibration window, the firmware maps observed temperature against observed strain to fit a **structure-specific thermal-offset coefficient** using linear/polynomial regression. This replaces generic textbook formulas with a model learned explicitly for *this* installation.

## 5. Spatial Tilt and Settling (Inclinometer)

If a support pillar is slowly sinking, vibration and acoustic sensors won't catch it because the movement is too slow.

- **The Hardware:** A 6-axis MEMS IMU (MPU6050 or BNO085) configured to read absolute tilt.
- **The Mathematical Role:** Acts as a long-term foundation sanity check, feeding into the multivariate sensor health matrix.

## 6. Sensor-Fault vs. Structural-Fault Discrimination via Mahalanobis Distance

Before any anomaly reaches the alert stage, the node runs a multivariate consistency check.

- **The Mathematical Role:** The system treats all sensors as a single multivariate point and measures its **Mahalanobis Distance** from the learned "normal" distribution. This catches subtle faults where individual sensors look fine, but their *relationship* has broken down (e.g., strain and acoustic no longer correlate). A high Mahalanobis distance without a physical trigger is flagged as a **sensor-health fault**.
- Furthermore, **Dempster-Shafer Evidence Theory** is used to combine conflicting confidence scores rigorously into a final decision.

## 7. Multi-Node Cross-Validation (Optional Scale-Up)

Where a second node is deployed on the same structure, a locally confirmed anomaly is broadcast over the LoRa mesh to nearby nodes. The alert is finalized only if a neighboring node independently reports a correlated anomaly, forming a distributed consensus without a cloud server.
