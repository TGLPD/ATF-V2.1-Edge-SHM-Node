# ATF V-2.1 Edge-Processed Structural Health Monitoring (SHM) Node
## Executive Summary & Project Overview

**One-Line Summary:** A self-contained, solar-powered, edge-computing structural health monitoring node that utilizes multi-modal sensor fusion and on-device TinyML to detect, validate, and report critical micro-fractures in concrete highway bridges while actively filtering out environmental noise and sensor faults.

## 1. The Problem Statement
Current Structural Health Monitoring (SHM) systems for concrete highway bridges suffer from four critical failures that prevent widespread, cost-effective deployment:

1. **High Power and Bandwidth Dependency:** Traditional continuous monitoring systems sample high-frequency data (like acoustics or vibration) constantly, requiring significant power (grid connection or large solar arrays) and high-bandwidth cellular connections to stream raw data to the cloud for analysis.
2. **Environmental False Positives:** Existing systems frequently trigger false alarms due to normal diurnal thermal expansion, traffic loads, or transient environmental noise. They often lack the on-device intelligence to distinguish between a harmless thermal shift and a genuine structural deformation.
3. **Delayed Micro-Fracture Detection:** Standard strain gauges and inclinometers only detect damage *after* significant macroscopic structural deformation has occurred. While acoustic emission sensors can detect the high-frequency "snap" of a micro-fracture, they are rarely integrated causally with strain data at the edge to confirm the event.
4. **No Sensor-Fault Distinction:** In harsh bridge environments, sensors frequently degrade, debond, or drift. Traditional systems treat anomalous sensor readings as structural anomalies, leading to expensive manual inspections for what is ultimately a broken sensor rather than a broken bridge.

## 2. The Proposed Solution
The ATF V-2.1 is an **Edge-Processed Structural Health Monitoring Node** designed to solve these exact failure modes. 

Built on a low-cost, dual-core ESP32-S3 microcontroller with vector instructions for ML/DSP acceleration, the system is entirely self-contained and solar-powered (~₹2,730 total node cost). Rather than streaming raw data to the cloud, it runs multi-modal sensor fusion and TinyML models directly on-device. It operates on an **event-triggered architecture**: the node remains in an ultra-low-power deep sleep, only waking its high-compute core when a passive hardware comparator detects a sudden acoustic transient. It then locally analyzes the acoustic signature, correlates it with subsequent strain deformation, compensates for thermal effects, and transmits only high-confidence, validated anomaly alerts over a low-bandwidth LoRa mesh network.

*See [System Architecture](./02_System_Architecture.md) for detailed hardware and software topology.*

## 3. Core Innovations (Prototype Scope)
The ATF V-2.1 architecture introduces six core patentable mechanisms for reliable edge SHM:

