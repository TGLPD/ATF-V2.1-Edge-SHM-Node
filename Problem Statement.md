### The Problem Statement

Monitoring the structural integrity of critical infrastructure (bridges, pipelines, overpasses) in remote areas is currently inefficient and prone to alert fatigue. Industry-standard solutions typically stream raw sensor data continuously to cloud servers for processing. This creates four critical failures:

1. **High Power & Bandwidth Dependency:** Continuous data streaming drains batteries rapidly and requires expensive, high-bandwidth cellular or Wi-Fi networks that are often unavailable in remote locations.

2. **Environmental False Positives:** Structures naturally bend and expand due to daily temperature shifts (thermal expansion) or vibrate harmlessly due to routine traffic. Standard monitoring systems frequently flag these natural occurrences as structural failures — and generic, fixed-formula thermal compensation only partially solves this, because every structure's thermal behavior is slightly different.

3. **Delayed Detection of Micro-Fractures:** By the time a structure exhibits macro-level bending or tilting, severe damage has already occurred. Current systems struggle to isolate the high-frequency acoustic "snaps" of micro-fractures hidden beneath low-frequency environmental noise, and — critically — struggle to confirm whether a detected transient actually *caused* lasting damage or was a harmless elastic event.

4. **No Distinction Between Sensor Failure and Structural Failure:** Existing systems generally assume their sensors are always healthy. A corroding strain gauge, a debonding piezo sensor, or a drifting IMU can silently produce readings that look identical to genuine structural distress, leading to false alarms — or worse, false confidence.

### The Proposed Solution

We are proposing the development of a fully self-contained, **Edge-Processed Structural Health Monitoring Node**. Instead of acting as a "dumb" data logger that relies on cloud computing, this device executes multi-modal sensor fusion and machine learning inference entirely on the edge using an ESP32-S3 microcontroller, powered by an onboard solar panel and battery buffer for indefinite off-grid operation.

The node actively cross-references different physical domains — high-frequency acoustic emissions (micro-fractures), physical strain (bending), structural tilt (settling), and ambient temperature — but goes a step further than simple sensor fusion. It reasons about the *relationship in time* between these signals, calibrates its own understanding of "normal" for the specific structure it is bolted to, and continuously checks whether its own sensors are still trustworthy before it ever raises an alarm. It wakes its low-power LoRa transmitter to alert engineers only when a critical structural anomaly is mathematically confirmed — and, where a second node is deployed, only after a neighboring node agrees.


### Expected Deliverables

1. A solar-and-battery-powered hardware prototype integrating the multi-sensor array and LoRa transceiver.
2. A localized TinyML model trained to evaluate thermally compensated, causally-correlated, fused sensor data.
3. An on-device self-calibration routine that learns structure-specific thermal coefficients during an initial deployment period.
4. A sensor-health diagnostic layer capable of distinguishing sensor faults from genuine structural anomalies.
5. An event-triggered sampling pipeline that keeps the acoustic channel in low-power listening mode until a transient is detected.
6. A physical demonstration proving the system's ability to: (a) ignore a false positive (induced heat/harmless vibration), (b) correctly reject an acoustic transient with no lasting strain shift, (c) correctly flag a simulated sensor fault as distinct from a structural anomaly, and (d) accurately trigger an alert for a genuine, causally-confirmed localized stress fracture.
7. (Stretch goal) A two-node consensus demonstration showing suppression of a single-node false positive.

