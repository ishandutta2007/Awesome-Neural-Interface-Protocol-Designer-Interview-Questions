# 1. Neuroscience & Signal Fundamentals

Foundational knowledge every neural interface designer needs, even if you're primarily a systems/protocol person — you must understand what you're actually transmitting.

---

### 1.1 🟢 What is the difference between an action potential (spike) and a local field potential (LFP)?

An action potential is the brief (~1 ms) all-or-nothing voltage spike produced when a single neuron fires, typically recorded with high-impedance microelectrodes placed very close (< 100 µm) to the soma or axon. An LFP is the low-frequency (roughly < 300 Hz) component of the extracellular signal, reflecting the summed synaptic and dendritic activity of many nearby neurons rather than any single cell firing. In practice: spikes require high sampling rates (20–30 kHz) and give single-neuron resolution but are unstable over time as electrodes shift; LFPs need lower sampling rates, are more stable long-term, and carry population-level information useful for decoding "state" (e.g., movement intent) rather than fine motor detail.

---

### 1.2 🟢 What is ECoG and how does it differ from EEG and intracortical recording?

Electrocorticography (ECoG) records electrical activity directly from the cortical surface using electrode grids placed under the dura (or sometimes epidurally), typically after a craniotomy. Compared to EEG, ECoG has much higher spatial resolution and bandwidth (up to a few hundred Hz vs. EEG's ~40 Hz practical limit) and a far better signal-to-noise ratio because it isn't attenuated by the skull and scalp. Compared to intracortical (penetrating) electrodes, ECoG is less invasive (no tissue penetration) but sacrifices single-neuron resolution — it captures population-level activity from many thousands of neurons per electrode. It's often viewed as the practical middle ground for chronic clinical BCIs where long-term stability matters more than single-unit resolution.

---

### 1.3 🟡 Explain the tradeoffs between invasive and non-invasive neural recording modalities from a protocol/system design perspective.

Non-invasive (EEG, fNIRS) systems avoid surgical risk and regulatory burden but must deal with low SNR, motion artifacts, and lower information bandwidth — so the protocol layer needs aggressive filtering, longer calibration, and often accepts lower command throughput (e.g., discrete selections rather than continuous control). Invasive systems (intracortical arrays, ECoG) deliver much higher bandwidth and SNR, enabling continuous, high-degree-of-freedom control (e.g., multi-axis robotic arm control), but the protocol must be designed around the constraints of implanted hardware: strict power budgets, wireless telemetry through tissue, hermeticity, and much stricter fault-tolerance requirements because failures can have surgical/clinical consequences. As a designer, this tradeoff drives almost every downstream decision — sampling rate, compression, error-correction strategy, and even choice of wireless band.

---

### 1.4 🟡 What is signal drift in chronic neural implants, and why does it matter for protocol design?

Signal drift refers to the gradual change in recorded signal characteristics over days to months, caused by factors like glial scarring (encapsulation) around electrodes, micromotion between the device and tissue, and electrode impedance changes. It matters for protocol design because a static, one-time-calibrated decoding or communication scheme will degrade in accuracy over time. This pushes designers toward protocols that support periodic recalibration, adaptive filtering, and even in-field firmware/model updates — meaning your communication protocol needs a reliable, secure channel for pushing updated decode parameters into the implant without requiring another surgery.

---

### 1.5 🟢 What is the "cocktail party problem" in the context of multi-electrode neural recordings, and how is it addressed?

It's the challenge of separating overlapping signals from multiple simultaneously active neurons (or artifact sources) recorded on the same or nearby electrodes — analogous to isolating one voice in a noisy room. In neural recording, this is addressed primarily through **spike sorting** (clustering spike waveforms by shape to attribute them to distinct neurons) and, for LFP/ECoG, through spatial filtering techniques like common average referencing, independent component analysis (ICA), or beamforming to separate sources based on their spatial signature across the electrode array.

---

### 1.6 🟡 What is the difference between open-loop and closed-loop neural interface systems?

An open-loop system only reads (or only writes) — e.g., a BCI that decodes intended movement and outputs a command, with no feedback path bringing information back into the nervous system. A closed-loop system creates a feedback cycle: sensory or state information is fed back into the nervous system (often via electrical stimulation) based on decoded output or an external sensor, in real time. Closed-loop systems (e.g., responsive neurostimulation for epilepsy, or bidirectional prosthetics providing tactile feedback) impose much stricter latency requirements on the protocol, since the loop delay directly affects both usability and, in stimulation contexts, safety — excessive delay can cause instability or make control feel unnatural.

---

### 1.7 🟡 What is charge-balanced stimulation and why is it critical in neural stimulation protocols?

Charge-balanced stimulation ensures that the net charge delivered to tissue over a stimulation pulse (or pulse train) is zero — typically via biphasic pulses where a cathodic phase is followed by an equal-and-opposite anodic phase. This matters because any net DC charge injected into tissue drives irreversible Faradaic electrochemical reactions at the electrode-tissue interface, which can cause electrode corrosion, gas generation, pH changes, and tissue damage. Any protocol that controls a stimulator must enforce charge balancing at the hardware and firmware level (not just "best effort" in software), often with dedicated safety circuitry (e.g., blocking capacitors, active charge-balancing circuits) as a redundant safeguard.

---

### 1.8 🟢 What is neural plasticity, and why does it matter for long-term BCI protocol design?

Neural plasticity is the brain's capacity to reorganize its connections and functional mapping in response to experience, injury, or repeated use of a device. For BCIs, it matters both as a challenge and an opportunity: decode models can become stale as the brain's representation of a task shifts with practice (challenge), but plasticity also means users can learn to produce more decodable, consistent neural patterns over time with the right feedback (opportunity). This motivates protocols that support co-adaptive learning — decoders that update alongside the user rather than assuming a fixed brain-to-signal mapping.

---

### 1.9 🟡 What are common sources of artifact in neural recordings, and how would you address them at the protocol/system level (not just algorithmically)?

Common sources include: electromagnetic interference from nearby electronics, muscle activity (EMG) bleeding into scalp/surface recordings, cardiac (ECG) artifact, motion/mechanical artifact from cable or electrode movement, and power-line noise (50/60 Hz). While filtering (notch filters, ICA) handles a lot of this in software, system-level mitigations are just as important: shielded/twisted-pair wiring or optical isolation for cabled systems, synchronous sampling with reference electrodes, mechanical strain relief to reduce motion artifact, and — for wireless implants — careful RF front-end design and shielding to prevent the telemetry link itself from injecting noise into the recording channels.

---

### 1.10 🔴 How would you decide the appropriate sampling rate and resolution (bit depth) for a new implantable recording channel, given power constraints?

This is a classic bandwidth-vs-power tradeoff. Start from the signal you actually need: spike detection requires capturing waveform shape up to ~5–7 kHz of meaningful content, so by Nyquist you'd want ≥ 15–20 kHz sampling (commonly 20–30 kHz in practice to preserve waveform shape for spike sorting); if you only need LFP or multi-unit activity power, you can sample at 1–2 kHz or even lower after analog anti-alias filtering. Bit depth is driven by dynamic range needs — typically 10–16 bits is enough to resolve spike amplitudes (tens to hundreds of µV) against the noise floor without wasting power on unnecessary ADC precision. Since power scales roughly with sampling rate × bit depth × channel count, and channel counts on modern arrays can be in the hundreds to thousands, I'd push hard on doing as much filtering/compression in the analog/mixed-signal front end as possible (e.g., on-chip spike detection thresholding so only detected events are digitized and transmitted, rather than streaming raw waveforms) — this is often the single biggest lever for power budget in a chronic implant.

---

### 1.11 🟢 What is impedance in the context of a neural electrode, and why do engineers monitor it over time?

Electrode impedance is the opposition to current flow at the electrode-tissue interface, measured typically at 1 kHz. It's monitored because it's a proxy for electrode health: rising impedance over time typically indicates encapsulation (glial scarring), corrosion, or a failing connection, while very low impedance can indicate a short or electrode delamination. Tracking impedance trends lets a system flag degrading channels, deprioritize them in the decode algorithm, or trigger a maintenance/replacement decision — this is often built into the device's periodic self-test protocol.

---

### 1.12 🟡 What is the difference between a motor imagery BCI and a P300/SSVEP-based BCI?

Motor imagery BCIs decode the neural signature of a user imagining a movement (e.g., imagining moving their left hand), which produces measurable changes in sensorimotor rhythms (mu/beta band power) — these require significant user training and calibration but allow relatively naturalistic, continuous control. P300 and SSVEP BCIs instead rely on evoked responses to external stimuli: P300 uses the brain's characteristic response to an unexpected/target stimulus in an "oddball" paradigm (e.g., a flashing letter grid), while SSVEP uses the brain's tendency to entrain to a flickering visual stimulus at its exact frequency. Both are more of a "selection" paradigm (choosing from a discrete set of options) than continuous control, but they require less user training and often achieve higher accuracy per selection — a protocol designer needs to know which paradigm they're serving because it fundamentally changes the data rate, latency tolerance, and stimulus-timing precision required of the system.

---

### 1.13 🔴 Why is the "gamma band" (roughly 30–150+ Hz) particularly important in ECoG-based BCI decoding, and what does that imply for your data pipeline?

High-gamma activity (roughly 70–150 Hz) correlates strongly with local cortical population firing rate and has become one of the most reliable ECoG features for decoding movement intent, speech, and other high-level cognitive states — more so than raw broadband voltage or lower-frequency bands in many paradigms. This has direct implications for the data pipeline: your sampling rate must be high enough to resolve up to 150+ Hz with margin (typically ≥ 1 kHz), your analog front end needs sufficient bandwidth and low enough noise floor to preserve this relatively low-amplitude, high-frequency signal, and your feature-extraction stage likely needs efficient onboard spectral estimation (e.g., a lightweight FFT or filter-bank running on an embedded DSP) rather than shipping raw broadband data off-device, both for power reasons and for latency in a closed-loop system.

---

### 1.14 🟢 What is the blood-brain barrier and why is it relevant to implant material selection?

The blood-brain barrier is a highly selective semipermeable boundary formed by tight junctions between endothelial cells lining brain capillaries, which restricts the passage of most substances (including many drugs and immune cells) from blood into brain tissue. It's relevant to implant design because materials in direct contact with brain tissue must be biocompatible and non-reactive to avoid triggering an inflammatory/immune response that could compromise the barrier, cause glial scarring around the electrode, or lead to chronic inflammation — all of which degrade recording quality over time and can pose safety risks.

---

### 1.15 🟡 Explain the difference between single-unit activity (SUA) and multi-unit activity (MUA) and when you'd design a system around each.

SUA is the isolated spiking activity of an individual, identifiable neuron, obtained through careful spike sorting. MUA is the combined, unsorted spiking activity of multiple nearby neurons detected on one electrode (typically just threshold-crossing events without sorting). SUA gives the highest-resolution information but requires more processing (sorting) and is more sensitive to electrode drift, since losing track of a single unit's waveform breaks the decode; MUA is more robust to drift and requires far less onboard compute, making it attractive for chronic implants where you want stability and low power over years, even if you sacrifice some decoding precision. Many production BCI systems default to MUA-based or threshold-crossing-rate decoding for exactly this robustness reason.

---

### 1.16 🟢 What are common electrode materials used in neural interfaces, and what properties make them suitable?

Common materials include platinum-iridium (Pt-Ir), iridium oxide (IrOx), titanium nitride (TiN), and increasingly thin-film polymer-based electrodes with conductive polymer coatings (e.g., PEDOT:PSS). Suitable materials need high charge-injection capacity (able to deliver stimulation current without exceeding the water electrolysis window and damaging tissue or the electrode), long-term biocompatibility and corrosion resistance, low and stable impedance, and mechanical properties compatible with the surrounding tissue (increasingly, flexible/compliant substrates are favored over rigid ones to reduce chronic micromotion injury).

---

### 1.17 🟡 What is "foreign body response" and how does it manifest as a signal degradation problem over the life of an implant?

Foreign body response is the body's immune/inflammatory reaction to an implanted device, involving macrophage and microglial activation followed by encapsulation of the device in a glial scar (astrocytes forming a dense sheath around the electrode). Over weeks to months, this scar tissue increases the effective distance between the electrode and viable neurons and raises electrode impedance, which together reduce signal amplitude and SNR — this is one of the primary reasons chronic intracortical recordings often show declining spike yield over the first several months to a year post-implant. It's a major driver of research into flexible, smaller-footprint, and bioactive-coated electrodes designed to minimize this response.

---

### 1.18 🟡 What is the difference between afferent and efferent neural signals, and why does that distinction matter for a bidirectional neural interface?

Efferent signals travel from the central nervous system outward to control muscles or organs (motor commands); afferent signals travel inward, carrying sensory information (touch, proprioception, pain) toward the central nervous system. A bidirectional neural interface — like a prosthetic hand that both reads motor intent and delivers tactile feedback — needs to record efferent (or cortical motor) signals on one channel/pathway and *deliver* stimulation into afferent pathways (or somatosensory cortex) on another, and the protocol must strictly separate and time-multiplex these read and write operations to avoid stimulation artifact corrupting the recording channels, which is a very real and common engineering problem (stimulation artifact can saturate recording amplifiers for milliseconds to seconds if not handled carefully).

---

### 1.19 🔴 A colleague proposes decoding hand movement intent directly from raw broadband voltage rather than extracting spike or band-power features first. What are the tradeoffs?

Decoding from raw broadband voltage (feeding largely unprocessed time-series data into a model, often a deep neural network) can capture information that hand-crafted features like spike rate or band power might discard, and it removes the engineering burden of building and maintaining a feature-extraction pipeline — this has shown strong results in recent research. The tradeoffs: raw broadband data is much higher bandwidth, so it's more expensive to transmit (especially wirelessly from an implant) and more expensive to process, likely requiring either on-device inference (power/compute budget problem) or high-bandwidth low-latency telemetry off-device (power/RF budget problem). It's also less interpretable and harder to validate/certify for a medical device pathway, since regulators generally want some ability to understand and bound model behavior. In practice, I'd frame this as a spectrum rather than a binary choice — e.g., on-device dimensionality reduction (like band-power features) as a middle ground that keeps most of the informative signal while controlling data rate, reserving raw-broadband streaming for research/offline model development rather than the always-on production path.

---

### 1.20 🟢 What does "channel count" mean in the context of a neural interface, and why does scaling it up create non-trivial engineering challenges beyond "just add more wires"?

Channel count refers to the number of independent recording (or stimulation) sites on an electrode array. Scaling channel count from, say, 100 to 1,000+ isn't just a matter of adding wires — each additional channel adds to the power budget (more amplifiers, ADCs), the data bandwidth that must be processed or transmitted, the physical routing complexity (especially for implantable devices where feedthrough count is limited by hermetic packaging), and the thermal budget (implanted electronics have strict heat dissipation limits, typically under 1°C tissue temperature rise, to avoid thermal damage). This is why high-channel-count systems increasingly push toward on-chip integration (CMOS-based neural probes with amplification and even spike detection built directly into the probe) rather than routing every raw channel off-chip.
