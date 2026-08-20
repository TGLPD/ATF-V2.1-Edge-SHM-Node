In the original version of project, the TinyML model was basically a static calculator: it took a single snapshot of the sensors, ran some math, and guessed if the bridge was failing.

But because we have upgraded to a **causal, event-driven system** to secure the patent, the TinyML model is no longer just looking at snapshots. It acts as an active detective analyzing a timeline of events.

## 1. The Calibration Learner (Regression Modeling)

When you first clamp the node to a structure, it doesn't know how that specific steel or concrete reacts to the hot midday sun.

- **What TinyML does:** For the first two weeks, a lightweight regression algorithm runs in the background, plotting the exact relationship between the BME280 temperature sensor and the HX711 strain gauge.
    
- **The Output:** It builds a custom mathematical curve (e.g., "At 35°C, this specific joint naturally expands by X amount"). Once trained, it uses this curve to automatically subtract thermal expansion from all future strain readings.
    

## 2. The Acoustic Profiler (Audio Classification)

When the piezo sensor detects a loud, high-frequency noise and wakes the system up, the node needs to know _what_ made that noise. Was it a micro-fracture inside the concrete, or did a truck just kick a loose piece of gravel against the pillar?

- **What TinyML does:** It analyzes the Fast Fourier Transform (FFT) of the audio signal. A rock hitting steel produces a different acoustic waveform and frequency spread than high-carbon steel snapping under tension.
    
- **The Output:** The model classifies the audio signature. If it confirms the sound is a material fracture, it authorizes the system to move to the next step.
    

## 3. The Causal Time-Judge (Time-Series Analysis)

This is your strongest patent claim. Once a fracture sound is confirmed, the model doesn't just look at the strain gauge once; it watches the strain data _over a specific window of time_ (e.g., 5 seconds).

- **What TinyML does:** It tracks the timeline of the deformation. It asks: _Did the strain spike and then bounce right back to normal (elastic vibration from heavy traffic)? Or did it spike and stay permanently shifted (plastic deformation from structural failure)?_
    
- **The Output:** The model only flags a "Structural Anomaly" if the sequence is strictly: **Confirmed Crack Sound -> Permanent Strain Shift**.
    

## 4. The Fault Discriminator (Logic Rules)

Machine learning models are great at finding contradictions. The model is trained to recognize when the sensors are lying to each other.

- **What TinyML does:** It looks for impossible physics. If the strain gauge suddenly reports a massive, permanent bend in the steel, but the acoustic model heard _absolutely zero_ fracture noises beforehand, the model knows steel doesn't just quietly bend in half.
    
- **The Output:** Instead of triggering a bridge collapse alarm, the model flags a "Hardware Fault" (e.g., the glue holding your strain gauge is peeling off).
    

**Summary:** The TinyML model isn't just a single algorithm anymore. It is a combination of a **regression model** (to learn the heat curve), an **audio classifier** (to verify the crack sound), and a **time-series logic gate** (to prove cause and effect).