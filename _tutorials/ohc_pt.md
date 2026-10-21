---
title: Orbital and spin Hall conductivity of Pt
description: Calculating the orbital Hall conductivity of Pt using a Wannier tight-binding Hamiltonian
date: 2026-10-21
topic: Spintronics & Orbitronics
format: markdown
tags: [spintronics, orbitronics]
---

In spintronics, Pt is the prototypical heavy metal with large spin-orbit coupling. In a bilayer structure Pt/FM, Pt generates a large spin current flowing into the FM layer through the spin Hall effect (SHE). The formula and setup are similar to the [RuO2 tutorial]({{ '/tutorials/ohc/' | relative_url }}). Here we emphasize one computational detail regarding the velocity operator.

$$
\hat{\mathbf{v}} = \frac{i}{\hbar}\left[ \hat{H}, \hat{\mathbf{r}} \right]
$$

$$
\mathbf{v}_{nm}(\mathbf{R}) = \frac{i}{\hbar} \left( \mathbf{R} H_{nm}(\mathbf{R}) + \sum_{\mathbf{R}'} \sum_{j} \left[ H_{nj}(\mathbf{R}') \mathbf{r}_{jm}(\mathbf{R}-\mathbf{R}') - \mathbf{r}_{nj}(\mathbf{R}') H_{jm}(\mathbf{R}-\mathbf{R}') \right] \right)
$$

where $$H_{nm}(\mathbf{R}) = \langle n\mathbf{0}\vert\hat{H}\vert m\mathbf{R}\rangle$$ and $$\mathbf{r}_{nm}(\mathbf{R}) = \langle n\mathbf{0}\vert\hat{\mathbf{r}}\vert m\mathbf{R}\rangle$$.

In some calculations, only the first term is considered. In momentum space, it is equivalent to

$$
\mathbf{v}_{nm}(\mathbf{k}) = \frac{1}{\hbar}\frac{\partial H_{nm}(\mathbf{k})}{\partial \mathbf{k}}, \qquad
H_{nm}(\mathbf{k}) = \sum_{\mathbf{R}} e^{i\mathbf{k}\cdot\mathbf{R}} H_{nm}(\mathbf{R}).
$$



## Calculation of OHC and SHC

The calculated OHC and SHC of Pt are shown in the figure below.

<img src="/assets/img/Pt_shc_ohc.png" alt="SHC and OHC of Pt"
     class="img-fluid d-block mx-auto" width="600" />

The results are consistent with Fig. 2(b)(e) in [[Go2024](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.109.174435)].

The comparison between the $$\partial H/\partial k$$ and full velocity is shown in the figure below. We see that the full velocity operator must be included to predict the OHC correctly.

<img src="/assets/img/Pt_ohc_velocity.png" alt="OHC of Pt using ∂H/∂k and full velocity"
     class="img-fluid d-block mx-auto" width="600" />
