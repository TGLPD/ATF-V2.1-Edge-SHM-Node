## 1. Wavelet Transform instead of FFT for acoustic emission (strong, moderate effort)

This is probably your single best upgrade. FFT tells you _what frequencies_ are present in a signal, averaged over the whole sampling window — it's bad at telling you _when_ a specific transient happened, because a fracture "snap" is a short, sharp, non-stationary event, not a steady periodic signal.

A **Continuous or Discrete Wavelet Transform (CWT/DWT)** decomposes the signal in both time _and_ frequency simultaneously, which is exactly suited to detecting a brief, sharp acoustic transient buried in continuous background noise (traffic, wind). This isn't just academic elegance — it's the standard tool in real acoustic-emission structural monitoring research precisely because fracture events are transient, not periodic.

- **Patent angle:** claim a specific wavelet basis (e.g., Daubechies or Morlet, chosen because it matches the known shape of AE fracture transients) applied on-device, with a specific decomposition-level threshold used to trigger the correlation window — this is much more specific than "run an FFT," and genuinely improves detection accuracy for micro-fractures over background noise, so it's not novelty for novelty's sake.

## 2. Changepoint Detection instead of a simple "return to baseline" window (strong, low effort)

Right now Innovation 1 says "strain fails to return to baseline within T seconds" — that's a blunt threshold rule. A much more rigorous approach is a **sequential changepoint detection algorithm**, such as the **Page-Hinkley test** or **CUSUM (cumulative sum control chart)**, run continuously on the strain signal.

These are statistical methods (developed originally for detecting shifts in industrial process control) that mathematically detect _when a time series has genuinely shifted to a new statistical regime_, distinguishing a real permanent shift from noise far more robustly than "did it cross a fixed threshold."

- **Patent angle:** claim the specific combination of an interrupt-triggered acoustic event with a sequential changepoint statistic applied to the post-event strain signal, where the changepoint confidence score (not just the acoustic score) feeds the final ML tensor. This turns your causal correlation logic from a simple rule into a formally grounded statistical test.

## 3. Mahalanobis Distance instead of independent per-sensor thresholds (moderate, low effort)

Your sensor-fault-discrimination logic currently compares each channel somewhat independently. A more mathematically rigorous version treats all sensor readings as a single multivariate point and measures its **Mahalanobis distance** from the learned "normal" distribution — which accounts for how the sensors _correlate_ with each other under healthy conditions, not just their individual ranges.

This matters because a subtle sensor fault often shows up not as one channel going obviously wrong, but as the _relationship_ between channels breaking down (e.g., strain and acoustic no longer correlate the way they historically have, even though both look individually "normal"). This is a much harder failure mode to catch with simple thresholds, and Mahalanobis distance is specifically designed for exactly this.

- **Patent angle:** a specific per-node, continuously-updated covariance model of "normal joint sensor behavior," used as the basis for both anomaly detection _and_ sensor-fault flagging — one mathematical structure serving both jobs is a nice, tight claim.

## 4. Physics-Informed Residual Modeling — a "digital twin" cross-check (strongest, highest effort)

This is the most powerful option and where genuinely defensible, modern patent territory is. Instead of only relying on learned ML patterns, you build a **lightweight physical model** of the structure (a simplified beam-deflection or finite-element approximation) that predicts _expected_ strain given an observed load event (inferred from the vibration/acoustic signature — e.g., a truck crossing produces an estimable load magnitude).

The anomaly signal then becomes the **residual**: the gap between what physics predicts the strain _should_ be for that load, and what the sensor actually measured. A large, persistent residual is strong evidence of real damage — because it's not just "strain moved," it's "strain moved _more than physics says it should have_ for the load applied."

- **Why this is strong:** physics-informed ML (hybrid physical-model + learned-model residual detection) is an active, fast-growing research area precisely because it fixes the biggest weakness of pure ML anomaly detection — a pure ML model doesn't know _why_ a reading is normal or abnormal, just that it's statistically unusual. A physics residual model gives you a principled, explainable "expected vs. observed" comparison, which is both more defensible technically and more convincing to engineers/inspectors who'll trust it.
- **Patent angle:** the specific method of deriving load magnitude from the acoustic/vibration signature, feeding it into a lightweight on-device physical deflection model, and using the residual between physical prediction and sensor observation — rather than raw sensor thresholds — as the primary anomaly signal. This is a genuinely distinctive claim, not a rehash of existing SHM literature.

## 5. Dempster-Shafer Evidence Theory for combining uncertain, conflicting sensor evidence (moderate-strong, moderate effort)

Instead of a single ML probability score, **Dempster-Shafer theory** lets you combine "belief" from each sensor channel (acoustic, strain, tilt, sensor-health check) in a way that explicitly represents _uncertainty and conflict_ between sources — rather than forcing everything into one blended probability. It's specifically good at situations like yours: "acoustic strongly suggests damage, but strain is ambiguous, and sensor-health is uncertain" — this framework combines that kind of mixed, partial evidence more rigorously than a simple weighted average or a single neural net output.

- **Patent angle:** claim the specific evidence-combination rule used to merge causal-correlation confidence, thermal-calibration confidence, and sensor-health confidence into a final decision — this is a more formally described decision architecture than "feed it into a tensor and get a probability out."

---

### Priority

1. **Wavelet transform for acoustic detection** — directly improves real-world accuracy on the exact failure mode (transient fracture snaps) you care about, and is a clean, specific claim.
2. **Changepoint detection (Page-Hinkley/CUSUM)** for the strain-recovery check — low effort, formalizes your strongest existing claim.
3. **Physics-informed residual modeling** — highest effort, but genuinely the strongest differentiator and the one most likely to survive an obviousness challenge, because it's not "more sensors + ML," it's a fundamentally different reasoning approach (expected-vs-observed physics residual).

Mahalanobis distance and Dempster-Shafer are both good, moderate-effort upgrades to the sensor-fault logic specifically — I'd treat them as a secondary pass once 1–3 are solid.
