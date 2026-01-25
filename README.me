# Eddy Current Probe (Klipper Module)

This module implements support for an **eddy current Z probe** (e.g. LDC1612-based) in Klipper.  
It provides:

- Sensor calibration (frequency → height mapping)
- Probe sampling and conversion to Z positions
- Endstop-style probing (for homing)
- Scanning probe modes for bed mesh
- A custom “virtual contact” probe command

This file is **not run as a Python script**.  
It is loaded by Klipper as an extension and exposes functionality via **G-code commands** and **config options**.

---

## G-code Commands

### `PROBE_EDDY_CURRENT_CALIBRATE`

```
PROBE_EDDY_CURRENT_CALIBRATE CHIP=<name> [PROBE_SPEED=<mm/s>]
```

- Builds the calibration table (height → frequency)
- Requires `SAVE_CONFIG`

---

### `RUN_PROBE_VIR_CONTACT`

```
RUN_PROBE_VIR_CONTACT
```

Runs a **virtual contact probe** using the eddy sensor.

- Uses eddy trigger point
- Forces:
  - `sensor_z = 0`
  - `z_offset = 0`
- Returns:
  ```
  Result is z=...
  ```

Used for:
- Z homing without a physical switch
- Testing trigger at arbitrary XY
- Compatibility with contact-style macros

Speed controlled by:

```
vir_contact_speed: <mm/s>
```

---

## Probing Modes

### `METHOD=scan`

- Printer pauses
- Samples sensor
- Continues

Stable but slow.

```
PROBE METHOD=scan SAMPLE_TIME=0.12
```

---

### `METHOD=rapid_scan`

- Samples continuously while moving
- No dwell

Fast but slightly noisier.

```
BED_MESH_CALIBRATE METHOD=rapid_scan
```

---

## Configuration Example

```
[probe_eddy_current eddy]
sensor_type: ldc1612
z_offset: 3.1
i2c_mcu: extra_mcu
i2c_bus: i2c1
x_offset: -16.43
y_offset: 10.22
vir_contact_speed: 2.8
```

Calibration table (auto-generated):

```
#*# calibrate =
#*#   0.050156:8503553.685,0.090000:8502612.845,...
```

---

## Offset Handling

### Probe-to-nozzle offsets (X/Y/Z)

From:

```
x_offset
y_offset
z_offset
```

Used to:
- Position probe over XY
- Convert probe Z → nozzle Z

During calibration:
- Nozzle touches bed
- System moves by X/Y offsets
- Sensor aligns to the same bed point

---

### Wrapper Z offset (trigger reference)

`z_offset` is also used as the **eddy trigger height**:

```
trigger_freq = height_to_freq(z_offset)
```

So it plays **two roles**:

1. Probe offset (probe vs nozzle)  
2. Eddy trigger reference height  

---

## (1) Test Checklist

### Verify X/Y offsets

```
G28
G1 X150 Y150
PROBE
```

Visually confirm probe is centered where expected.

---

### Verify virtual contact

```
G1 X150 Y150
RUN_PROBE_VIR_CONTACT
```

Expected:

```
Result is z=...
```

---

### Verify scan mode

```
PROBE METHOD=scan SAMPLE_TIME=0.12
```

---

### Verify rapid scan

```
BED_MESH_CALIBRATE METHOD=rapid_scan
```

---

## (2) Troubleshooting

### No sensor data

Error:
```
probe_eddy_current sensor outage
```

Likely causes:
- I2C wiring
- Wrong `i2c_bus`
- Wrong `i2c_mcu`

---

### Calibration fails monotonic check

Error:
```
Failed calibration - frequency not increasing each step
```

Causes:
- Metal under bed
- Tilted probe
- Electrical noise

Fix:
- Raise probe slightly
- Remove steel sheet
- Re-run calibration

---

### Virtual contact crashes nozzle

Cause:
- Wrong `z_offset`
- Bad calibration

Fix:
- Recalibrate
- Increase `z_offset`
- Reduce `vir_contact_speed`

---

### Mesh warped or shifted

Cause:
- Incorrect X/Y offsets

Fix:
- Measure nozzle → probe offsets physically

---

## (3) Offset Diagram

Top view:

```
      +Y
       ↑
       |
       |   Probe (eddy)
       |     ●
       |
       |        Nozzle
       |          ●
       +----------------→ +X
```

Relationship:

```
probe_x = nozzle_x + x_offset
probe_y = nozzle_y + y_offset
```

Your example:

```
x_offset = -16.43   (probe is left of nozzle)
y_offset =  10.22   (probe is in front of nozzle)
```

---

## (4) Safe Macro Examples

### Safe virtual contact test

```
[gcode_macro TEST_VIR_CONTACT]
gcode:
  G91
  G1 Z5 F300
  G90
  G1 X150 Y150 F6000
  RUN_PROBE_VIR_CONTACT
```

---

### Safe Z home macro

```
[gcode_macro HOME_Z_EDDY]
gcode:
  G91
  G1 Z5 F300
  G90
  G1 X150 Y150 F6000
  RUN_PROBE_VIR_CONTACT
  G92 Z0
```

---

### Safe mesh macro

```
[gcode_macro MESH_EDDY]
gcode:
  BED_MESH_CLEAR
  BED_MESH_CALIBRATE METHOD=rapid_scan
  BED_MESH_PROFILE SAVE=eddy
```

---

## Summary

- `PROBE_EDDY_CURRENT_CALIBRATE` builds the height → frequency map  
- `METHOD=scan` = pause and sample  
- `METHOD=rapid_scan` = sample while moving  
- `RUN_PROBE_VIR_CONTACT` = eddy probe acting as contact  
- X/Y offsets are explicitly used during calibration  
- Z offset is used as both probe offset and trigger reference  
