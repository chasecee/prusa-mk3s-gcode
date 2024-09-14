# Prusa MK3S+ with Revo Roto Configuration

This README contains information about the Prusa MK3S+ 3D printer configuration with a Revo Roto hotend, including startup and end G-code.

## Printer Specifications

- Printer Model: Prusa MK3S+
- Hotend: E3D Revo Roto

## Startup G-code

```gcode
; PrusaSlicer start gcode for Prusa i3 Mk3S with E3D Revo HF
; Last updated 20240913
; Added next three lines for E3D Revo HF
M350 E16 ; microstepping (x16)
M92 E2682 ; steps per mm
M907 E450 ; extruder motor current (mA)
; Original code from muppetlabs (see credits below)
M300 S60 P10 ; chirp
M862.3 P "[printer_model]" ; printer model check
M862.1 P[nozzle_diameter] ; nozzle diameter check
M115 U3.14.0 ; tell printer latest fw version
M117 Initializing
; Set coordinate modes
G90 ; use absolute coordinates
M83 ; extruder relative mode
; Reset speed and extrusion rates
M200 D0 ; disable volumetric e
M220 S100 ; reset speed
M221 S100 ; reset extrusion rate
; Set initial warmup temps
M117 Nozzle preheat
M104 S160 ; set extruder no-ooze temp
M140 S[first_layer_bed_temperature] ; set bed temp
; Nozzle warmup before home to avoid driving hardened ooze into PEI surface
M109 S160 ; wait for extruder no-ooze warmup temp before mesh bed leveling
; Home
M300 S40 P10 ; chirp
M117 Homing
G28 W ; home all without mesh bed level
; Present bed for final cleaning
G0 Z3 ; Raise nozzle before move
G0 X125 Y180 F10200 ; Move nozzle to center position
G0 Z0.15 F10200 ; Lower nozzle
M117 MK3S with E3D Revo HF detected
M140 S[first_layer_bed_temperature] ; set target bed temp
G0 Z3 ; Raise nozzle before move
; Mesh bed leveling
M300 S40 P10 ; chirp
M117 Mesh bed leveling
G80 ; mesh bed leveling
M117 Saving results
G81 ; save mesh leveling results
; Final warmup routine
M117 Final warmup
G0 Z5 ; Raise nozzle to avoid denting bed while nozzle heats
M140 S[first_layer_bed_temperature] ; set bed final temp
M104 S[first_layer_temperature] ; set extruder final temp
M109 S[first_layer_temperature] ; wait for extruder final temp
M190 S[first_layer_bed_temperature] ; wait for bed final temp
; Prime line routine
M300 S40 P10 ; chirp
M117 Printing prime line
G0 Z0.15 ; Restore nozzle position
M900 K0 ; Disable Linear Advance for prime line
G92 E0.0 ; reset extrusion distance
G1 Y-3.0 F1000.0 ; go outside print area
G1 E2 F1000 ; de-retract and push ooze
G1 X20.0 E6 F1000.0 ; fat 20mm intro line @ 0.30
G1 X60.0 E3.2 F1000.0 ; thin +40mm intro line @ 0.08
G1 X100.0 E6 F1000.0 ; fat +40mm intro line @ 0.15
G1 E-0.8 F3000 ; retract to avoid stringing
G1 X99.5 E0 F1000.0 ; -0.5mm wipe action to avoid string
G1 X110.0 E0 F1000.0 ; +10mm intro line @ 0.00
G1 E0.6 F1500 ; de-retract
G92 E0.0 ; reset extrusion distance
; Final print adjustments
M300 S40 P10 ; chirp
M117 Preparing to print
M900 K0.08 ; Re-enable Linear Advance with K value of 0.8 (adjust as needed)
; Adjust extrusion rate if desired
; M221 S{if layer_height >= 0.32}90{else}100{endif} ; compensate for thick layer heights
M117 Print in progress
M220 S{if layer_z < 20}100{else}90{endif} ; M220 adjusts feed rates to 90% at 20mm height
```

## End G-code

```gcode
; Last edited 20200215
G4 ; wait
; Raise nozzle and present bed
M117 Printing complete
G4 ; wait
G92 E0 ; prepare to retract
G1 E-0.8 F3000; retract to avoid stringing
; Anti-stringing end wiggle
G91 ; use relative coordinates
G1 E-12 F800 ;retract filament from meltzone
G1 X-0.5 Y-0.5 F1200
G1 X1 Y1 F1200
G90 ; use absolute coordinates
{if layer_z < max_print_height}G1 Z{z_offset+min(layer_z+60, max_print_height)}{endif} ; Move print head up
G0 X0 Y210 F10200; present bed
; Reset print setting overrides
M200 D0 ; disable volumetric e
M220 S100 ; reset speed factor to 100%
M221 S100 ; reset extruder factor to 100%
M900 K0 ; reset linear acceleration
; Shut down printer
M104 S0 ; turn off temperature
M140 S0 ; turn off heatbed
M107 ; turn off fan
M84 ; disable motors
M300 S40 P10 ; chirp
```

## Key Features

1. **E3D Revo Roto Hotend**: This configuration is optimized for the E3D Revo Roto hotend, which offers quick and easy nozzle changes.

2. **Mesh Bed Leveling**: The startup G-code includes mesh bed leveling for improved first layer adhesion.

3. **Linear Advance**: Linear Advance is enabled with a K value of 0.04, which can be adjusted as needed.

4. **Dynamic Feed Rate Adjustment**: The feed rate is automatically adjusted to 90% when the print height reaches 20mm, which can help improve print quality for taller objects.

5. **Anti-stringing Measures**: Both the startup and end G-code include measures to reduce stringing, such as retraction and end wiggle.

## Usage Notes

- Make sure to adjust the `[first_layer_temperature]` and `[first_layer_bed_temperature]` values in your slicer settings to match your filament requirements.
- The Linear Advance K value (M900 K0.04) may need to be fine-tuned for your specific setup and filament.
- Always monitor the first few layers of your print to ensure proper adhesion and extrusion.

## Customization

Feel free to modify the G-code to suit your specific needs. Some areas you might want to customize include:

- Adjusting temperatures for different filament types
- Modifying the prime line routine
- Changing the Linear Advance K value
- Adjusting the feed rate reduction threshold and percentage

## Troubleshooting

If you encounter issues with your prints, consider the following:

1. Verify that your E3D Revo Roto hotend is properly installed and calibrated.
2. Check that your bed is clean and properly leveled.
3. Adjust the first layer height and extrusion multiplier if necessary.
4. Fine-tune the Linear Advance K value for your specific filament.

## Credits

Startup G-code adapted from: [@muppetlabs.co](https://muppetlabs.co/3dprinting_prusaslicer_start_gcode_mk3.html)

## Conclusion

This configuration for the Prusa MK3S+ with E3D Revo Roto hotend aims to provide a solid starting point for high-quality 3D prints. Remember to always test and adjust settings based on your specific needs and materials. Happy printing!
