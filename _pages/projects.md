---
permalink: /projects/
title: "Projects"
---

## <img src="/assets/images/etx4velo-logo.png" alt="logo" style="height:50px; vertical-align:middle;"> ETX4VELO: Tracking with GNNs
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">High Energy Physics</span>
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Machine Learning</span>

Real-time particle track reconstruction from high-frequency, noisy data using ML on GPUs and FPGAs at the LHC. 

[![etx4velo-1 preview](/assets/images/etx4velo-1.png)](/assets/images/etx4velo-1.pdf)
**Figure:** Illustration of the process of moving from hits in the detector to a graph of the event.
[![etx4velo-2 preview](/assets/images/etx4velo-2.png)](/assets/images/etx4velo-2.pdf)
**Figure:** Illustration of the process of moving from the event graph to the reconstructed tracks of the event.

[Paper](https://dx.doi.org/10.1088/1748-0221/19/12/P12022), [Paper](https://arxiv.org/abs/2502.02304), [Poster](https://indico.cern.ch/event/1405026/contributions/6103447/), [Code(Python, C++, CUDA, HDL)](https://gitlab.cern.ch/gdl4hep)

## <img src="/assets/images/xim-logo.png" alt="logo" style="height:50px; vertical-align:middle;"> Traffic Anomaly Detection
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Machine Learning</span>
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Anomaly Detection</span>

Learning traffic anomalies from generative models on real-time spatio-temporal data from 125 cameras in Gothenburg, Sweden.

![xim-1](/assets/images/xim-1.png){: width="48%" style="display:inline-block; margin-right:2%" }
![xim-2](/assets/images/xim-2.png){: width="48%" style="display:inline-block" }
**Figure:** Detection of the beginning of heavy snowfall. Scenes from Camera 14 on Nov. 19, 2020, at 14:10 (left) and 14:20 (right).

[Paper](https://arxiv.org/abs/2502.01391), [Code(Python)](https://gitlab.cern.ch/fgiasemi/traffic-anomaly-detection)

## <img src="/assets/images/pxp-logo.jpg" alt="logo" style="height:50px; vertical-align:middle;"> Energy Transport in the PXP Spin Chain
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Quantum Chaos</span>
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Many-Body Physics</span>

Exploring weak ergodicity breaking in the PXP spin chain.

![pxp-1](/assets/images/pxp-1.png)
**Figure:** Spins on a chain, interacting only if they are close to each other. [Figure](https://uebungen.physik.uni-heidelberg.de/c/image/exp/d/vorlesung/20191/983/wagner_slides.pdf) by Alexander Wagner.
![pxp-2](/assets/images/pxp-2.png)
**Figure:** Observation of linear fronts in the space-time diagram of the evolution of the energy density of a PXP chain.

[Master Thesis](https://dspace.lib.ntua.gr/xmlui/bitstream/handle/123456789/55932/quantum-chaos.pdf), [Code(Julia, Python)](https://github.com/fgias/quantum-chaos)

## <img src="/assets/images/chirikov-logo.png" alt="logo" style="height:50px; vertical-align:middle;">  Chimera States in Oscillator Networks
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Nonlinear Dynamics</span>
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Complex Systems</span>

Chimera states in the leaky integrate-and-fire model of spiking neuron oscillators.

![lif-1](/assets/images/lif-1.png)
**Figure:** Development of a chimera state in a network of coupled identical oscillators, with coexisting domains of coherence and incoherence.

[Code(Python, C++, Java)](https://github.com/fgias/leaky-integrate-and-fire)

## <img src="/assets/images/xbtusd-logo.png" alt="logo" style="height:50px; vertical-align:middle;">  Exploring Trading Strategies for Crypto
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Quantitative Finance</span>
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Cryptocurrencies</span>

![xbtusd-1](/assets/images/xbtusd-1.png)
**Figure:** Illustration of buy and sell signals based on moving averages, on historical data of bitcoin prices.

[Code(Python)](https://github.com/fgias/freqtrade-strategies), [Code(Python)](https://github.com/fgias/bitcoin-backtester)

## <img src="/assets/images/cmi-logo.png" alt="logo" style="height:50px; vertical-align:middle;">  Yang–Mills Existence and Mass Gap
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Theoretical Physics</span>
<span style="background-color:#eee; border-radius:6px; padding:2px 6px; font-size: 0.9em; margin-right:6px;">Mathematical Physics</span>

Studied the *Yang–Mills existence and mass gap problem*, an open question in mathematical physics and mathematics, and one of the seven Millennium Prize Problems established by the [Clay Mathematics Institute](https://www.claymath.org/), which offers a $1M reward for a correct solution.

<div>
    \[
    \mathcal{L} = -\frac{1}{2} \text{tr}(F^2) = -\frac{1}{4} F^{a\mu\nu} F_{\mu\nu}^a
    \]
</div>
**Equation:** Lagrangian of gauge theories with a non-abelian symmetry group.

<div>
    \[
    \mathcal{L}_{\text{SM}} = \mathcal{L}_{\text{gauge}} + \mathcal{L}_{\text{fermion}} + \mathcal{L}_{\text{Higgs}} + \mathcal{L}_{\text{Yukawa}}
    \]

    \[
    \begin{aligned}
    \mathcal{L}_{\text{gauge}} &= -\frac{1}{4}G_{\mu\nu}^a G^{a\mu\nu} - \frac{1}{4}W_{\mu\nu}^i W^{i\mu\nu} - \frac{1}{4}B_{\mu\nu} B^{\mu\nu} \\
    \mathcal{L}_{\text{fermion}} &= \sum_{\psi} \bar{\psi} i\gamma^\mu D_\mu \psi \\
    \mathcal{L}_{\text{Higgs}} &= (D_\mu \phi)^\dagger(D^\mu \phi) - V(\phi), \quad V(\phi) = \mu^2 \phi^\dagger \phi + \lambda (\phi^\dagger \phi)^2 \\
    \mathcal{L}_{\text{Yukawa}} &= -\left( y_u \bar{Q}_L \tilde{\phi} u_R + y_d \bar{Q}_L \phi d_R + y_e \bar{L}_L \phi e_R + \text{h.c.} \right)
    \end{aligned}
    \]
</div>
**Equations:** The Standard Model Lagrangian \\(\mathcal{L}_{\text{SM}}\\), showing gauge, fermion, Higgs, and Yukawa sectors.

[Master Thesis](/assets/pdf/yang-mills.pdf)
