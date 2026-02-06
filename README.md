# Faraday Rotation Estimation
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An Amateur program used for estimating the loss due to the distortion by Faraday Rotation between to EME stations (DX and Home location)

## Overview

In EME communications, signal degradation often occurs due to polarization mismatch. 
This tool helps operators estimate:

	 - Spatial Rotation: Geometric polarization shift caused by the Earth's curvature between two distant stations.
	 - Faraday Rotation: The rotation of the plane of polarization as the radio wave passes through the ionosphere, influenced by the Earth's magnetic field ($B$) and Total Electron Content (TEC).

 ## Mathematical Modeling Description

### 1. Model Overview

The core objective of this project is to model the polarization state evolution in an EME link. We utilize **Jones Calculus** as the primary mathematical framework to track the complex vector transformations from the transmitter, through the lunar reflection, to the receiver.

### 2. Coordinate System & Definitions

We define a right-handed orthogonal coordinate system $(\hat{h}, \hat{v}, \hat{k})$ for each station:

- $\hat{k}$: Direction of wave propagation.
- $\hat{h}$: Local horizontal axis (Horizon).
- $\hat{v}$: Local vertical axis (Zenith).

### 3. Parameterization of Antenna Polarization

Any antenna's polarization state is represented by a normalized polarization vector $\mathbf{P}$. To maintain generality for non-standard polarizations, we use the tilt angle $\psi$ and ellipticity angle $\chi$:

The complex Jones vector $\mathbf{P}(\psi, \chi)$ is formulated as:

$$\mathbf{P} = \begin{pmatrix} P_h \\ P_v \end{pmatrix} = \begin{pmatrix} \cos\psi & -\sin\psi \\ \sin\psi & \cos\psi \end{pmatrix} \begin{pmatrix} \cos\chi \\ i \sin\chi \end{pmatrix}$$

Expanding into its complex form:

$$\mathbf{P} = \begin{pmatrix} \cos\psi \cos\chi - i \sin\psi \sin\chi \\ \sin\psi \cos\chi + i \cos\psi \sin\chi \end{pmatrix}$$

Thus, we obtain the source vector $\mathbf{P}_{TX}$ and the reference vector $\mathbf{P}_{RX}$ for the respective stations.

### 4. Channel Transfer Matrices

The propagation path involves three major coordinate and phase transformations.

#### A. Spatial Polarization Angle ($\Theta_{spatial}$)

Due to Earth's curvature, the local "vertical" definitions at two distant points do not align. We compute the **Parallactic Angle** $\nu$:

$$\nu = \arctan \left( \frac{\sin H \cos \phi}{\sin \phi \cos \delta - \cos \phi \sin \delta \cos H} \right)$$

The net spatial rotation is the sum of both stations' contributions: $\Theta_{spatial} = \nu_D + \nu_H$.

#### B. Faraday Rotation Integral ($\Omega$)

Faraday rotation is a non-reciprocal effect caused by ionospheric birefringence. The one-way rotation $\Omega$ is given by the integral:

$$\Omega = \frac{e^3}{8\pi^2 \epsilon_0 m^2 c f^2} \int_{path} N_e(s) \mathbf{B}(s) \cdot d\mathbf{s}$$

#### C. Net Rotation Matrix ($\mathbf{R}_{net}$)

The total rotation angle $\Phi_{net}$ combines both spatial and ionospheric effects:

$$\Phi_{net} = \Theta_{spatial} + (\Omega_{uplink} + \Omega_{downlink})$$

$$\mathbf{R}(\Phi_{net}) = \begin{pmatrix} \cos \Phi_{net} & -\sin \Phi_{net} \\ \sin \Phi_{net} & \cos \Phi_{net} \end{pmatrix}$$

#### D. Lunar Reflection Matrix ($\mathbf{M}_{moon}$)

Reflection off the lunar surface causes a polarization reversal. In the $H/V$ basis, the ideal mirror reflection matrix is:

$$\mathbf{M}_{moon} = \begin{pmatrix} 1 & 0 \\ 0 & -1 \end{pmatrix}$$

*Note: This reverses the vertical component's phase, causing an RHCP $\leftrightarrow$ LHCP flip.*

### 5. Link Equation & Polarization Loss Factor (PLF)

The signal evolution must follow the chronological path because the reflection matrix $\mathbf{M}$ is **non-commutative** with the rotation matrices.

#### Full Signal Evolution

The final incident wave at the receiving end $\mathbf{P}_{final}$ is:

$$\mathbf{P}_{final} = \mathbf{R}(\Phi_{down}) \cdot \mathbf{M}_{moon} \cdot \mathbf{R}(\Phi_{up}) \cdot \mathbf{P}_{TX}$$

Where $\Phi_{up} = \nu_D + \Omega_{D}$ and $\Phi_{down} = \nu_H + \Omega_{H}$.

#### Efficiency Calculation

The **Polarization Loss Factor (PLF)**, representing the power matching efficiency, is derived from the inner product of the incident wave and the receiving antenna's vector:

$$\text{PLF} = \left| \mathbf{P}_{RX}^\dagger \cdot \mathbf{P}_{final} \right|^2$$

This yields the **Final Polarization Transformation Matrix** that describes the entire EME link's polarization characteristic.

## Build & Environment

### Requirements

- **Compiler**: Visual Studio 2026 (or VS2019+ with C++20 support)
- **Platform**: Windows (x64), compatible with WDK 10.0.26100.0 environments
- **Standards**: C++20 (`/std:c++20`)

### Compile

- MSVC

  ```bash
  cl /std:c++20 /EHsc /Fe:FaradayRotation.exe `
     FaradayRotation.cpp `
     main_interactive.cpp
  ```

- GCC

  ```bash
  g++ -std=c++20 -O2 -o FaradayRotation \
      FaradayRotation.cpp \
      main_interactive.cpp
  ```

- Clang

  ```bash
  clang++ -std=c++20 -O2 -o FaradayRotation \
      FaradayRotation.cpp \
      main_interactive.cpp
  ```

##  Example Calculation

**Scenario**: EME between **BI6DX (OM81ks)** and **UA3PTW (KO93bs)** at 432 MHz.

| **Parameter**      | **BI6DX (OM81ks)** | **UA3PTW (KO93bs)** |
| ------------------ | ------------------ | ------------------- |
| **Coordinates**    | 31.79°N, 116.87°E  | 53.77°N, 38.13°E    |
| **Antenna $\psi$** | 90° (Vertical)     | 0° (Horizontal)     |
| **Moon Elevation** | ~27°               | ~27°                |

**Results**:

```bash
===========================================================================
  Calculation Results
===========================================================================

--- Station Information ---
DX Grid: KO93bs
Home Grid: OM81ks
Ground Distance: 6503.0 km
Frequency: 432.1 MHz (70cm band)

--- Rotation Components ---
Spatial Rotation: -42.766 deg
DX Faraday Rotation: 0.000 deg
Home Faraday Rotation: 0.003 deg
Total Rotation: -42.763 deg

--- Link Parameters ---
Path Length: 754840.0 km
Propagation Delay: 2517.875 ms

--- POLARIZATION LOSS ---
PLF (Polarization Loss Factor): 0.460999
Loss: -3.363 dB
Efficiency: 46.10 %

--- Interpretation ---
Fair: Moderate loss, may affect weak signal work.
===========================================================================
```





### GL on your EME activities! 73s from Izumi@BI6DX
