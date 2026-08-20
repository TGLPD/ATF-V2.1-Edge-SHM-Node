# ATF V-2.1 Edge-Processed Structural Health Monitoring (SHM) Node
## 15. Testing & Validation Protocols

This document defines the comprehensive test protocols and acceptance criteria for both the individual subsystems and the fully integrated edge-processed SHM node. These protocols are designed to rigorously validate the system's ability to reliably detect structural faults (specifically in concrete highway bridges) while rejecting environmental noise and sensor degradation.

---

### 1. Testing Philosophy

The testing methodology for the ATF V-2.1 SHM Node follows a structured, bottom-up approach:

- **Independent Subsystem Validation:** Each core subsystem (acoustic, strain, environmental, etc.) must be independently tested and validated against its specific acceptance criteria before full integration. This isolates faults and ensures unit-level reliability.
- **Full Pipeline Integration Testing:** Integration tests validate the causal relationship pipeline—ensuring that an acoustic event correctly triggers the strain correlation window and that the final decision logic handles true and false positives appropriately.
- **Real-World Demo Scenarios:** High-level demonstration scenarios are defined to prove the four key capabilities of the system to stakeholders: environmental robustness, elastic event rejection, sensor fault discrimination, and genuine fracture detection.
- **Reproducibility:** All tests must be reproducible with documented physical setups, known reference signals, and precise procedures.

---

### 2. Subsystem Unit Tests

#### Test 1: Piezo Sensor + Comparator Interrupt
- **Objective:** Verify that the acoustic emission hardware can reliably wake Core 1 upon detecting a sharp transient impact while ignoring gentle baseline noise.
- **Setup:** High-frequency piezoelectric contact microphone bonded to a concrete sample; LM358 comparator circuit connected to the ESP32-S3 external wake interrupt pin.
- **Procedure:** 
  1. Apply impacts of varying intensities to the concrete surface (e.g., fingernail tap, coin drop, hammer strike).
  2. Apply gentle rubbing or low-frequency continuous vibration to the surface.
- **Expected Result:** The interrupt fires for sharp impacts exceeding the reference threshold. The interrupt does NOT fire for gentle touches or continuous low-amplitude noise.
- **Acceptance Criteria:** >95% detection rate for threshold-crossing impacts; <5% false trigger rate.
- **Measurement Method:** Log interrupt wake timestamps via serial output and compare against a manual physical event log.

#### Test 2: Wavelet Transform Accuracy
- **Objective:** Validate the Daubechies Discrete Wavelet Transform (DWT) implementation on the ESP32-S3 for correctly extracting energy from high-frequency transient signals.
- **Setup:** Inject known digital test signals (a synthetic fracture transient superimposed with low-frequency noise) directly into the processing pipeline (bypassing the ADC).
- **Procedure:** Run the DWT algorithm on the synthetic test signals and compute the energy distribution across decomposition levels.
- **Expected Result:** Signal energy is heavily concentrated in the fast decomposition levels (Levels 1-3) for the sharp transient, and in the slower levels (Levels 4+) for the low-frequency noise.
- **Acceptance Criteria:** The transient energy is correctly isolated in >90% of test cases.
- **Measurement Method:** Compare the onboard calculated wavelet coefficients against a known, pre-calculated MATLAB or Python reference implementation.

#### Test 3: Acoustic Classifier (TinyML)
- **Objective:** Ensure the machine learning classifier correctly distinguishes between genuine fracture acoustic signatures and background noise.
- **Setup:** A dataset of pre-recorded audio samples representing fractures, traffic rumble, wind buffeting, and non-structural impacts.
- **Procedure:** Pass the wavelet-extracted features from each audio sample into the TinyML classifier and record the output classification.
- **Expected Result:** The classifier accurately categorizes the samples as either "Fracture" or "Non-Fracture / Noise".
- **Acceptance Criteria:** >90% accuracy on a held-out test set; <10% false positive rate for non-structural noise.
- **Measurement Method:** Generate a confusion matrix (fracture vs. non-fracture) from the test run results.

#### Test 4: HX711 Strain Reading
- **Objective:** Verify the linearity, accuracy, and noise floor of the physical strain measurement hardware.
- **Setup:** Foil strain gauge bonded to a concrete test beam in a full or half Wheatstone bridge configuration; HX711 load-cell amplifier connected to the ESP32-S3.
- **Procedure:** Apply incrementally increasing known weights to the beam to induce bending. Record the digital output from the HX711 at each step.
- **Expected Result:** A highly linear relationship between the applied load and the digital reading, with a stable baseline (low noise).
- **Acceptance Criteria:** Noise floor < 50 microstrain equivalent; linearity regression coefficient (R²) > 0.95.
- **Measurement Method:** Plot the applied load against the digital reading and compute the R² value.

