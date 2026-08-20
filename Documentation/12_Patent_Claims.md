# 12. Patent Claims & Intellectual Property Strategy

## 1. Prior Art Analysis

The structural health monitoring (SHM) technology landscape generally falls into three categories:

*   **Standard Edge Sensors:** These rely on a single sensor (e.g., an accelerometer) and a simple threshold crossing (e.g., vibration > X g). They are well-known, widely deployed, and lack significant novelty.
*   **Enterprise Cloud SHM:** These systems use multi-sensor arrays and transmit raw or lightly processed data to the cloud for complex analysis. This is an established industry practice and lacks edge-centric novelty.
*   **Academic Edge-TinyML SHM (2024-2026):** Recent literature extensively documents the combination of accelerometers, strain gauges, and environmental sensors with on-device machine learning inference. This combination *alone* is no longer sufficiently novel for a strong patent claim.

**Our Position:** Our novelty does not stem from simply putting ML on an edge device. Instead, we **layer specific, mathematically rigorous methods on top of a common hardware baseline**. The non-obviousness lies in the *integration* of these mathematical methods (wavelets, changepoint detection, Mahalanobis distance) into a cohesive, causally-linked processing chain operating under extreme power constraints.

---

## 2. Our Six Core Innovations

### Innovation 1: Event-Triggered Wavelet-Based Acoustic Detection
*   **Prior Art:** Relies on continuous Fast Fourier Transform (FFT) analysis (which is power-hungry and loses time-domain precision) or simple amplitude thresholding.
*   **Our Method:** A low-power hardware interrupt triggers the execution of a Daubechies wavelet transform (DWT) on a dedicated microcontroller burst-compute core.
*   **Non-Obviousness:** The specific selection of a wavelet basis (e.g., Daubechies db4) is mathematically matched to the morphology of a concrete micro-fracture transient. Performing this strictly on-device, only upon a hardware interrupt, optimizes both time-frequency resolution and power consumption in a way that continuous FFT cannot.
*   **Specific Claim Language:** "...executing a wavelet transform (e.g., Continuous or Discrete Wavelet Transform using a Daubechies basis) on a dedicated microcontroller core only upon detection of a threshold-crossing acoustic transient..."

### Innovation 2: Causal Temporal-Correlation via Sequential Changepoint Detection
*   **Prior Art:** Uses simple thresholds or fixed timing rules (e.g., 'did the strain return to the baseline within 5 seconds?').
*   **Our Method:** The Page-Hinkley test mathematically evaluates the strain reading immediately following a wavelet-confirmed acoustic transient to confirm a *permanent regime shift* in the structural baseline.
*   **Non-Obviousness:** The integration of the causal chain—where a specific acoustic morphology (identified via wavelet) triggers a sequential statistical test (Page-Hinkley) to accumulate evidence of a permanent strain shift—creates a robust, integrated filter against false positives that standard independent thresholds lack.
*   **Specific Claim Language:** "...applying a sequential changepoint detection algorithm (Page-Hinkley test or CUSUM) to a physical strain reading following said acoustic transient to mathematically identify a permanent regime shift..."

### Innovation 3: Self-Calibrating Structure-Specific Thermal Model
*   **Prior Art:** Applies a generic, textbook thermal-expansion coefficient offset to strain readings.
*   **Our Method:** A 14-day autonomous, on-device regression phase learns the *structure-specific* thermal coefficient by correlating temperature and strain variations during the node's initial deployment.
*   **Non-Obviousness:** The shift from a hardcoded, generic offset to an autonomous, unattended machine-learning calibration phase that produces a localized, learned coefficient significantly improves accuracy for diverse structures without requiring manual tuning by structural engineers.
*   **Specific Claim Language:** "...compensating said strain reading using a structure-specific thermal coefficient derived via autonomous on-device regression during an initial calibration phase..."

