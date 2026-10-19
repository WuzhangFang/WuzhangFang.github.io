---
title: Calculating magnetic anisotropy energy using Wannier interpolation
description: Calculating the magnetic anisotropy energy of FePt using a Wannier tight-binding Hamiltonian with on-site spin-orbit coupling
date: 2026-10-19
topic: Magnetism
format: markdown
tags: [magnetism, magnetic anisotropy, wannier90]
---

The calculation of magnetic anisotropy energy (MAE) requires a very dense k-point mesh and can be very time-consuming. Using a Wannier Hamiltonian with a much smaller basis set can significantly reduce the computational time. In this tutorial, we use FePt, which has a large perpendicular magnetic anisotropy (PMA), as an example to illustrate this approach.

## Installation of Wannier90 and the VASP2Wannier90 interface

The installation of Wannier90 can be found at [http://www.wannier.org/download/](http://www.wannier.org/download/). After compiling the code, we have a static library `libwannier.a` and two executables `postw90.x` and `wannier90.x`. Wannier90 should be used in conjunction with an electronic structure code that calculates the eigenstates of the Hamiltonian. Here we use VASP 5.4.4, which only supports Wannier90 up to version 2.1.0. There are some known bugs when spin-orbit coupling (SOC) is turned on; a fix is available at [https://github.com/Chengcheng-Xiao/VASP2WAN90_v2_fix](https://github.com/Chengcheng-Xiao/VASP2WAN90_v2_fix).

VASP is compiled as usual, except that we add `-DVASP2WANNIER90v2` to the `CPP_OPTIONS` tag and the path of `libwannier.a` to the `LLIBS` tag in the makefile. The newly compiled VASP has an interface to Wannier90 and can generate the input files needed by Wannier90.

## Generation of the Wannier Hamiltonian

The first step is an accurate self-consistent collinear calculation of FePt using VASP. It is better to fix the number of bands by setting `NBANDS = 32`. After the self-consistent calculation, we analyze the projected band structures and density of states to determine the energy window and the band characters.

The second step is generating the input files for Wannier90 using VASP. An example `INCAR` is

```text
SYSTEM = FePt
ISTART = 1
ICHARG = 1
NPAR = 8
LREAL = .FALSE.
NELM = 1              # maximum number of electronic SC steps
PREC = Accurate
ALGO = None
ENCUT = 400
KSPACING = 0.15
ISPIN = 2
LORBIT = 11
MAGMOM = 3 0
LMAXMIX = 4
GGA_COMPAT = .FALSE.
NBANDS = 32
ISYM = 2
EDIFF = 1E-07
ISMEAR = -5
LWANNIER90 = .TRUE.
LWAVE = .FALSE.
LCHARG = .FALSE.
```

We also prepare two Wannier90 input files, `wannier90.up.win` and `wannier90.dn.win`, which contain the following tags:

```text
! for generating the necessary files using VASP
num_bands = 32
num_wann  = 18
begin projections
Fe: s;p;d
Pt: s;p;d
end projections

! disentanglement
dis_win_min   = -2.0d0
dis_win_max   = 50.0d0
dis_froz_min  = -2.0d0
dis_froz_max  = 20.0d0
dis_num_iter  = 8000
dis_mix_ratio = 1.0
dis_conv_tol  = 1.0e-8

! Wannierization
num_iter    = 2000
conv_tol    = 1.0e-5
conv_window = 2

write_hr        = true   ! write the Hamiltonian
write_xyz       = true
guiding_centres = true
```

Here `num_wann` is the number of Wannier functions, which equals the number of initial projections. After running VASP, the `.mmn`, `.amn`, and `.eig` files for the spin-up and spin-down channels are generated. VASP also adds the structure and k-point coordinates to the `.win` files if they are missing. It is better to let VASP do this to avoid potential mistakes. The last step is running Wannier90:

```bash
mpirun wannier90.x wannier90.up.win
mpirun wannier90.x wannier90.dn.win
```

After Wannierization, the `.chk` and `_hr.dat` files are generated. `wannier90.up_hr.dat` and `wannier90.dn_hr.dat` contain the Wannier Hamiltonian $$H_{mn}(\mathbf{R}) = \langle w_m(\mathbf{0})\lvert \hat{H} \rvert w_n(\mathbf{R})\rangle$$ for the spin-up and spin-down channels. The format of `_hr.dat` is:

- The 1st line is a comment line.
- The 2nd line is the number of Wannier functions.
- The 3rd line is the number of $$\mathbf{R}$$ lattice vectors.
- The following lines give the degeneracy of each lattice vector, 15 items per line.
- The remaining lines are the tight-binding parameters. The first three columns are the coordinates of $$\mathbf{R}$$ in terms of lattice vectors, the 4th and 5th columns are the Wannier function indices $$m$$ and $$n$$, and the last two columns are the real and imaginary parts of the matrix element.

## Calculation of magnetocrystalline anisotropy

We first add on-site SOC to the collinear Hamiltonians, with the spin quantization axis along $$x$$ and along $$z$$. The on-site SOC parameters are taken from Table 1 in [[Ke2019](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.99.054418)]. We use the Python package [wanSOC](https://github.com/Chengcheng-Xiao/wanSOC), which combines the spin-up and spin-down Hamiltonians and adds the on-site SOC to the full Hamiltonian. The script to generate the SOC Hamiltonian with spin quantization along $$x$$ is

```python
from wanSOC.helper import *
from wanSOC.hamiltonian import *

# generate full Hamiltonian from collinear Hamiltonians
hop_spinor_dn, Rlatt_dn, indR0_dn, deg_dn = read_hr('wannier90.dn_hr.dat', spin='dn')
hop_spinor_up, Rlatt_up, indR0_up, deg_up = read_hr('wannier90.up_hr.dat', spin='up')

# sanity check
assert (Rlatt_dn == Rlatt_up).all(), "Real-lattice hopping vector mismatch."
assert indR0_dn == indR0_up, "Home cell index mismatch."
assert (deg_dn == deg_up).all(), "Degeneracy mismatch."

# add hop_spinor to get full spinor Hamiltonian
hop_spinor = hop_spinor_dn + hop_spinor_up

# [orbital_type, quantization_axis, strength/2, orbital_index (start from 1)]
# here we adjust the SOC strength for the p orbitals on the Pt atom
ratio = 0.81 / 0.50  # from Table VI in PRB 99, 054418 (2019)
p_1 = 0.1972 / 2
p_2 = 2.6264 / 2 / ratio  # scaled
d_1 = 0.055 / 2
d_2 = 0.5749 / 2
q_axis = [1, 0, 0]  # spin quantization axis along x
orb_chg = [['d', q_axis, d_1, 5], ['d', q_axis, d_2, 14],
           ['p', q_axis, p_1, 2], ['p', q_axis, p_2, 11]]
full_Hsoc = get_full_Hsoc(orb_chg, hop_spinor)

# add H_soc to the on-site block of the full Hamiltonian
hop_spinor[indR0_dn] += full_Hsoc

# output final Hamiltonian
write_hr(deg_dn, Rlatt_dn, hop_spinor, Filename='wannier90_hr_X.dat')
```

The Hamiltonian with spin quantization along $$z$$ is obtained by setting `q_axis = [0, 0, 1]` and writing to `wannier90_hr_Z.dat`. With the full Hamiltonians, we compute the band structures and compare them with VASP. The agreement is excellent, as shown in the figure below.

<div class="row justify-content-center">
  <div class="col-sm-6 mt-3 mt-md-0">
    <img src="/assets/img/FePt_band_X.png" alt="FePt band structure with spin quantization along x"
         class="img-fluid d-block mx-auto" />
    <p class="text-center">(a) Spin quantization along <i>x</i></p>
  </div>
  <div class="col-sm-6 mt-3 mt-md-0">
    <img src="/assets/img/FePt_band_Z.png" alt="FePt band structure with spin quantization along z"
         class="img-fluid d-block mx-auto" />
    <p class="text-center">(b) Spin quantization along <i>z</i></p>
  </div>
</div>
<p class="text-center">Band structures of FePt calculated from the Wannier Hamiltonians (red dashed lines) and VASP (black lines).</p>

The magnetocrystalline anisotropy (MCA) is calculated as the difference between the grand canonical potentials with the magnetization along $$x$$ and $$z$$,

$$
K = \sum_{n\mathbf{k}} \left[
      \left(\varepsilon^{x}_{n\mathbf{k}} - E_F\right) f^{x}_{n\mathbf{k}}
    - \left(\varepsilon^{z}_{n\mathbf{k}} - E_F\right) f^{z}_{n\mathbf{k}}
  \right],
$$

where $$\varepsilon_{n\mathbf{k}}$$ is the band energy, $$\mathbf{k}$$ is the wave vector in the first Brillouin zone, $$n$$ is the band index, $$E_F$$ is the Fermi energy from the collinear calculation, and $$f_{n\mathbf{k}}$$ is the band occupancy. Here we use [WannierTools](https://www.wanniertools.org/) to diagonalize the Hamiltonian and a Fermi function at 100 K for the band occupancy. The calculated MCA is 2.4 meV/f.u., which is smaller than the DFT value of 2.8 meV/f.u. from VASP. Using the tetrahedron method with Blöchl corrections [[Blöchl1994](https://journals.aps.org/prb/abstract/10.1103/PhysRevB.49.16223)] to perform the Brillouin-zone integration is expected to improve the result; this is left for future investigation.
