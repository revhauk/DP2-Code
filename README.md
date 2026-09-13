# Boxed Bend Pressure Model

Thermodynamic model estimating pressure buildup from residual hydrotest water evaporating inside a sealed boxed-bend leak-detection enclosure.

## Background

If water remains within a sealed enclosure that is heated, it will increase the pressure significantly compared to a dry enclosure.

This model calculates how much the enclosure's internal pressure rises, for a given amount of residual water, when heated from ambient temperature up to process operating temperature. The goal is to determine the maximum residual water volume that can be left in the enclosure after sealing without generating a "ceiling" pressure.

## How it works

The enclosure is treated as a rigid, sealed, fixed-volume vessel containing trapped air plus a known volume of residual liquid water. As temperature rises, the model tracks which of three physical states the water is in:

1. **Dry air** - no water present; pressure comes from the trapped air alone.
2. **Liquid water + saturated vapour** - some liquid remains, with vapour above it sitting exactly at its saturation pressure.
3. **Fully evaporated (superheated) vapour** - all the water has flashed to vapour, which is now unsaturated at the enclosure's volume.

At each temperature step, the model checks which state applies and calculates air and water partial pressures accordingly (Dalton's law), then sums them to get total pressure.

Water/steam properties (saturation pressure, liquid density, vapour density, and superheated vapour pressure) are sourced from [pyfluids](https://github.com/portyanikhin/PyFluids), a Python wrapper around [CoolProp](http://www.coolprop.org/), which implements the IAPWS-95 reference equation of state for water. Air is treated with the ideal gas law throughout, which is an excellent approximation at these temperatures and pressures (air stays far from any real-gas nonideality here, unlike water vapour near its saturation dome).

## Requirements

- Python 3.9+
- [numpy](https://numpy.org/)
- [matplotlib](https://matplotlib.org/)
- [pyfluids](https://github.com/portyanikhin/PyFluids)

## Installation

```bash
pip install numpy matplotlib pyfluids
```

## Usage

```bash
python boxed_bend_pressure.py
```

Key inputs are set at the top of the script and can be adjusted for a different scenario:

| Variable | Meaning |
|---|---|
| `MAX_WATER_L` | Largest residual water volume considered in the sweep (L) |
| `V_TOTAL_PIPE` | Fixed internal volume of the enclosure (m³) |
| `T_INITIAL_C` | Ambient temperature at which the enclosure is sealed (°C) |
| `T_FINAL_C` | Hot process operating temperature after installation (°C) |
| `POPPER_ACTIVATION_KPA` | Gauge pressure at which the popper is set to activate (kPa) |

## Outputs

Running the script produces:

- A console table of total pressure, air/water partial pressures, and phase state for a spread of water volumes at the final operating temperature.
- `pressure_curves_2D.png` - gauge pressure vs. temperature for several water volumes, with the popper activation pressure marked as a reference line.
- `pressure_surface_3D.png` - a pressure surface across the full temperature/water-volume grid, with the popper activation pressure shown as a reference plane.

## Key assumptions and limitations

- Air behaves as an ideal gas throughout; water/steam properties use the real IAPWS-95 equation of state via CoolProp.
- Air and water vapour are treated as a non-interacting ideal mixture (Dalton's law) - no enhancement-factor or dissolved-air effects are modelled.
- Trapped air is assumed to be fully saturated with water vapour (100% relative humidity) at the moment the enclosure is sealed, if any water is present.
- The enclosure is treated as rigid (fixed volume) and at uniform temperature, with no transient/dynamic heating effects.
- This is a screening-level model intended to support design decisions, not a substitute for formal engineering sign-off.

## Status

Part of an ongoing university design project; the pressure model output feeds into the drying/venting strategy recommended.
