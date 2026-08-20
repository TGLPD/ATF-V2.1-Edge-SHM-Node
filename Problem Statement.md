### The Problem Statement

Monitoring the structural integrity of critical infrastructure (bridges, pipelines, overpasses) in remote areas is currently inefficient and prone to alert fatigue. Industry-standard solutions typically stream raw sensor data continuously to cloud servers for processing. This creates four critical failures:

1. **High Power & Bandwidth Dependency:** Continuous data streaming drains batteries rapidly and requires expensive, high-bandwidth cellular or Wi-Fi networks that are often unavailable in remote locations.

2. **Environmental False Positives:** Structures naturally bend and expand due to daily temperature shifts (thermal expansion) or vibrate harmlessly due to routine traffic. Standard monitoring systems frequently flag these natural occurrences as structural failures — and generic, fixed-formula thermal compensation only partially solves this, because every structure's thermal behavior is slightly different.

3. **Delayed Detection of Micro-Fractures:** By the time a structure exhibits macro-level bending or tilting, severe damage has already occurred. Current systems struggle to isolate the high-frequency acoustic "snaps" of micro-fractures hidden beneath low-frequency environmental noise, and — critically — struggle to confirm whether a detected transient actually *caused* lasting damage or was a harmless elastic event.

4. **No Distinction Between Sensor Failure and Structural Failure:** Existing systems generally assume their sensors are always healthy. A corroding strain gauge, a debonding piezo sensor, or a drifting IMU can silently produce readings that look identical to genuine structural distress, leading to false alarms — or worse, false confidence.

### The Proposed Solution

We are proposing the development of a fully self-contained, **Edge-Processed Structural Health Monitoring Node**. Instead of acting as a "dumb" data logger that relies on cloud computing, this device executes multi-modal sensor fusion and machine learning inference entirely on the edge using an ESP32-S3 microcontroller, powered by an onboard solar panel and battery buffer for indefinite off-grid operation.

The node actively cross-references different physical domains — high-frequency acoustic emissions (micro-fractures), physical strain (bending), structural tilt (settling), and ambient temperature — but goes a step further than simple sensor fusion. It reasons about the *relationship in time* between these signals, calibrates its own understanding of "normal" for the specific structure it is bolted to, and continuously checks whether its own sensors are still trustworthy before it ever raises an alarm. It wakes its low-power LoRa transmitter to alert engineers only when a critical structural anomaly is mathematically confirmed — and, where a second node is deployed, only after a neighboring node agrees.

### Our Core Innovation & Patentable USP

Baseline edge-computed sensor fusion (acoustic + strain + tilt + temperature, filtered by an onboard ML model) is now a documented approach in existing academic literature and at least one prior patent. To secure a defensible, non-obvious patent position, this project introduces **five specific technical methods** layered on top of that baseline architecture:

- **1. Causal Temporal-Correlation Fracture Confirmation:**
   Rather than evaluating sensor readings as an isolated snapshot, the firmware requires a specific causal sequence before classifying an event as a genuine fracture: a high-frequency acoustic transient must be followed by a **permanent, non-recovering shift in the strain baseline within a defined time window** (e.g., strain does not return to its pre-event resting value within *T* seconds). An acoustic snap with no lasting strain shift is elastic (harmless); a strain shift with no preceding acoustic transient is likely slow settling, not fracture. Only the paired, time-ordered combination confirms damage.

- **2. Self-Calibrating, Structure-Specific Thermal Model:**
   Standard strain gauges use a fixed, generic thermal-offset formula. Our system instead runs an on-device calibration routine during an initial deployment window (e.g., the first 1–2 weeks), correlating naturally occurring temperature swings with strain readings to *learn* the specific thermal-expansion coefficient of the exact structure it is mounted on. This structure-specific model is then applied going forward, replacing the generic formula and adapting further if the coefficient drifts over seasons.

- **3. Sensor-Fault vs. Structural-Fault Discrimination:**
   The node cross-checks its own sensors against each other before trusting an anomaly signal. If a reading is inconsistent with what the other physical channels would predict (e.g., a strain jump with no corresponding acoustic or tilt signature, or a channel that has gone statistically flat/erratic relative to its own history), the system flags a **sensor-health fault** rather than a structural alarm — preventing false alarms caused by corroding gauges, debonded piezo sensors, or drifting IMUs, and enabling predictive maintenance of the node itself.

- **4. Event-Triggered Adaptive Sampling:**
   Instead of running continuous high-rate FFT analysis at all times, the acoustic channel operates in a low-power listening mode until a threshold-crossing transient is detected via interrupt. Only then does the node burst into high-resolution sampling of strain and tilt for a defined post-event window, directly feeding the causal-correlation check in Innovation 1. This preserves solar/battery margin during quiet periods while still capturing the fast dynamics that matter.

- **5. Multi-Node Cross-Validation Consensus (optional, scalable deployment):**
   Where budget allows a second node on the same structure, an anomaly detected locally is treated as provisional until a neighboring node, observing the same structural event from a different point, independently reports a correlated anomaly within a defined time and confidence window. Only mutually corroborated events trigger the final alert, sharply reducing single-node false positives (e.g., a localized impact or a single sensor's own drift) without requiring continuous cloud processing.

Together, these five methods move the system from "generic edge sensor fusion" (already documented in prior art) to a specific, defensible **method of temporally-correlated, self-calibrating, self-diagnosing, event-triggered, and optionally consensus-validated structural anomaly detection.**

### Expected Deliverables

1. A solar-and-battery-powered hardware prototype integrating the multi-sensor array and LoRa transceiver.
2. A localized TinyML model trained to evaluate thermally compensated, causally-correlated, fused sensor data.
3. An on-device self-calibration routine that learns structure-specific thermal coefficients during an initial deployment period.
4. A sensor-health diagnostic layer capable of distinguishing sensor faults from genuine structural anomalies.
5. An event-triggered sampling pipeline that keeps the acoustic channel in low-power listening mode until a transient is detected.
6. A physical demonstration proving the system's ability to: (a) ignore a false positive (induced heat/harmless vibration), (b) correctly reject an acoustic transient with no lasting strain shift, (c) correctly flag a simulated sensor fault as distinct from a structural anomaly, and (d) accurately trigger an alert for a genuine, causally-confirmed localized stress fracture.
7. (Stretch goal) A two-node consensus demonstration showing suppression of a single-node false positive.