### Innovation 4: Multivariate Sensor-Fault Discrimination via Mahalanobis Distance
*   **Prior Art:** Generally assumes sensors are healthy, or relies on simple min/max out-of-bounds checks.
*   **Our Method:** Uses multivariate covariance-based detection to measure the inter-channel correlation between acoustic, strain, and tilt readings. A breakdown in this correlation (measured via Mahalanobis distance) flags sensor degradation.
*   **Non-Obviousness:** Utilizing a single mathematical structure (Mahalanobis distance from a continuously updated joint-distribution model) to simultaneously perform structural anomaly detection *and* discriminate sensor hardware health is a highly efficient, non-obvious application of multivariate statistics on edge devices.
*   **Specific Claim Language:** "...evaluating multivariate consistency across acoustic, strain, and tilt readings using Mahalanobis distance to distinguish sensor hardware degradation from genuine structural distress..."

### Innovation 5: Event-Triggered Adaptive Sampling
*   **Prior Art:** Employs continuous high-rate sampling, draining battery life.
*   **Our Method:** Maintains a low-power listening mode where a hardware interrupt triggers high-rate burst sampling only during the acoustic event and the subsequent causal strain window.
*   **Non-Obviousness:** The integration of the adaptive sampling rate with the causal correlation chain (the acoustic trigger explicitly defining and opening the strain monitoring window) represents a novel power-management strategy tailored specifically for structural health monitoring.
*   **Specific Claim Language:** "...maintaining an acoustic sensing channel in a low-power listening state and... initiating high-rate burst sampling only upon detection..."

### Innovation 6: Multi-Node Cross-Validation Consensus
*   **Prior Art:** Relies on cloud-based arbitration or isolated single-node decisions.
*   **Our Method:** Utilizes distributed, on-device consensus via a local LoRa mesh network. A node generates a provisional alert and waits for corroboration from a neighboring node before confirming the anomaly.
*   **Non-Obviousness:** Implementing a provisional/confirmed two-stage alert protocol directly on the edge mesh, entirely removing the dependency on centralized cloud processing for false-positive arbitration.
*   **Specific Claim Language:** "...initiating a wireless alert transmission only when the changepoint confidence score exceeds a predetermined threshold... and is corroborated by a neighboring node..."

---

## 3. Primary Independent Claim (Full Text)

> An edge-computed method for structural anomaly detection, comprising: maintaining an acoustic sensing channel in a low-power listening state and executing a wavelet transform (e.g., Continuous or Discrete Wavelet Transform using a Daubechies basis) on a dedicated microcontroller core only upon detection of a threshold-crossing acoustic transient; applying a sequential changepoint detection algorithm (Page-Hinkley test or CUSUM) to a physical strain reading following said acoustic transient to mathematically identify a permanent regime shift in the structural baseline; compensating said strain reading using a structure-specific thermal coefficient derived via autonomous on-device regression during an initial calibration phase; evaluating multivariate consistency across acoustic, strain, and tilt readings using Mahalanobis distance to distinguish sensor hardware degradation from genuine structural distress; and initiating a wireless alert transmission only when the changepoint confidence score exceeds a predetermined threshold, devoid of sensor fault indication.

---

## 4. Dependent Claims

*   **Claim 2 (Wavelet Basis):** The method of claim 1, wherein the wavelet transform utilizes a Daubechies basis (e.g., db4 or db6) mathematically matched to the expected morphological characteristics of a concrete structural micro-fracture acoustic emission.
*   **Claim 3 (Changepoint Specification):** The method of claim 1, wherein the sequential changepoint detection algorithm comprises a Page-Hinkley test or a Cumulative Sum (CUSUM) test utilizing dynamic thresholding parameters derived from the local structural noise floor.
*   **Claim 4 (Calibration Refresh):** The method of claim 1, further comprising the periodic re-derivation and updating of the structure-specific thermal coefficient at defined intervals (e.g., seasonal re-fits) to maintain long-term compensation accuracy.
*   **Claim 5 (Consensus):** The method of claim 1, wherein the wireless alert transmission is maintained in a provisional state and withheld from final transmission to a central server until a statistically correlated structural anomaly is independently reported and confirmed by at least one adjacent sensing node within a defined time window via a localized mesh network.
*   **Claim 6 (Fault Localization):** The method of claim 1, wherein the multivariate consistency evaluation further comprises decomposing the Mahalanobis distance to determine individual per-channel contributions, thereby localizing the specific degrading sensor hardware.

