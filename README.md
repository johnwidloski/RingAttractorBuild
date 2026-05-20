[README.md](https://github.com/user-attachments/files/28075216/README.md)
# Ring Attractor Network GUI

*A browser-based simulation tool for exploring excitatory-inhibitory ring attractor dynamics*

---

## Run it

Open the following URL in any modern browser:

https://johnwidloski.github.io/RingAttractor/RingNetworkGui.html

Press **Start** to begin the simulation. No installation or server required.

---

## Overview

This tool implements a one-dimensional ring attractor network intended to model head direction tuning in neural circuits. The network consists of two spatially-arranged populations — excitatory (E) and inhibitory (I) neurons — whose connectivity is defined by Gaussian weight profiles over a circular topology. Synaptic interactions are computed efficiently via FFT-based circular convolution.

The GUI runs entirely in the browser with no installation required. The simulation runs continuously in a background thread, and all parameters can be adjusted in real time.

---

## Network dynamics

Each neuron receives synaptic input from four weight kernels:

- **w_EE**: recurrent excitation (E→E)
- **w_IE**: excitatory drive onto inhibitory population (E→I)
- **w_EI**: inhibitory feedback onto excitatory population (I→E)
- **w_II**: recurrent inhibition (I→I)

Each kernel is a Gaussian profile (parameterized by σ and amplitude) centered on the presynaptic neuron's position, computed over the ring with circular boundary conditions. Kernels can be inverted (converting a peaked profile to a trough) or binarized via a Heaviside threshold.

Total synaptic input to each population is:

```
G_E = g_EE + g_EI + β_E
G_I = g_IE + g_II + β_I
```

where β_E and β_I are uniform (spatially homogeneous) drive terms. Firing rates are computed via a rectified-linear transfer function applied to G_E and G_I.

In **rate mode**, firing rates evolve deterministically. In **spiking mode**, spikes are drawn from an inhomogeneous Poisson process at each timestep (dt = 0.5 ms), producing stochastic dynamics.

The update rule follows a leaky integrator with synaptic time constant τ_s = 30 ms.

---

## Default parameters

The default configuration implements a ring attractor in the **absence of recurrent E-E and I-I connectivity** (amp_EE = amp_II = 0). A stable bump is sustained purely through the E→I→E feedback loop: broad excitatory drive onto inhibitory neurons (w_IE) combined with narrow inhibitory feedback onto excitatory neurons (w_EI, inverted profile). This produces a Mexican-hat effective connectivity in the excitatory population, supporting a localized, translation-invariant activity bump.

---

## Parameters

| Parameter | Description |
|---|---|
| Spiking | Toggles between deterministic rate dynamics and Poisson spiking |
| Neurons | Number of neurons per population (powers of 2, 8–1024) |
| σ | Gaussian width of each weight kernel (log scale, 0.001–10) |
| Amplitude | Peak synaptic weight strength |
| Invert | Flips the weight profile (peaked → trough) |
| Binarize | Thresholds the weight profile to 0 or 1 via a Heaviside function |
| β_E, β_I | Uniform excitatory drive to E and I populations |
