## Subsystem Architecture — Consolidated & Updated

This document merges and de-duplicates all prior subsystem descriptions (original workflow, self-calibration/sensor-fault updates, and the advanced signal-processing upgrades) into one current reference. Where the same idea appeared more than once under different names (e.g. "Environmental Compensation" and "Autonomous Structural Thermal Profiling" were the same mechanism), they've been merged into a single, more complete section.

---

### 1. Acoustic Emission (Micro-Fracture Detection) — Event-Triggered, Wavelet-Based

Vibration measures macro-movements; Acoustic Emission (AE) measures micro-events — the high-frequency energy released when a rigid material begins to micro-fracture.

- **The Hardware:** A high-frequency piezoelectric contact microphone clamped flush to the material, with a comparator circuit enabling low-power interrupt wake-up.
- **The Method:** The piezo channel sits in a low-power listening mode — no continuous sampling. Once a transient crosses an amplitude threshold, an interrupt wakes Core 1, which runs a **Daubechies wavelet transform** on the burst rather than a plain FFT. Because a fracture "snap" is a short, sharp, non-periodic event, a wavelet transform (which resolves both *time* and *frequency* together) pinpoints the exact instant the snap occurred far more precisely than FFT can. The system looks specifically at energy in the fastest decomposition levels (where a snap shows up) and ignores the slower levels (traffic rumble, wind). The precise snap timestamp this produces is what kicks off the correlation window on the strain channel below.

### 2. Strain and Deformation (Physical Bending) — Causally Correlated via Changepoint Detection

A structure might vibrate heavily but snap right back into place (elastic deformation). We need to know if it is permanently bending (plastic deformation) — and specifically, whether that permanent shift was *caused by* a fracture-like acoustic event, not just coincidentally nearby in time.

- **The Hardware:** A foil strain gauge attached via a Wheatstone bridge circuit and a load-cell amplifier (HX711 or ADS1115).
- **The Method:** When the wavelet detector (above) flags a snap, the system starts running a **Page-Hinkley changepoint test** on the incoming, thermally compensated strain readings. Rather than checking a single fixed threshold, this test accumulates statistical evidence over successive readings, distinguishing a genuine sustained shift in the strain baseline from ordinary sensor noise or a brief blip. If the accumulated evidence crosses its threshold within the defined post-event window, the pairing is logged as a **causally confirmed fracture candidate**. If not, it's discarded as elastic/harmless. A strain drift with no preceding acoustic transient is routed separately to the tilt/foundation check (below), not treated as a fracture.

### 3. Spatial Tilt and Settling (Inclinometer) — Long-Term Foundation Check

If a support pillar is slowly sinking, vibration and acoustic sensors won't catch it because the movement is too slow.

- **The Hardware:** A 6-axis MEMS IMU (MPU6050, or BNO085 for better long-term bias stability) reading absolute tilt to fractions of a degree.
- **The Method:** Acts as a long-term sanity check for foundation drift. Because IMU bias drift can itself mimic slow tilt, the node cross-checks the IMU's reported drift rate against its own historical noise floor — a drift rate outside the sensor's known statistical envelope is treated as a possible sensor fault (see Section 5) rather than an automatic foundation-failure alarm.

### 4. Environmental Compensation — Self-Calibrating, Structure-Specific Thermal Model

Steel and concrete expand in summer heat and contract in winter cold, shifting baseline strain readings — and every structure does this slightly differently depending on material, thickness, and sun exposure. A generic textbook thermal formula is always an approximation.

- **The Hardware:** A BME280 measuring ambient temperature and humidity.
- **The Method:** On deployment, the node enters a **14-day autonomous calibration phase**. It continuously logs paired (temperature, raw strain) samples across the structure's natural day/night thermal cycle and fits a regression between them — producing a **structure-specific thermal-expansion coefficient**, learned entirely on-device with no human input, rather than a fixed formula. This coefficient is what gets subtracted from raw strain going forward. The node periodically re-runs a lightweight version of this fit (e.g., seasonally) to catch drift in the structure's thermal behavior over time.

### 5. Sensor-Fault vs. Structural-Fault Discrimination — via Mahalanobis Distance

Existing systems generally assume their sensors are always healthy. A debonding strain gauge sticker, a degraded piezo bond, or a drifting IMU can produce readings that look exactly like real structural distress — a classic failure mode being a debonding sensor being misread as a bending bridge.

- **The Method:** Rather than checking each sensor channel independently against its own fixed threshold, the node treats a full reading (acoustic + strain + tilt) as a single multivariate point and measures its **Mahalanobis distance** from a continuously-updated model of "normal joint sensor behavior." This accounts for how the channels normally *correlate* with each other, not just their individual ranges — so it catches the subtler failure mode where one channel quietly stops agreeing with the others even though no single reading looks obviously wrong. Example: if strain drifts wildly while acoustic shows zero transients and tilt shows zero movement, the system flags a **sensor-debonding fault**, not a structural alarm — saving engineers from dispatching an emergency crew for a broken sticker, and doubling as a predictive-maintenance signal for the node itself.

### 6. Multi-Node Cross-Validation (Optional Scale-Up)

Where a second node is deployed on the same structure, a locally confirmed anomaly is held as provisional and broadcast over the LoRa mesh to nearby nodes. Only if a neighboring node independently reports a correlated anomaly within a defined time/confidence window does the system escalate to a final alert — filtering out node-local false positives (a localized impact, animal activity, or a single unit's own sensor fault) without needing a cloud arbitration layer.

---

### Phase 2 / Stretch Enhancements (Higher Effort, Highest Patent Strength)

These two are not yet required for the core working prototype but are the strongest remaining upgrades if time/resources allow — each replaces a "black box" decision step with a more rigorous, explainable one.

- **7. Physics-Informed Residual Modeling ("Digital Twin" Cross-Check):**
   Instead of relying purely on learned ML patterns, a lightweight physical model of the structure (a simplified beam-deflection approximation) predicts the *expected* strain for a given load, where the load itself is estimated from the acoustic/vibration signature (e.g., a truck crossing). The anomaly signal becomes the **residual** — the gap between physics-predicted strain and actually observed strain. A large, persistent residual is stronger, more explainable evidence of real damage than a raw statistical anomaly score, because it directly answers "did the structure respond the way physics says it should?"

- **8. Dempster-Shafer Evidence Fusion:**
   Instead of collapsing everything into one blended ML probability, this combines "belief" from each subsystem — causal-correlation confidence (Section 2), thermal-calibration confidence (Section 4), and sensor-health confidence (Section 5) — using a formal evidence-combination rule that explicitly represents uncertainty and conflict between sources (e.g., "acoustic strongly suggests damage, but sensor-health is uncertain"). This produces a more rigorously justified final decision than a single opaque probability score, and gives you a cleaner, more specific patent claim around *how* evidence is combined.

---

### What Changed From Earlier Drafts (for reference)

- "Environmental Compensation" and "Autonomous Structural Thermal Profiling" were the same mechanism described twice — merged into Section 4.
- "Sensor-Fault vs. Structural-Fault Discrimination" and "Asymmetric Sensor-Health Diagnostics" were the same mechanism described twice — merged into Section 5, now upgraded with Mahalanobis distance instead of simple per-channel thresholds.
- FFT-based acoustic detection (original) has been superseded by wavelet-based detection (Section 1).
- The simple fixed-threshold "return to baseline" strain check (original) has been superseded by Page-Hinkley changepoint detection (Section 2).
- Mahalanobis distance, physics-informed residual modeling, and Dempster-Shafer fusion — previously listed only as general "innovation ideas" — are now placed into their specific pipeline positions above.


