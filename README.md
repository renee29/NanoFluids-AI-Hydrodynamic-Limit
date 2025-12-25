# NanoFluids-AI: Hydrodynamic Limit Validation Suite

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Python 3.8+](https://img.shields.io/badge/Python-3.8%2B-green.svg)
![Status: Validated](https://img.shields.io/badge/Status-Validated-brightgreen.svg)
![Physics: Molecular Dynamics](https://img.shields.io/badge/Physics-Molecular_Dynamics-orange.svg)

## 1. Scientific Overview

This repository contains the **Hydrodynamic Limit Validation Suite** for the NanoFluids-AI framework. It serves as a rigorous computational laboratory to demonstrate the emergence of continuum mechanics (Navier-Stokes equations) from discrete, chaotic molecular dynamics (MD) in nanoconfined regimes.

The core objective is to solve the **forward structural problem**: verifying that a microscopic Hamiltonian system, when driven out of equilibrium, recovers the exact constitutive laws of continuum mechanics without *a priori* assumptions. This establishes the mathematical baseline required for data-driven operator discovery.

### Key Capabilities
*   **Micro-to-Macro Lifting:** Implementation of the **Irving-Kirkwood stress formalism** to rigorously map discrete particle positions/momenta to continuous stress tensor fields.
*   **Continuum Recovery:** Quantitative validation of the Navier-Stokes momentum balance ($\nabla \cdot \boldsymbol{\tau} = -\rho \mathbf{f}$) in the presence of thermal fluctuations.
*   **Transport Coefficients:** First-principles extraction of effective shear viscosity from non-equilibrium steady states.

---

## 2. Visual Demonstration: The Emergence of Flow

The animation below visualises the central physical phenomenon: the transition from thermal equilibrium to a structured hydrodynamic flow.

<p align="center">
  <img src="multiphysics_publication.gif" alt="Hydrodynamic Limit Convergence" width="100%">
</p>

> **Figure 1: Real-time convergence to the Hydrodynamic Limit.**
> *   **Left Panel (Equilibrium):** The system exhibits pure Brownian motion. The time-averaged velocity profile is zero, validating the thermostat's neutrality.
> *   **Right Panel (Driven):** Under an external body force, a parabolic velocity profile emerges from the chaos. The red markers (MD data) converge to the analytical Navier-Stokes solution (black dashed line) as the flow develops.
> *   **Centre:** The velocity profile evolution tracks the convergence of the Reynolds number ($Re$) and Kinetic Energy ($KE$) towards a steady state.

### Quantitative Animation Check
To ensure visual fidelity, the visualisation suite performs a cross-check against the analytical solution:

<p align="center">
  <img src="velocity_profiles_final.png" alt="Velocity Profile Validation" width="60%">
</p>
<p align="center"><em><strong>Figure 2:</strong> Detailed comparison of the time-averaged velocity profile from the animation (points) vs. the analytical Poiseuille flow solution (dashed line).</em></p>

---

## 3. Theoretical Framework & Methodology

### 3.1. Physics Engine Architecture
The simulation implements a canonical **Lennard-Jones (LJ) fluid** confined in a planar nanochannel. The physics engine is designed to preserve symplectic structure while managing thermal dissipation:

*   **Interaction Potential:** Truncated and shifted Lennard-Jones potential ($r_c = 2.5\sigma$) for fluid-fluid interactions, coupled with a WCA (Weeks-Chandler-Andersen) repulsive potential for wall confinement.
*   **Integration Scheme:** **Velocity-Verlet algorithm**. This symplectic, time-reversible integrator ensures long-term energy stability ($O(\Delta t^2)$), crucial for distinguishing physical dissipation from numerical drift.
*   **Spatially-Selective Thermostat:** To avoid damping the hydrodynamic modes in the bulk, we implement a **Langevin thermostat** strictly localised at the boundaries (walls). This acts as a heat sink and enforces no-slip boundary conditions via friction, leaving the bulk fluid Newtonian (NVE-like).

### 3.2. Mathematical Formalism (The Lifting Map)
To bridge the scale gap, we compute the local stress tensor $\boldsymbol{\tau}(\mathbf{r})$ using the **Irving-Kirkwood (IK) formalism**. Unlike simple virial estimators, the IK method distributes the interaction contribution along the bond vector connecting particle pairs, ensuring that the differential momentum balance is satisfied locally:

$$ \frac{\partial}{\partial t}(\rho \mathbf{u}) + \nabla \cdot (\rho \mathbf{u} \otimes \mathbf{u}) = \nabla \cdot \boldsymbol{\tau} + \mathbf{f}_{ext} $$

In the steady-state Poiseuille geometry, this simplifies to a linear stress gradient: $$ \frac{\partial \tau_{xy}}{\partial y} = -\rho f_x $$

---

## 4. Validation Results

### A. Constitutive Law Recovery
We performed a production run ($N=800$, $t=200\tau$) to validate the recovery of the Navier-Stokes constitutive laws.

<p align="center">
  <img src="nanofluids_continuum_validation_poiseuille.png" alt="Stress and Velocity Profiles" width="90%">
</p>

*   **Panel A (Shear Stress):** The measured shear stress profile $\tau_{xy}(y)$ (blue points) exhibits a perfect linear dependence on channel position ($R^2 > 0.99$). The slope matches the theoretical prediction ($-\rho f_x$) within $<5\%$ error, validating the momentum balance at the molecular scale.
*   **Panel B (Velocity Field):** The velocity profile $u_x(y)$ recovers the classical parabolic shape. Fitting this profile allows for the extraction of the **effective shear viscosity** $\eta$, providing a direct link between molecular parameters ($\epsilon, \sigma$) and continuum coefficients.

### B. Consistency & Conservation Diagnostics
To ensure the robustness of the simulation, we perform rigorous diagnostic tests.

<p align="center">
  <img src="diagnostic_energy_conservation_nve.png" width="48%" alt="Energy Conservation">
  <img src="diagnostic_thermostat_spatial_profile.png" width="48%" alt="Thermostat Profile">
</p>

*   **Energy Conservation (Left):** In the NVE ensemble (thermostat off), the total energy drift is $< 10^{-4}$ over $10^5$ steps, confirming the stability of the integrator.
*   **Thermostat Topology (Right):** The friction coefficients $\gamma_x, \gamma_y$ are spatially plotted. Note the sharp localisation of $\gamma_x$ (blue) at the walls to enforce no-slip, while the bulk remains friction-free ($\gamma_x=0$).

---

## 5. Repository Structure

```text
.
├── continuum_validation_poiseuille.py    # MAIN ENGINE: MD simulation & validation logic
├── physics_diagnostics.py                # TEST SUITE: Conservation laws & consistency checks
├── visualize_multiphysics_publication.py # VISUALIZATION: Generates the comparative GIF
├── requirements.txt                      # Python dependencies
├── CITATION.cff                          # Citation metadata
└── README.md                             # Documentation
```

---

## 6. Usage

### Prerequisites
The suite requires a standard scientific Python stack. Install dependencies via:

```bash
pip install -r requirements.txt
```

### Execution

**1. Run the Main Validation:**
Executes the MD simulation, computes Irving-Kirkwood tensors, and generates the stress/velocity validation plot.
```bash
python continuum_validation_poiseuille.py
```

**2. Run Diagnostic Suite:**
Performs energy conservation tests (NVE) and verifies thermostat topology.
```bash
python physics_diagnostics.py
```

**3. Generate Visualisation:**
Renders the high-resolution GIF comparing equilibrium vs. driven states.
```bash
python visualize_multiphysics_publication.py
```

---

## 7. Citation

If you use this validation suite or the Irving-Kirkwood implementation in your research, please cite:

```bibtex
@software{nanofluids_ai_hydrodynamic_limit_2025,
  author = {NanoFluids-AI Research Team},
  title = {NanoFluids-AI: Hydrodynamic Limit Validation Suite},
  version = {1.0.0},
  year = {2025},
  url = {https://github.com/renee29/NanoFluids-AI-Hydrodynamic-Limit}
}
```

## License
This project is licensed under the MIT License.
