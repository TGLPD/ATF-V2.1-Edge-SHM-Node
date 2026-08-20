## The Core Novelty & Innovation

Current Structural Health Monitoring (SHM) solutions fail in one of three ways:

1. **Single-sensor edge devices** (like standard accelerometers) trigger constant false alarms from routine environmental noise like heavy traffic or wind.
2. **Enterprise multi-sensor systems** stream raw, uncompressed telemetry to cloud servers, requiring high-bandwidth connections, high monthly cloud costs, and heavy power draw.
3. **Recent academic edge-TinyML systems** (2024–2026) have already begun combining accelerometers, strain gauges, and BME280-style environmental sensors with on-device inference. This pattern is now documented and is **not, by itself, sufficient novelty for a new filing.**

Our project's highly defensible patent position rests on **six specific, mathematically rigorous mechanisms** layered on top of the common hardware baseline:

### 1. Acoustic Emission (Micro-Fracture Detection) via Wavelet Transforms

- **The Problem:** Continuous high-rate FFT sampling of the acoustic channel drains batteries, and FFT only provides frequency averages, making it poor at identifying short, non-stationary transient "snaps" characteristic of micro-fractures.
- **Our Innovation:** The acoustic channel sits in a low-power interrupt mode. A threshold-crossing transient wakes a dedicated DSP core to run a **Continuous or Discrete Wavelet Transform (CWT/DWT)** using a specific wavelet basis (e.g., Daubechies or Morlet). This isolates the exact time and frequency of a fracture snap, significantly improving detection accuracy over background noise.

### 2. Causal Temporal-Correlation via Sequential Changepoint Detection

- **The Problem:** Existing fusion approaches evaluate sensor readings as a static snapshot. Furthermore, basic causal checks like "did strain return to baseline in T seconds" are mathematically weak and vulnerable to noise.
- **Our Innovation:** The system requires a time-ordered causal sequence: a wavelet-confirmed acoustic snap must trigger an immediate shift in strain. To prove this shift is genuine plastic deformation rather than elastic noise, the device runs a **sequential changepoint detection algorithm (Page-Hinkley test or CUSUM)** continuously on the post-event strain signal. This mathematically confirms a permanent regime shift in the structure.

### 3. Physics-Informed Residual Modeling (The "Digital Twin" Cross-Check)

- **The Problem:** Pure ML models don't know *why* a reading is normal or abnormal, just that it's statistically unusual. This makes them black boxes that structural engineers often mistrust.
- **Our Innovation:** The device runs a lightweight physical model of the structure (a simplified beam-deflection approximation). It infers a load event (e.g., a truck crossing) from the acoustic/vibration signature and predicts the *expected* strain. The anomaly signal is the **residual**: the gap between physics-predicted strain and actual measured strain. A large residual indicates strain changed *more than physics dictates it should have* for that load, providing an explainable, highly defensible anomaly metric.

### 4. Self-Calibrating, Structure-Specific Thermal Model

- **The Problem:** Standard systems use generic thermal-offset formulas. Real structures vary wildly in thermal expansion based on material batch, geometry, and constraints.
- **Our Innovation:** During an initial on-device calibration window (1–2 weeks), the firmware maps natural temperature swings against strain to fit a structure-specific regression coefficient. This replaces generic approximations with a highly accurate, on-device learned thermal profile.

### 5. Multivariate Sensor-Fault Discrimination via Mahalanobis Distance

- **The Problem:** Degrading sensors (corroding strain gauges, drifting IMUs) produce signatures that look identical to structural damage, causing false alarms.
- **Our Innovation:** Instead of checking independent thresholds, the node treats all sensor readings as a single multivariate point and measures its **Mahalanobis distance** from the learned "normal" distribution. This detects when the *relationship* between channels breaks down (e.g., strain jumps but acoustics are completely silent). A high Mahalanobis distance without a corresponding physical trigger flags a **sensor-health fault**, routing to maintenance rather than sounding a structural alarm. Additionally, **Dempster-Shafer Evidence Theory** is used to combine conflicting sensor signals rigorously.

### 6. Multi-Node Cross-Validation Consensus (Optional Scale-Up)

