
# Contributing to Powder Box

Thank you for your interest in contributing to **Powder Box**! We welcome contributions of all kinds, from adding new chemical elements and physical reactions to improving performance, UI accessibility, or documentation.

This guide provides an overview of the codebase structure and step-by-step instructions for implementing new features, fixing bugs, and submitting pull requests.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Architecture & Code Base Overview](#architecture--code-base-overview)
  - [Element Data Format (`SRC`)](#element-data-format-src)
  - [Reaction Syntax (`RXS`)](#reaction-syntax-rxs)
  - [Data Arrays & Physics Simulation Loop](#data-arrays--physics-simulation-loop)
- [How to Add New Elements](#how-to-add-new-elements)
  - [1. Adding a Raw Element String](#1-adding-a-raw-element-string)
  - [2. Option Flags Breakdown](#option-flags-breakdown)
- [How to Add New Reactions](#how-to-add-new-reactions)
- [Development Setup](#development-setup)
- [Submitting a Pull Request](#submitting-a-pull-request)
- [Code Style Guidelines](#code-style-guidelines)

---

## Project Overview

Powder Box is a single-file, zero-dependency HTML5 cellular automata sandbox simulating thousands of unique elements, materials, life forms, chemical compounds, and reactions running via Canvas rendering.

Key constraints:
- **Single-file distribution**: The core application lives in a self-contained HTML file containing inline CSS and JavaScript.
- **High performance**: State is tracked using flat TypedArrays (`Uint16Array`, `Float32Array`, `Uint8Array`) to maximize cache locality and maintain high frame rates.

---

## Architecture & Code Base Overview

### Element Data Format (`SRC`)

Elements are defined as pipe-delimited (`|`) strings inside the main data block:

```text
Name|ColorHex|Kind|Density|Options
```

| Field | Type | Description |
| :--- | :--- | :--- |
| **Name** | `String` | Display name of the element (e.g., `Molten Glass`). |
| **ColorHex** | `String` | 3 or 6-digit hex string representing the base color (e.g., `e6c86e`). |
| **Kind** | `Integer` | State type: `1` = Powder, `2` = Liquid, `3` = Gas, `4` = Solid. |
| **Density** | `Number` | Relative weight determining buoyancy/layering for liquids and powders. |
| **Options** | `String` | Space-separated flag strings defining thermal transitions, ignition properties, aging, decay, or special capabilities. |

### Reaction Syntax (`RXS`)

Custom 2-body chemical and physical reactions are declared as plain text formatted strings:

```text
ReactantA+ReactantB>ProductA+ProductB Probability
```

*Example:*
```text
Water+Salt>Salt Water+ .3
Fire+Water>+Steam 1
Lava+Water>Stone+Steam 1
```

- Spaces denote empty output products (e.g., `Water+Salt>Salt Water+ .3` transforms `Water` into `Salt Water` and clears the `Salt` cell).
- The final floating-point value represents the per-frame reaction probability (`0.0` to `1.0`).

### Data Arrays & Physics Simulation Loop

Grid states are indexed by 1D array coordinates calculated via `index = y * W + x`:

- `g`: `Uint16Array` containing element IDs (`0` = Erase/Empty).
- `a`: `Uint16Array` tracking particle lifetime (`LF`) or clone state.
- `v`: `Uint8Array` storing visual color variation or movement direction.
- `T`: `Float32Array` holding cell temperature in °C.
- `m`: `Uint8Array` storing update iteration flags to prevent double-processing in a single tick.

---

## How to Add New Elements

### 1. Adding a Raw Element String

Locate the `SRC` definition string in `Powder_Box.html` and append a new line:

```javascript
// Format: Name|HexColor|Kind|Density|Options
MyElement|ff0055|1|5.0|T200 h400>Lava i150 b20
```

### 2. Option Flags Breakdown

Use option flags to configure behavior without writing custom JavaScript logic:

| Flag | Parameter Syntax | Description | Example |
| :--- | :--- | :--- | :--- |
| `T` | `T<number>` | Starting temperature in °C | `T-50` |
| `K` | `K<number>` | Fixed held temperature in °C | `K500` |
| `h` | `h<temp><target>` | Heat transition (turns into `<target>` above `<temp>` °C) | `h1000>Molten_Iron` |
| `c` | `c<temp><target>` | Cold transition (turns into `<target>` below `<temp>` °C) | `c0>Ice` |
| `i` | `i<temp>` | Ignition threshold in °C | `i250` |
| `b` | `b<ticks>` | Burn duration in simulation frames | `b30` |
| `x` | `x<radius>` | Blast radius upon ignition (explosion) | `x8` |
| `L` | `L<ticks><target>` | Life span tick count before decaying into `<target>` | `L120>Ash` |
| `f` | `f<speed>` | Liquid flow rate/viscosity modifier | `f.3` |
| `s` | `s<target>` | Generator block (spawns `<target>`) | `s>Water` |
| `C` | `C` | Cloner (replicates adjacent elements) | `C` |
| `V` | `V` | Void (absorbs touching elements) | `V` |
| `P` | `P` | Plant growth behavior (expands into water) | `P` |
| `A` | `A` | Acidic (dissolves non-resistant solids/powders) | `A` |
| `r` | `r` | Resistant to acid and blast shockwaves | `r` |
| `w` | `w<type>` | Life form movement mode: `1` = Walk, `2` = Swim, `3` = Fly | `w1` |
| `d` | `d<prob><target>` | Decay process: probability per frame into `<target>` | `d.001>Lead` |

> **Note:** Replace spaces in target element names with underscores inside options (e.g., `Molten_Glass`).

---

## How to Add New Reactions

To add procedural or custom reactions, append your rule to the `RXS` template string or use the helper function `rx()` inside the script block:

```javascript
// Adding via RXS string:
// ReactantA+ReactantB>ProductA+ProductB Probability
Lava+Sand>Lava+Glass .03

// Adding programmatically via helper:
// rx(ReactantA, ReactantB, ProductA, ProductB, Probability)
rx('Sulfur Acid', 'Iron', 'Rust', 'Hydrogen', 0.1);
```

---

## Development Setup

No complex toolchains, Node.js packages, or build processes are required to work on Powder Box.

1. **Fork and Clone the Repository**
   ```bash
   git clone https://github.com/your-username/powder-box.git
   cd powder-box
   ```

2. **Run Locally**
   Open `Powder_Box.html` directly in any WebGL/Canvas2D supported modern web browser:
   ```bash
   # On macOS
   open Powder_Box.html
   
   # On Linux
   xdg-open Powder_Box.html
   ```

3. **Testing Changes**
   Refresh your browser to load edits made to `Powder_Box.html`. Use the in-game **Lookup** button (`lu`) to inspect element flags and reactions to verify configuration correctness.

---

## Submitting a Pull Request

1. **Create a Feature Branch**
   ```bash
   git checkout -b feature/add-new-element
   ```

2. **Commit Your Changes**
   Provide clear, concise commit messages explaining the added elements or logic:
   ```bash
   git commit -m "Add Titanium metal and high-temperature oxidation reaction"
   ```

3. **Test Simulation Balance**
   - Ensure performance remains stable during large-scale interactions.
   - Verify that new elements do not crash the lookup modal or render incorrectly.

4. **Push and Open PR**
   ```bash
   git push origin feature/add-new-element
   ```
   Open a Pull Request on GitHub against the `main` branch. Provide a brief description of the new elements, reactions, or fixes introduced.

---

## Code Style Guidelines

- **Conciseness**: Keep single-file constraints in mind. Utilize existing option flags (`SRC`) and reaction rules (`RXS`) before adding imperative logic to `upd()`.
- **Naming Conventions**: Use `PascalCase` for element display names (e.g., `Liquid Nitrogen`).
- **Data Integrity**: Ensure densities and thermal thresholds reflect reasonable relative properties compared to real-world physics where applicable.
