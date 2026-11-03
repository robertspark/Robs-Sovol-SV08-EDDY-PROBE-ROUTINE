# SOVOL SV08 Eddy Current Probe (Klipper Module)

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

## Repository status and workstream maintenance

Documentation review: 3 November 2026. This is repository-local status, not an upstream release or deployment acceptance.

### Purpose and function

Sovol SV08/Klipper eddy-current probe module and calibration/probing guidance.

### Current objective

Maintain the documented project purpose and make future work resumable through current objectives, evidence and next actions.

### Progress and evidence

Project files or documentation are present on the inspected default branch. This review establishes README coverage, not build/runtime/deployment/hardware acceptance. Existing development claims are not promoted to verified completion.

### Next steps and roadmap

1. Confirm the next owner-approved outcome and reconcile this default branch with the workstream's actual source, test evidence and deployed version.
2. Record the relevant build, test, usage or document-review evidence; live-service or hardware acceptance needs its own checks.
3. Keep completed/in-progress/blocked work and next actions current here. Broader feature priorities remain for the owning workstream to agree; this update does not invent a feature commitment.

### Documentation chain

Read this README first, then the existing records and source/example entry points below. Listed paths were observed on the inspected default branch; source folders are not test results.

No separate root roadmap/handover was present. Maintain the objective and next actions here until a dedicated, linked record is needed.

### Standing instruction for every workstream

Every agent, developer or project workstream maintaining this repository must automatically keep this README current with its purpose/function, agreed objective, dated progress, blockers, roadmap/next actions and links to authoritative records. Update it when these facts materially change, at meaningful milestones and before handback, interruption or completion; do not wait for a separate owner reminder.

Read applicable project instructions, preserve concurrent work and existing attribution/licences, and distinguish source commits from deployed versions and verified results. Link requirements, design, handover, validation and recovery records when they exist rather than creating conflicting copies. If a fact is unknown, say so and identify the next verification action. Check affected links, publish authorised changes and verify the remote commit before claiming completion.

Keep credentials, private infrastructure/access routes, private datasets and confidential material out of public documentation. This instruction does not grant new access, replace project-specific authority or approve untested hardware/live operations. Retain detailed historical evidence in project records/version history rather than an ever-growing README log.
