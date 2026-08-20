
## 1. The Foil Strain Gauge (The Sensor)

Imagine taking a thick rubber band and stretching it. As it gets longer, it also gets thinner in the middle. Wire acts the same way. When a wire gets stretched and becomes thinner, electricity has a harder time flowing through it. We call this an increase in **electrical resistance**.

A foil strain gauge is just a microscopic metal wire zig-zagging back and forth on a tiny sticker.

You glue this sticker firmly onto the steel beam or concrete pillar you want to monitor. When the heavy beam bends even a fraction of a millimeter, it stretches the sticker. That stretches the zig-zag wire, making it slightly thinner, which increases its electrical resistance.

## 2. The Wheatstone Bridge (The Detective)

Here is the problem: the beam might only bend by a microscopic amount, causing an incredibly tiny change in resistance. Your ESP32 microcontroller cannot detect a change that small directly.

To solve this, we use a **Wheatstone bridge** — a diamond-shaped circuit of four resistors.

- When all four resistors are exactly equal, the electricity is perfectly balanced, and exactly **zero volts** flow across the middle of the diamond.
- We replace *one* of those four resistors with our strain gauge sticker.
- Now, when the beam bends and the sticker stretches, its resistance changes slightly. This unbalances the perfect diamond.
- Because the scale is tipped, a tiny trickle of voltage (a few millivolts) spills across the middle of the circuit.

The Wheatstone bridge essentially acts as a magnifying glass, turning a tiny change in *resistance* into a tiny change in *voltage*.

## 3. The HX711 Load-Cell Amplifier (The Translator)

We now have a tiny trickle of voltage, but it is still too weak — often just a few thousandths of a volt. If you feed that directly into the ESP32, it will just read it as zero.

This is where the **HX711** (or ADS1115) comes in:

1. **The Amplifier:** It acts like a megaphone, boosting that tiny millivolt signal so it is large enough to measure accurately.
2. **The ADC (Analog-to-Digital Converter):** The HX711 takes that boosted physical voltage and translates it into a clean, precise digital number (like `14052`) that the ESP32 can easily read.

## 4. From a Fixed Formula to a Self-Calibrating Thermal Model (New)

The original design used a textbook thermal-offset formula to strip out heat-related expansion before feeding strain into the ML model. The problem: every structure's actual thermal response depends on its specific material batch, mounting geometry, and constraints — a generic formula is always an approximation.

Instead, the node now runs a **calibration phase** during its first 1–2 weeks of deployment:

1. It logs paired (temperature, raw strain) samples continuously as the structure naturally heats up during the day and cools at night.
2. It fits a simple regression between temperature and strain across that window, producing a thermal-expansion coefficient *specific to this exact structure and mounting point* — not a generic material constant from a textbook.
3. Going forward, this learned coefficient — not a fixed formula — is what gets subtracted from raw strain before the reading is compensated and fed to the fusion model.
4. The node periodically re-runs a lightweight version of this fit (e.g., once a season) to catch any drift in the structure's thermal behavior over time — such as a support becoming looser or a bonding layer aging.

This turns thermal compensation from "one-size-fits-all math" into a **structure-specific, self-learned baseline**, which is both more accurate in practice and a stronger technical differentiator than a fixed offset formula.

## 5. Causal Correlation Handoff (New)

Once strain is thermally compensated, it isn't evaluated alone. Whenever the acoustic channel (see the main workflow document) detects a transient, the system watches this compensated strain signal for a defined window afterward. If the strain fails to return to its pre-event baseline in that window, the pairing is treated as a confirmed fracture candidate — not the strain reading in isolation, and not the acoustic transient in isolation.

### Putting it All Together

When a heavy truck drives over your monitored bridge:

1. The bridge bends, stretching the **strain gauge** sticker.
2. The stretched wire changes its resistance, unbalancing the **Wheatstone bridge** to produce a tiny voltage.
3. The **HX711** boosts that tiny voltage and converts it into a digital number.
4. The node subtracts its **self-learned thermal offset** for the current temperature.
5. The compensated reading returns to baseline within the correlation window → logged as harmless elastic flex, no alert.

When a genuine fracture occurs instead, the acoustic channel wakes the system, the strain reading fails to recover within the same window, and the two channels together — not either one alone — confirm the anomaly.
