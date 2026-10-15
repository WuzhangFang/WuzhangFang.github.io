---
title: Linear spin-wave theory for a Heisenberg ferromagnet
description: Derive the magnon dispersion with the Holstein–Primakoff transformation, from the spin Hamiltonian to a few lines of NumPy.
date: 2026-10-15
topic: Magnons & spin waves
format: markdown
level: Beginner
tags: [magnons, spin-waves, holstein-primakoff]
---

Magnons are the elementary excitations of an ordered magnet: quantized spin waves. This tutorial walks through the standard linear spin-wave calculation for a Heisenberg ferromagnet. It's the starting point for almost everything else, from fitting exchange constants to neutron data to computing magnon–phonon coupling from first principles.

**You'll need:** basic quantum mechanics (ladder operators) and a little Python.

## The model

Start from the nearest-neighbor Heisenberg Hamiltonian

$$
H = -J \sum_{\langle ij \rangle} \mathbf{S}_i \cdot \mathbf{S}_j, \qquad J > 0,
$$

where each bond $\langle ij \rangle$ is counted once. With $J > 0$, the classical ground state has all spins aligned along $\hat{z}$ with energy $E_0 = -J S^2 N z/2$, where $N$ is the number of sites and $z$ the number of nearest neighbors.

## Holstein–Primakoff bosons

Spin operators are awkward to diagonalize directly, so we map them onto bosons $a_i$ that count deviations from full alignment:

$$
S_i^z = S - a_i^\dagger a_i, \qquad
S_i^+ = \sqrt{2S - a_i^\dagger a_i}\; a_i \approx \sqrt{2S}\, a_i, \qquad
S_i^- \approx \sqrt{2S}\, a_i^\dagger .
$$

The approximation keeps only the leading order in $1/S$. That's valid at low temperature, when few magnons are excited.

Write the dot product as $\mathbf{S}_i \cdot \mathbf{S}_j = S_i^z S_j^z + \tfrac{1}{2}\left(S_i^+ S_j^- + S_i^- S_j^+\right)$ and keep terms up to quadratic order in the bosons:

$$
\mathbf{S}_i \cdot \mathbf{S}_j \approx S^2 - S\left(a_i^\dagger a_i + a_j^\dagger a_j - a_i^\dagger a_j - a_j^\dagger a_i\right).
$$

## Going to momentum space

Use the Fourier transform $a_i = N^{-1/2} \sum_{\mathbf{k}} e^{i\mathbf{k}\cdot\mathbf{r}_i} a_{\mathbf{k}}$ and define the structure factor

$$
\gamma_{\mathbf{k}} = \frac{1}{z} \sum_{\boldsymbol{\delta}} e^{i\mathbf{k}\cdot\boldsymbol{\delta}},
$$

where $\boldsymbol{\delta}$ runs over the $z$ nearest-neighbor vectors. The Hamiltonian becomes diagonal:

$$
H = E_0 + \sum_{\mathbf{k}} \hbar\omega_{\mathbf{k}}\, a_{\mathbf{k}}^\dagger a_{\mathbf{k}},
\qquad
\boxed{\hbar\omega_{\mathbf{k}} = zJS\left(1 - \gamma_{\mathbf{k}}\right)}
$$

For a 1D chain with lattice constant $a$, $z = 2$ and $\gamma_k = \cos ka$, so

$$
\hbar\omega_k = 2JS\left(1 - \cos ka\right).
$$

## The long-wavelength limit

Expanding $\gamma_{\mathbf{k}}$ for small $k$ gives a quadratic dispersion, $\hbar\omega_{\mathbf{k}} \approx D k^2$, which is the hallmark of a ferromagnet. Antiferromagnets, by contrast, have linear dispersion. The **spin-stiffness** $D$ depends on the lattice:

| Lattice     | $z$ | $D$                 |
| ----------- | --- | ------------------- |
| 1D chain    | 2   | $JSa^2$             |
| square      | 4   | $JSa^2$             |
| simple cubic | 6   | $JSa^2$             |
| triangular  | 6   | $\tfrac{3}{2}JSa^2$ |

Here $a$ is the nearest-neighbor distance.

## In code

The general formula translates directly into NumPy: give it the neighbor vectors and a list of $\mathbf{k}$-points.

```python
import numpy as np

def magnon_energy(kpts, neighbors, J=1.0, S=1.0):
    """Linear spin-wave energy  hbar*omega_k = z J S (1 - gamma_k).

    kpts      : (Nk, d) array of wavevectors
    neighbors : (z, d) array of nearest-neighbor vectors
    """
    z = len(neighbors)
    gamma = np.cos(kpts @ neighbors.T).mean(axis=1)  # real for centrosymmetric lattices
    return z * J * S * (1.0 - gamma)

# 1D chain, a = 1
k = np.linspace(-np.pi, np.pi, 201)[:, None]
E = magnon_energy(k, neighbors=np.array([[1.0], [-1.0]]))
print(E.max())  # 4JS at the zone boundary
```

Because $\gamma_{\mathbf{k}}$ is an average over neighbors, the same function works for any Bravais lattice. Just change `neighbors`.

## Where to go next

- The companion notebook, [Plotting magnon dispersions with NumPy]({{ '/tutorials/magnon-dispersion-notebook/' | relative_url }}), plots these dispersions along high-symmetry paths and checks the $Dk^2$ limit numerically.
- Adding magnetic anisotropy opens a gap at $\mathbf{k} = 0$. Antiferromagnets need a Bogoliubov transformation, and their dispersion becomes linear.
- In real materials, $J$ and $D$ come from first-principles calculations, for example from total energies of spin spirals or from magnetic force theorems.
