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

The default configuration implements a ring attractor in the **absence of recurrent E-E and I-I connectivity** (amp_EE = amp_II = 0). A stable bump is sustained purely through the E→I→E feedback loop: a like-to-like excitatory drive onto inhibitory neurons (w_IE) combined with local disinhibitory feedback back onto excitatory neurons (w_EI). This produces a Mexican-hat effective connectivity in the excitatory population, supporting a localized, translation-invariant activity bump.

---

## Parameters

| Parameter | Description |
|---|---|
| Spiking | Toggles between deterministic rate dynamics and Poisson spiking |
| Neurons | Number of neurons per population (powers of 2, 8–1024) |
| Slow | Playback-speed toggle: off = 1× (real-time), on = 0.01× (slow motion) |
| 1D / 2D | Switches between a 1D ring and a 2D toroidal sheet of neurons |
| σ | Gaussian width of each weight kernel (log scale, 0.001–10) |
| Amplitude | Peak synaptic weight strength |
| Invert | Flips the weight profile (peaked → trough) |
| Binarize | Thresholds the weight profile to 0 or 1 via a Heaviside function |
| envelope | Gaussian window applied to all weight kernels |
| β_E, β_I | Uniform excitatory drive to E and I populations |
| Adaptation (w_a, τ_a) | Spike-frequency adaptation on E cells: strength w_a, time constant τ_a |
| Reset (T_reset) | Periodically re-seeds the bump every T_reset seconds |
| Reset σ / amp / dur | Shape of the reset pulse: Gaussian width (fraction of N), peak amplitude, and duration (ms) |

---

## Playback speed

The **Slow** toggle (top row) controls how much simulation time advances per rendered frame. Off runs at full speed (1×); on drops to 0.01× for slow-motion inspection of fast transients. It scales the number of integration steps the background worker runs per UI update, so it changes apparent speed without altering the underlying dynamics (dt is unchanged).

---

## Adaptation and periodic reset

**Adaptation** adds a slow self-inhibitory current to the excitatory population (strength `w_a`, time constant `τ_a`), modeling spike-frequency adaptation. This can destabilize a static bump and induce traveling-wave or drifting dynamics.

**Periodic reset** re-establishes a clean bump at the ring/sheet center every `T_reset` seconds: activity is briefly silenced, then a Gaussian drive pulse is injected into the excitatory population only. The pulse is fully adjustable:

- **σ** — Gaussian width, as a fraction of the population size
- **amp** — peak excitatory drive at the center
- **dur** — how long the pulse is held (milliseconds)

---

## 2D / torus mode

Toggling **2D** arranges the neurons on an `L × L` toroidal sheet (capped at 64×64) with isotropic Gaussian kernels and 2D FFT-based convolution. Firing rates and synaptic conductances are shown as viridis heatmaps instead of line plots.

---

## Presets

Presets capture the full parameter set and can be named and saved. They are **shared**: the page loads a curated `presets.json` from this repository on startup, so everyone who opens the page sees the same set of presets.

Saving is **read-only for visitors** — anyone can load the shared presets, but only the repository owner can publish new ones. Publishing is automatic: with a GitHub token configured (see below), clicking **Save** commits the updated `presets.json` to the repo via the GitHub API, and it goes live for everyone within about a minute.

### Publishing presets (owner only)

1. Create a GitHub **fine-grained personal access token** scoped to only this repository, with **Contents: Read and write** permission.
2. Click the **"Presets"** section header in the GUI and paste the token when prompted. It is stored only in your browser's `localStorage` — never committed and never present in the page source.
3. From then on, saving or deleting a preset automatically commits `presets.json` to the repository.

Visitors without a token never see this prompt's effect and cannot modify the shared presets; their own saves stay local to their browser.
