---
title: Calculating orbital Hall conductivity
description: Calculating the orbital Hall conductivity of RuO2 using a Wannier tight-binding Hamiltonian
date: 2026-10-16
topic: Spintronics & Orbitronics
format: markdown
tags: [spintronics, orbitronics]
---

In spintronics, spin angular momentum is used as the information carrier. In recent years, it is widely proposed to use orbital angular momentum as the alternative information carrier.

## Linear response theory

Within linear response theory [[Zelezny2017](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.95.014403)], the formula to calculate OHC is

$$
\sigma^{\gamma}_{\alpha\beta}
  = 2e^{2}\hbar \int_{\mathrm{BZ}} \frac{d^{3}k}{(2\pi)^{3}}
    \sum_{n}^{\mathrm{occ}} \sum_{m}^{\mathrm{unocc}}
    \operatorname{Im}\!\left[
      \langle n\mathbf{k} | \hat{j}^{\gamma}_{\alpha} | m\mathbf{k} \rangle
      \langle m\mathbf{k} | \hat{v}_{\beta} | n\mathbf{k} \rangle
    \right]
    \frac{\Gamma^{2} - \left(\varepsilon_{n\mathbf{k}} - \varepsilon_{m\mathbf{k}}\right)^{2}}
         {\left[\left(\varepsilon_{n\mathbf{k}} - \varepsilon_{m\mathbf{k}}\right)^{2} + \Gamma^{2}\right]^{2}},
$$

where $$\hat{j}^{\gamma}_{\alpha} = \tfrac{1}{2}\{\hat{L}_{\gamma}, \hat{v}_{\alpha}\}$$ is the orbital current operator, $$\hat{v}_{\alpha}$$ is the velocity operator, and $$\Gamma$$ is a constant band broadening. Following the Wannier90 convention, $$\alpha$$ is the direction of the current, $$\beta$$ is the direction of the applied electric field, and $$\gamma$$ is the polarization of the current; e.g., $$\sigma^{z}_{xy}$$ is a $$z$$-polarized current flowing along $$x$$ in response to a field along $$y$$. The same formula gives the spin Hall conductivity (SHC) when $$\hat{L}_{\gamma}$$ is replaced by $$\hat{S}_{\gamma} = \hat{\sigma}_{\gamma}/2$$. Both are given in units of $$(\hbar/e)\,\mathrm{S/cm}$$.

## Wannier tight-binding Hamiltonian

The Wannier tight-binding Hamiltonian is built using Wannier90 with VASP. The essential input parameters are

```text
begin projections
Ru:d
O:p
end projections

num_bands = 44
num_wann  = 44
num_iter  = 0                ! projected (Lowdin) WFs, no maximal localization

guiding_centres  = .true.
write_hr         = .true.
write_tb         = .true.    ! H(R) and r(R) -> wannier90_tb.dat
write_xyz        = .true.
write_u_matrices = .true.    ! U(k), Bloch -> Wannier basis
spinors          = .true.
spn_formatted    = .true.
```

Note that we do not perform the maximal localization step (`num_iter = 0`), so the Wannier functions keep the character of the projected atomic orbitals. This makes it straightforward to construct the orbital angular momentum operator $$\hat{L}$$ in the atomic basis. If the maximal localization step is performed, $$\hat{L}$$ must be rotated from the Bloch basis to the MLWF basis using the unitary matrices $$U^{(\mathbf{k})}$$ from Wannier90 (`write_u_matrices = .true.`).

## Build orbital angular momentum operator

We approximate $$\hat{L}$$ by the atomic angular momentum operator on each site, which is on-site ($$\mathbf{R} = 0$$) and block diagonal in atoms.

**1. $$\hat{L}$$ in the complex spherical-harmonic basis.** For a shell with angular momentum $$l$$, in the basis $$\lvert l,m\rangle$$ with $$m = -l, \dots, l$$ and in units of $$\hbar$$,

$$
\langle l,m'|\hat{L}_z|l,m\rangle = m\,\delta_{m'm}, \qquad
\hat{L}_+|l,m\rangle = \sqrt{l(l+1) - m(m+1)}\,|l,m+1\rangle,
$$