#### Test 5: Page-Hinkley Changepoint Detection
- **Objective:** Validate the algorithm's ability to quickly and accurately detect a permanent baseline shift in the strain time-series.
- **Setup:** Provide a synthetic, noisy strain signal containing a known, permanent shift (changepoint) at a specific sample index.
- **Procedure:** Stream the synthetic signal through the Page-Hinkley changepoint detection algorithm and monitor for the trigger flag.
- **Expected Result:** The algorithm flags a changepoint shortly after the actual shift occurs, without triggering on the baseline noise preceding the shift.
- **Acceptance Criteria:** Detection occurs within 500ms (equivalent sample count) of the actual changepoint; zero false positives on the stable portion of the signal.
- **Measurement Method:** Compare the detected changepoint sample index to the known ground-truth index.

#### Test 6: Thermal Compensation
- **Objective:** Verify the autonomous thermal compensation model can successfully decouple structural temperature-induced strain from physical load strain.
- **Setup:** Node installed on a concrete surface with no physical load applied. A heat lamp is positioned to gradually induce temperature changes in the concrete and sensors.
- **Procedure:** Gradually heat the concrete surface over a specified period. Continuously monitor the BME280 temperature reading, the raw strain reading, and the thermally compensated strain output.
- **Expected Result:** The raw strain reading will drift significantly with temperature. The compensated strain reading should remain flat and stable near zero.
- **Acceptance Criteria:** The variance of the compensated strain is < 20% of the variance of the raw strain over the full tested temperature range.
- **Measurement Method:** Plot raw strain vs. compensated strain over the temperature ramp.

#### Test 7: Mahalanobis Distance Sensor-Fault Detection
- **Objective:** Prove the multivariate anomaly detection system can distinguish between a structural fault and a localized sensor degradation or failure.
- **Setup:** A fully calibrated node with known-good sensors operating in a steady state.
- **Procedure:** Deliberately induce specific sensor faults:
  - (a) Loosen or partially peel the strain gauge bond to simulate debonding.
  - (b) Cover the piezo sensor with foam or damping material to simulate sensitivity loss.
  - (c) Apply a magnet near the IMU to simulate bias drift.
- **Expected Result:** The Mahalanobis distance calculation exceeds the established fault threshold for each induced condition, identifying the inter-sensor relationship breakdown.
- **Acceptance Criteria:** Correct fault detection in >90% of induced fault scenarios; correct identification of the anomalous channel.
- **Measurement Method:** Log the Mahalanobis distance and the per-channel contribution scores during the test.

#### Test 8: LoRa Communication
- **Objective:** Ensure reliable transmission of alert payloads over the required distance using the 865-867 MHz ISM band.
- **Setup:** Two SX1278 modules—one acting as the transmitting node, the other as a gateway receiver connected to a PC.
- **Procedure:** Transmit test alert packets at various distances, ending with a 1 km line-of-sight test.
- **Expected Result:** Reliable packet reception at the gateway without corruption.
- **Acceptance Criteria:** >99% packet delivery success rate at 1 km line-of-sight.
- **Measurement Method:** Record the Received Signal Strength Indicator (RSSI) and calculate the packet loss rate.

---

### 3. Integration Tests

#### Integration Test 1: Full Pipeline — Genuine Fracture
- **Setup:** A fully calibrated node bonded to a concrete test beam.
- **Procedure:** Apply an increasing load using a hydraulic press until the concrete audibly cracks and physically deforms.
- **Expected Flow:** 
  1. Piezo interrupt wakes Core 1.
  2. Wavelet analysis confirms the acoustic signature as a fracture transient.
  3. The causal time window opens, and the Page-Hinkley algorithm monitors strain.
  4. A sustained changepoint in the strain data is detected.
  5. The Mahalanobis check confirms sensors are healthy (no debonding).
  6. A genuine fracture alert is compiled and transmitted via LoRa.
- **Acceptance Criteria:** The LoRa alert is successfully transmitted within 15 seconds of the physical crack event.