1. **Wavelet-Based Acoustic Emission Detection:** Instead of power-hungry continuous Fast Fourier Transforms (FFT), the system uses an event-triggered wake-up followed by a Daubechies wavelet transform. Wavelets resolve both time and frequency, perfectly isolating the short, non-periodic transient bursts typical of concrete micro-fractures.
2. **Causal Temporal-Correlation via Page-Hinkley Changepoint Detection:** When the acoustic subsystem flags a micro-fracture "snap," it triggers a tight temporal window where the strain subsystem uses Page-Hinkley changepoint detection to search for a statistically significant, permanent baseline shift. A "snap" followed by a permanent strain shift equals a validated structural event.
3. **Self-Calibrating Structure-Specific Thermal Model:** The node undergoes a 14-day autonomous calibration phase upon installation, learning the specific thermal expansion coefficient of its localized bridge segment via regression. It dynamically subtracts this modeled thermal strain from raw strain readings, eliminating diurnal false positives.
4. **Multivariate Sensor-Fault Discrimination via Mahalanobis Distance:** The node evaluates acoustic, strain, and tilt data not as isolated channels, but as a single multivariate point. By measuring the Mahalanobis distance from a continuously updated model of "normal joint sensor behavior," it can identify when inter-channel correlations break down (e.g., strain drifts but tilt doesn't), flagging sensor debonding or degradation rather than a structural fault.
5. **Event-Triggered Adaptive Sampling:** The dual-core FreeRTOS architecture allows Core 0 to perform ultra-low-power environmental polling (every 2-5s) while Core 1 remains entirely powered down. Core 1's burst compute (ML inference, wavelet transforms) is only triggered via hardware interrupts during specific acoustic events, minimizing the power budget.
6. **Multi-Node Cross-Validation Consensus:** Nodes communicate via a local LoRa mesh. Before broadcasting a critical alarm, a node requests cross-validation from its immediate neighbors. If neighbors do not independently report a correlated anomaly within a strict time/confidence window, the event is downgraded or logged as localized noise, achieving distributed consensus without cloud connectivity.

*See [TinyML Integration](./04_TinyML_Integration.md) for details on the ML components and [Patent Claims](./12_Patent_Claims.md) for intellectual property specifics.*

## 4. Phase 2 Stretch Goals
The following enhancements are planned for future iterations (Phase 2) and are outside the scope of the current V-2.1 prototype:

*   **Physics-Informed Residual Modeling ("Digital Twin"):** Implementing a lightweight, on-device beam-deflection physical model that predicts expected strain for a given load. Anomalies will be detected as residuals between the physics-based prediction and the actual observed sensor data.
*   **Dempster-Shafer Evidence Fusion:** Upgrading the decision engine to use the formal Dempster-Shafer theory of evidence. This will allow the system to merge "belief" from each independent subsystem (acoustic, strain, tilt) while explicitly representing uncertainty and conflicting data, mathematically resolving edge cases where sensors disagree.

## 5. Architectural Comparison

| Feature | Industry Standard SHM | Enterprise Cloud SHM | Published Academic Edge-TinyML | **ATF V-2.1 Node (Our Solution)** |
| :--- | :--- | :--- | :--- | :--- |
| **Data Processing** | Cloud-dependent, high bandwidth | Cloud-dependent, edge-filtering | Edge processing (usually single-modal) | **Multi-modal Edge Processing (No cloud required for inference)** |
| **Sensor Inputs** | Strain or Vibration | Multi-modal, streamed raw | Usually audio or vibration | **Acoustic + Strain + Inclinometer + Environmental** |
| **Acoustic Processing** | N/A or raw stream | Continuous FFT / Spectrograms | FFT or MFCC features | **Event-triggered Daubechies Wavelet Transform** |
| **False-Positive Filter** | Manual review / Cloud thresholds | Cloud ML models | Basic local thresholds | **Causal Time-Judge (Acoustic -> Strain Changepoint)** |
| **Thermal Compensation** | Factory datasheet values | Cloud-based historical modeling | Rarely implemented | **On-device 14-day autonomous structure-specific regression** |
| **Sensor Health** | Manual inspection | Cloud-based anomaly detection | Not addressed | **On-device Mahalanobis multivariate fault discrimination** |

## 6. Unique Selling Proposition (USP)
The ATF V-2.1 delivers enterprise-grade, multi-modal structural health monitoring at a fraction of the cost (~₹2,730/node) by moving the intelligence entirely to the edge. By utilizing event-triggered wakeups and causally correlating high-frequency acoustic emissions with macroscopic strain deformation, it provides instant, mathematically validated micro-fracture alerts. It eliminates the fatal flaws of existing systems—cloud dependency, thermal false positives, and sensor-fault alarms—creating a truly autonomous, deploy-and-forget monitoring network for critical civil infrastructure.

## 7. Primary Target Application
**Primary Focus:** Concrete highway bridges. 
The current acoustic signatures, thermal models, and load profiles are optimized specifically for the structural dynamics of reinforced concrete under vehicular load.

**Future Expansion:** The core causal-correlation architecture is modular and can be adapted in future firmware updates for:
*   Pipeline monitoring (pressure/flow correlation)
*   Steel truss bridges (different acoustic propagation and thermal expansion profiles)
*   Wind turbine blades

## 8. Expected Deliverables
1. **Hardware Prototype:** Fully assembled, solar-powered ESP32-S3 sensor node with custom analog front-end.
2. **Firmware:** FreeRTOS-based dual-core embedded C/C++ application.
3. **TinyML Models:** Trained weights for Acoustic Profiler and Calibration Learner, optimized for ESP32 vector instructions.
4. **DSP Pipeline:** Efficient C++ implementation of the event-triggered Daubechies wavelet transform and Page-Hinkley changepoint detection.
5. **System Documentation:** Comprehensive multi-document engineering suite (including this overview).
6. **LoRa Mesh Protocol:** Lightweight localized consensus and alerting protocol.
7. **Testing & Validation Report:** Bench-test data demonstrating successful causal correlation and thermal compensation.
8. *(Stretch)* **Phase 2 Technical Spec:** Design documents for Physics-Informed Residuals and Dempster-Shafer fusion.