- **The Problem:** Even an advanced single node can be fooled by a highly localized event (like a bird strike or direct impact).
- **Our Innovation:** Where a second node is deployed, a local anomaly is broadcast over the LoRa mesh. The alert is finalized only if a neighboring node independently reports a correlated anomaly within an agreed time window, forming a distributed consensus without a cloud server.

## Comparison: Industry Standard vs. Published Academic Edge-SHM vs. Our Node

| **Feature** | **Standard Edge Sensor** | **Enterprise Cloud SHM** | **Published Academic Edge-TinyML SHM** | **Our Advanced Node Architecture** |
|---|---|---|---|---|
| **Data Processing** | Basic thresholding | Cloud server / heavy AI | On-device TinyML (heuristic) | **On-device Wavelet Transforms, CUSUM, & Physics Residuals** |
| **Sensor Inputs** | Single (Accelerometer) | Multi-sensor array | Fused (accel/strain/temp) | Fused (Piezo + Strain + IMU + Temp), **causally time-ordered** |
| **Acoustic Processing**| N/A | Continuous streaming | Continuous FFT | **Event-triggered Wavelet Transform (CWT/DWT)** |
| **False-Positive Filter**| None | Manual/cloud rules | Multi-sensor thresholding | **Changepoint detection + Physics Residual + Consensus** |
| **Thermal Comp.** | None | Filtered in cloud | Fixed/generic formula | **Self-calibrated, structure-specific regression** |
| **Sensor Health** | Not addressed | Not addressed | Not addressed | **Mahalanobis Distance multi-variate correlation check** |

## Your Unique Selling Proposition (USP)

> **"The first self-contained structural health monitor that mathematically confirms anomalies through wavelet-isolated acoustic transients, sequential changepoint detection, and physics-informed residuals—while distinguishing sensor faults via Mahalanobis distance—all operating indefinitely off-grid without cloud dependency."**

## How This Translates to Patent Claims

The claims below are restructured around the rigorous mathematical mechanisms introduced, guaranteeing significant distance from existing "TinyML sensor fusion" prior art.

### Primary Independent Claim (The Core Method)

> *An edge-computed method for structural anomaly detection, comprising: maintaining an acoustic sensing channel in a low-power listening state and executing a wavelet transform (e.g., Continuous or Discrete Wavelet Transform) on a dedicated microcontroller core only upon detection of a threshold-crossing acoustic transient; applying a sequential changepoint detection algorithm to a physical strain reading following said acoustic transient to mathematically identify a permanent regime shift in the structural baseline; generating a physics-informed residual by comparing the measured strain against an expected strain derived from an onboard, lightweight physical approximation model; evaluating multivariate consistency across acoustic, strain, and tilt readings using Mahalanobis distance to distinguish sensor hardware degradation from genuine structural distress; and initiating a wireless alert transmission only when the physics-informed residual and changepoint confidence score exceed a predetermined threshold, devoid of sensor fault.*

### Secondary Dependent Claims (Specific Implementations)

- **Claim 2 (Wavelet Basis Selection):** The method of Claim 1, wherein the wavelet transform utilizes a specific basis function (e.g., Daubechies or Morlet) matched to the expected morphological profile of micro-fracture acoustic emissions.
- **Claim 3 (Changepoint Statistics):** The method of Claim 1, wherein the sequential changepoint detection algorithm comprises a Cumulative Sum (CUSUM) control chart or Page-Hinkley test evaluated continuously against the post-event strain signal.
- **Claim 4 (Dempster-Shafer Evidence Combination):** The method of Claim 1, wherein the probability of structural damage is derived by combining conflicting sensor evidence using Dempster-Shafer evidence theory rather than a simple weighted average.
- **Claim 5 (Self-Calibration Refresh):** The method of Claim 1, wherein the on-device thermal compensation coefficient is periodically re-derived at defined intervals to account for seasonal or long-term drift.
- **Claim 6 (Multi-Node Consensus):** The method of Claim 1, further comprising withholding final alert transmission until a second independently operating node reports a correlated anomaly within a defined time window via a low-power mesh network.