#### Integration Test 2: Full Pipeline — False Positive Rejection (Elastic)
- **Setup:** A fully calibrated node bonded to a concrete test beam.
- **Procedure:** Apply a substantial load to the beam to induce bending, then release the load to allow elastic recovery to the baseline state.
- **Expected Flow:**
  1. Piezo *may* trigger due to structural noise during loading.
  2. Wavelet *may* confirm an acoustic event.
  3. Strain shifts during load but returns to baseline upon release.
  4. No permanent changepoint is detected within the causal window.
  5. The event is classified as harmless elastic deformation; NO alert is transmitted.
- **Acceptance Criteria:** Zero false alerts generated during 10 distinct elastic loading cycles.

#### Integration Test 3: Full Pipeline — Thermal False Positive Rejection
- **Setup:** A fully calibrated node bonded to a concrete test beam, with a heat lamp directed at the assembly.
- **Procedure:** Heat the beam surface continuously over a 30-minute period to induce thermal expansion.
- **Expected Flow:**
  1. Raw strain drifts significantly due to thermal expansion.
  2. Thermal compensation model neutralizes the drift based on BME280 readings.
  3. Compensated strain remains below the changepoint threshold.
  4. NO alert is transmitted.
- **Acceptance Criteria:** Zero false alerts generated during a thermal ramp of at least 20°C.

---

### 4. Demo Scenarios (For Stakeholder Presentation)

These scenarios are designed to practically demonstrate the 4 core capabilities of the node in a presentation environment.

#### Demo A: Ignore Environmental False Positive
- **Action:** Induce a non-destructive heat gradient and harmless localized vibration (e.g., dropping a book nearby).
- **Result:** The system correctly stays silent.
- **Proves:** The thermal compensation model and wavelet filtering effectively reject environmental noise.

#### Demo B: Reject Elastic Acoustic Event
- **Action:** Deliver a sharp tap to the structure, causing an acoustic trigger but no permanent physical deformation.
- **Result:** The system evaluates the strain timeline, finds no permanent shift, and suppresses the alert.
- **Proves:** The causal correlation engine (wavelet + changepoint) correctly rejects non-damaging acoustic events.

#### Demo C: Detect Sensor Fault
- **Action:** Deliberately peel back a portion of the strain gauge to degrade its bond.
- **Result:** The system flags a "Sensor Fault/Debonding" maintenance notification, NOT a "Structural Fracture" alarm.
- **Proves:** The Mahalanobis distance multivariate analysis correctly discriminates between sensor degradation and true structural failure.

#### Demo D: Detect Genuine Fracture
- **Action:** Simulate a fracture by cracking a sacrificial concrete sample under a controlled load.
- **Result:** The system successfully navigates the entire pipeline and transmits a verified fracture alert.
- **Proves:** The end-to-end anomaly detection pipeline functions correctly for true positives.

#### Demo E (Phase 2 Stretch): Multi-Node Consensus
- **Action:** Deploy two nodes on the same structure. Induce a highly localized event (e.g., tapping directly on Node 1 only).
- **Result:** Node 1 detects an anomaly but Node 2 does not. The alert is suppressed locally due to lack of peer consensus.
- **Proves:** The cross-validation consensus mechanism prevents hyper-localized false positives from triggering system-wide alarms. *(Note: Phase 2 feature).*

---

### 5. Test Equipment Required

| Equipment | Purpose | Estimated Cost |
|-----------|---------|----------------|
| Concrete beam samples (3-5) | Physical testing substrate for integration tests | ₹500 - ₹1000 |
| Hydraulic press or clamp | Apply controlled, measurable loads to beams | ₹2000 - ₹5000 |
| Heat lamp / heat gun | Induce thermal variation for compensation tests | ₹500 |
| Digital multimeter | Verify circuit voltages and hardware sanity | ₹500 |
| Oscilloscope (optional) | Debug and verify high-speed analog signals (piezo) | ₹5000+ |
| Known weights (1kg, 5kg, 10kg) | Calibrate strain gauge linearity and scaling | ₹500 |

### 6. Test Data Recording and Documentation
To maintain rigorous quality control, all testing must be documented according to the following standards:
- **Data Logging:** All test data (raw sensor readings, state machine transitions, timing data) must be logged to CSV files containing accurate timestamps, system state identifiers, and a unique Test ID.
- **Visual Documentation:** High-resolution photos and video documentation of the physical test setup must be captured for each scenario.
- **Reporting:** Test results must be summarized in a standardized Pass/Fail format matrix, detailing any deviations from the acceptance criteria.
