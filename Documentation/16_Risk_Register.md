# 16. Risk Register

This document catalogs all technical, schedule, and operational risks associated with the ATF V-2.1 Edge-Processed Structural Health Monitoring (SHM) Node project.

## 1. Risk Assessment Framework

Risks are evaluated based on their likelihood of occurrence and their potential impact on the project's success. 

- **Probability Ratings:** Low, Medium, High
- **Impact Ratings:** Low, Medium, High, Critical
- **Risk Level:** Calculated intuitively as Probability × Impact

Each identified risk includes the following attributes:
- **Description:** A detailed explanation of the risk.
- **Category:** Technical, Schedule, or Operational.
- **Probability:** The likelihood of the risk occurring.
- **Impact:** The severity of the consequences if the risk materializes.
- **Mitigation Strategy:** Proactive steps taken to reduce the probability or impact.
- **Contingency Plan:** Reactive steps to take if the risk materializes despite mitigation.
- **Owner:** The team member or group responsible for managing the risk.

---

## 2. Technical Risks

### TR-01: Wavelet Transform Computational Budget
- **Description:** Executing the Daubechies Discrete Wavelet Transform (DWT) on 1024 samples may exceed the ESP32-S3 processing budget, causing pipeline timeouts or dropped events.
- **Probability:** Medium
- **Impact:** High
- **Mitigation:** Benchmark DWT execution on the ESP32-S3 early in the development cycle (Week 4). Utilize the ESP-DSP library optimized for Xtensa vector instructions. Reduce the sample buffer to 512 samples if necessary.
- **Contingency:** Fall back to an FFT-based detection method. This provides reduced accuracy for transient signals but is proven to run efficiently on the ESP32.

### TR-02: Page-Hinkley Parameter Tuning
- **Description:** Tuning changepoint detection parameters (δ, λ) for concrete bridge strain signals can be challenging, potentially leading to missed genuine fracture detections or excessive false alarms.
- **Probability:** High
- **Impact:** Medium
- **Mitigation:** Collect extensive ground-truth strain data during Phase 1 lab testing. Perform systematic parameter sweeps against datasets with known, induced changepoints.
- **Contingency:** Fall back to a simpler threshold-based anomaly detection mechanism with wider safety margins.

### TR-03: Strain Gauge Bonding Quality
- **Description:** Cyanoacrylate adhesives may not adhere reliably to rough or wet concrete surfaces, resulting in noisy readings, baseline drift, or complete sensor debonding.
- **Probability:** High
- **Impact:** High
- **Mitigation:** Test multiple adhesives (cyanoacrylate, epoxy, Z70) on representative concrete samples. Implement strict surface preparation protocols (grinding smooth, cleaning with solvents, thorough drying). Validate bonding in a humidity-controlled test chamber.
- **Contingency:** Employ mechanical clamping or bolting mechanisms as a backup attachment method. Accept a higher noise floor and adjust detection thresholds accordingly.

### TR-04: Insufficient Temperature Range During Calibration
- **Description:** The mandatory 14-day calibration phase may coincide with weather patterns (e.g., overcast monsoon season) that do not provide sufficient diurnal temperature variation to accurately model the thermal expansion coefficient.
- **Probability:** Medium
- **Impact:** Medium
- **Mitigation:** Schedule deployment during seasons known for wide diurnal temperature ranges (e.g., March–June in India). Accept a polynomial fit even if the R² value is borderline, provided it improves upon uncompensated raw data.
- **Contingency:** Extend the calibration period to 21 or 28 days. Implement a fallback mechanism allowing manual entry of expected thermal coefficients based on structural material properties.

### TR-05: TinyML Model Accuracy
- **Description:** The acoustic classifier (neural network) may fail to achieve the >90% accuracy target due to a lack of varied training data representing genuine concrete fractures.
- **Probability:** Medium
- **Impact:** High
- **Mitigation:** Augment the training dataset with synthetically generated fracture signals. Ensure data collection spans multiple different concrete samples and structural geometries.
- **Contingency:** Rely solely on wavelet energy thresholding without the neural network classifier. This reduces the system's ability to discriminate between fractures and complex noise but maintains baseline functionality.

### TR-06: HX711 Noise Floor
- **Description:** The inherent noise floor of the HX711 ADC may obscure the subtle strain level changes required for reliable changepoint detection.
- **Probability:** Low
- **Impact:** High
- **Mitigation:** Configure the HX711 for 80 SPS mode (which exhibits lower noise than the 10 SPS mode). Ensure proper electromagnetic shielding of analog lines and utilize true differential measurements.
- **Contingency:** Upgrade the hardware design to incorporate the ADS1115 (16-bit I2C ADC), which offers lower intrinsic noise and better software filtering capabilities, albeit at a slightly higher cost.

### TR-07: LoRa Range in Deployment Environment
- **Description:** LoRa RF signals may suffer severe attenuation due to the dense steel reinforcement (rebar) present in concrete bridges, significantly reducing the effective communication range.
- **Probability:** Medium
- **Impact:** Medium
- **Mitigation:** Strategically place the LoRa antenna external to the main steel structural elements. Conduct comprehensive range testing during Phase 1 field trials.
- **Contingency:** Configure the LoRa transceiver to use a higher spreading factor (sacrificing data rate for increased range). Deploy intermediate repeater nodes to bridge communication gaps.