---

## 5. Phase 2 Dependent Claims (Future Filing)

These claims cover future stretch enhancements and should be reserved for continuation filings once the baseline prototype is protected.

*   **Claim 7 (Physics-Informed Residual):** The method of claim 1, further comprising executing a lightweight physics-based beam-deflection model on the edge device to predict an expected strain value for a given load, and utilizing the residual difference between the expected strain and observed strain as a primary anomaly indicator.
*   **Claim 8 (Dempster-Shafer Fusion):** The method of claim 1, wherein the confidence scores from the acoustic, strain, and tilt subsystems are fused utilizing Dempster-Shafer evidence combination, formally representing and resolving uncertainty and conflicting data prior to alert initiation.

---

## 6. Comparison Table

| Feature | Standard Edge | Cloud SHM | Academic TinyML SHM | Our System (ATF V-2.1) |
| :--- | :--- | :--- | :--- | :--- |
| **Data Processing** | Minimal (Thresholds) | Centralized (Cloud Server) | Edge ML (Neural Nets) | **Dual-Core Edge Math (Wavelet + Changepoint)** |
| **Sensor Inputs** | Single (Accel/Vibration) | Multi-Sensor Array | Multi-Sensor Array | **Multi-Sensor (Acoustic + Strain + IMU + Env)** |
| **Acoustic Processing** | N/A or Simple Amplitude | High-res FFT in Cloud | On-device FFT / MFCCs | **Event-Triggered Daubechies Wavelet Transform** |
| **False-Positive Filter**| None (High False Alarm) | Cloud-based Arbitration | Neural Net Classification | **Causal Chain (Acoustic Trigger → Strain Changepoint)** |
| **Thermal Compensation**| Ignored or Hardcoded | Handled in Cloud Analytics | Generic Textbook Offset | **Autonomous On-Device Regression (Self-Learning)** |
| **Sensor Health** | Ignored | Cloud-based Analytics | Rarely Addressed | **On-Device Mahalanobis Multivariate Consistency** |
| **Evidence Combination**| N/A | Bayesian / Cloud Logic | Simple Weighted Average | **Dempster-Shafer Belief Fusion (Phase 2)** |

---

## 7. Claim Strength Assessment

| Claim | Focus | Novelty | Non-Obviousness | Technical Defensibility | Commercial Value |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Primary Independent** | Integrated Pipeline | High | High (Specific integration) | High (Math-based, traceable) | Very High (Solves false positives) |
| **2. Wavelet Basis** | Acoustic Profiling | Medium | High (Specific application) | High (Code verifiable) | High (Power efficiency) |
| **3. Changepoint** | Causal Correlation | High | High (Causal linkage) | High (Statistical proof) | Very High (Core filter) |
| **4. Thermal Model** | Self-Calibration | High | Medium (Auto-regression) | High (Demonstrable code) | High (Reduces deployment cost) |
| **5. Mesh Consensus** | Validation | Medium | High (Edge-only logic) | Medium (Network dependent) | High (System reliability) |
| **6. Sensor Health** | Mahalanobis Use | High | High (Dual-purpose math) | High (Math-based) | High (Maintenance efficiency) |

---

## 8. Filing Strategy Notes

1.  **Provisional Patent Filing:** A provisional patent application (PPA) is highly recommended *before* any public disclosure, academic publication, or open-source release of the codebase or hardware schematics. This secures the priority date.
2.  **Focus of the Primary Claim:** The primary independent claim MUST focus on the *integrated method* (the causal chain from acoustic interrupt to wavelet to changepoint to Mahalanobis). Claiming any individual mathematical piece (like just "using wavelets on edge") is too broad and likely unpatentable over existing academic literature. The novelty is the combination.
3.  **Dependent Claims:** The dependent claims (Claims 2-6) serve to protect specific implementations and fallbacks in case the primary claim is challenged or overly narrowed during examination.
4.  **Phase 2 Claims:** Claims 7 and 8 (Physics-Informed Residual and Dempster-Shafer) represent future enhancements. They can be introduced in the provisional filing as potential embodiments and formalized later via continuation-in-part (CIP) filings once the Phase 2 development is complete.