$$
\hat{L}_x = \tfrac{1}{2}\left(\hat{L}_+ + \hat{L}_-\right), \qquad
\hat{L}_y = \tfrac{1}{2i}\left(\hat{L}_+ - \hat{L}_-\right).
$$

**2. Rotate to the Wannier90 real orbitals.** Each real orbital is written as $$\lvert r_j\rangle = \sum_m T_{jm}\lvert l,m\rangle$$, using the Wannier90 orbital order:

| $$l$$ | real orbital | in terms of $$\lvert l,m\rangle$$ |
|---|---|---|
| 1 | $$p_z$$ | $$\lvert 1,0\rangle$$ |
| 1 | $$p_x$$ | $$(\lvert 1,-1\rangle - \lvert 1,1\rangle)/\sqrt{2}$$ |
| 1 | $$p_y$$ | $$i(\lvert 1,-1\rangle + \lvert 1,1\rangle)/\sqrt{2}$$ |
| 2 | $$d_{z^2}$$ | $$\lvert 2,0\rangle$$ |
| 2 | $$d_{xz}$$ | $$(\lvert 2,-1\rangle - \lvert 2,1\rangle)/\sqrt{2}$$ |
| 2 | $$d_{yz}$$ | $$i(\lvert 2,-1\rangle + \lvert 2,1\rangle)/\sqrt{2}$$ |
| 2 | $$d_{x^2-y^2}$$ | $$(\lvert 2,-2\rangle + \lvert 2,2\rangle)/\sqrt{2}$$ |
| 2 | $$d_{xy}$$ | $$i(\lvert 2,-2\rangle - \lvert 2,2\rangle)/\sqrt{2}$$ |

The matrix elements in the real-orbital basis are then

$$
\langle r_i|\hat{L}_a|r_j\rangle = \sum_{m,m'} T^{*}_{im}\,\langle l,m|\hat{L}_a|l,m'\rangle\,T_{jm'}, \qquad a = x, y, z.
$$

**3. Assemble the full spinor basis.** With `spinors = .true.`, Wannier90 orders the basis as atom → orbital → spin (↑, ↓), giving 44 Wannier functions:

| Atom | Orbitals | Wannier function indices |
|---|---|---|
| Ru1 | $$d$$ (5 × 2 spins) | 1–10 |
| Ru2 | $$d$$ (5 × 2 spins) | 11–20 |
| O1 | $$p$$ (3 × 2 spins) | 21–26 |
| O2 | $$p$$ (3 × 2 spins) | 27–32 |
| O3 | $$p$$ (3 × 2 spins) | 33–38 |
| O4 | $$p$$ (3 × 2 spins) | 39–44 |

Since $$\hat{L}$$ acts as the identity on spin,

$$
\hat{L}_a = \bigoplus_{\text{atoms}} \left[ L^{(l)}_a \otimes \mathbb{1}_{2} \right].
$$

## OHC and SHC of RuO2

The calculated OHC and SHC of RuO2 are shown in the figure below.

<img src="/assets/img/RuO2_shc_ohc.png" alt="SHC and OHC of RuO2"
     class="img-fluid d-block mx-auto" width="600" />

The results are consistent with Fig. 1(g) in [[Zhang2026](https://journals.aps.org/prl/abstract/10.1103/lf8d-26b6)].

## OHC and SHC of Pt

In spintronics, Pt is the prototypical heavy metal with large spin-orbit coupling. In a bilayer structure Pt/FM, Pt generates a large spin current flowing into the FM layer through the spin Hall effect (SHE). Here we emphasize one computational detail regarding the velocity operator.

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

The calculated OHC and SHC of Pt are shown in the figure below.

<img src="/assets/img/Pt_shc_ohc.png" alt="SHC and OHC of Pt"
     class="img-fluid d-block mx-auto" width="600" />

The results are consistent with Fig. 2(b)(e) in [[Go2024](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.109.174435)].

The comparison between the $$\partial H/\partial k$$ and full velocity is shown in the figure below. We see that the full velocity operator must be included to predict the OHC correctly.

<img src="/assets/img/Pt_ohc_velocity.png" alt="OHC of Pt using ∂H/∂k and full velocity"
     class="img-fluid d-block mx-auto" width="600" />