### TR-08: ESP32-S3 Memory Overflow
- **Description:** The combined memory requirements of the TFLite Micro runtime, large wavelet analysis buffers, and FreeRTOS task stacks may exceed the ESP32-S3's available internal SRAM.
- **Probability:** Low
- **Impact:** Critical
- **Mitigation:** Establish a strict memory budget early in the architectural design phase (refer to [07_Firmware_Architecture.md](./07_Firmware_Architecture.md)). Allocate non-critical, large buffers (like extended historical logs) to external PSRAM.
- **Contingency:** Reduce the size of the TinyML model through aggressive quantization or pruning. Decrease the length of the acoustic and strain buffers. Switch to a streaming, windowed wavelet implementation.

---

## 3. Schedule Risks

### SR-01: Component Procurement Delays
- **Description:** Global supply chain issues may result in specialized components (e.g., ESP32-S3 modules, specific piezoelectric sensors) being out of stock or delayed.
- **Probability:** Medium
- **Impact:** Medium
- **Mitigation:** Place orders for all critical components during Week 1 of the project. Identify and vet secondary and tertiary suppliers.
- **Contingency:** Utilize older ESP32 or ESP32-S2 development boards as temporary substitutes for prototyping, accepting reduced ML execution speeds until correct components arrive.

### SR-02: Calibration Phase Calendar Dependency
- **Description:** The structural thermal calibration requires 14 actual calendar days of continuous data collection. This phase cannot be artificially accelerated in a real-world deployment, posing a hard constraint on the schedule.
- **Probability:** Certain
- **Impact:** Medium
- **Mitigation:** Initiate the hardware deployment and start the calibration data collection as early as possible, running it concurrently with ongoing firmware development and tuning.
- **Contingency:** For lab validation, use heat lamps and environmental chambers to artificially cycle temperatures, accelerating the calibration process (simulated deployment only).

### SR-03: Training Data Collection Time
- **Description:** Recording a statistically significant number of genuine fracture events on physical concrete samples may be more time-consuming than anticipated.
- **Probability:** Medium
- **Impact:** Medium
- **Mitigation:** Procure and prepare concrete test samples early in Phase 1. Schedule dedicated, focused data collection sessions with specialized destructive testing equipment.
- **Contingency:** Heavily rely on data augmentation techniques and synthetic data generation to synthesize a robust training dataset from a smaller number of physical samples.

---

## 4. Operational Risks

### OR-01: Outdoor Test Site Availability
- **Description:** Securing permission and access to a real, representative concrete highway bridge for field testing may prove difficult or face bureaucratic delays.
- **Probability:** Medium
- **Impact:** Medium
- **Mitigation:** Identify potential test sites early in the project lifecycle. Initiate contact with local highway departments or municipal authorities immediately.
- **Contingency:** Conduct "field" testing on representative concrete structures within a university campus or controlled laboratory environment (e.g., large concrete beams in a structural engineering lab).

### OR-02: Weather During Testing
- **Description:** Heavy monsoon rains (common from June to September in India) may severely interrupt planned outdoor testing periods or damage improperly sealed prototype hardware.
- **Probability:** High
- **Impact:** Low
- **Mitigation:** Schedule major outdoor field tests outside of the primary monsoon season. Ensure the node is housed in a verified IP65 or better weather-resistant enclosure.
- **Contingency:** Relocate testing to a covered outdoor location (e.g., under a large overpass or within an open-air testing facility) to protect equipment while maintaining environmental exposure.

---

## 5. Risk Matrix Summary

The following matrix categorizes the identified risks based on their Probability and Impact.

| Probability | Low Impact | Medium Impact | High Impact | Critical Impact |
| :--- | :--- | :--- | :--- | :--- |
| **High** | OR-02 (Weather) | TR-02 (Page-Hinkley Tuning) | TR-03 (Strain Bonding) | - |
| **Medium** | - | TR-04 (Temp Range)<br>TR-07 (LoRa Range)<br>SR-01 (Procurement)<br>SR-03 (Data Collection)<br>OR-01 (Site Availability) | TR-01 (DWT Budget)<br>TR-05 (TinyML Accuracy) | - |
| **Low** | - | - | TR-06 (HX711 Noise) | TR-08 (Memory Overflow) |
| **Certain**| - | SR-02 (Calibration Time) | - | - |

---

## 6. Risk Monitoring

Active risk management is critical for the success of the ATF V-2.1 project. The following protocols will be observed:
- **Bi-Weekly Reviews:** The risk register will be reviewed and updated every two weeks during project execution meetings.
- **Escalation Protocol:** Any risk whose probability increases to "High" AND whose impact is rated "High" or "Critical" must be immediately escalated to the project lead for dedicated intervention.
- **Dynamic Mitigation:** Mitigation strategies and contingency plans will be updated dynamically based on empirical test results, particularly following the Phase 1 lab validations.
