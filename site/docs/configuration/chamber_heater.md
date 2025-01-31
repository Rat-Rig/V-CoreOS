# Chamber Heater Control

RatOS includes built-in chamber heater control that supports three different scenarios:

- [1. Bed Heater](#1-bed-heater)
- [2. Internal Heater](#2-internal-klipperratos-controlled-heater)
- [3. External Heater](#3-external-stand-alone-controller-heater)

## Prerequisites

- Configure your slicer according to the [official RatOS slicer configuration](../slicers.md)

## 1. Bed Heater

By default, RatOS uses the bed heater to heat the chamber. Either the hotend thermistor or a chamber thermistor will be used to monitor the initial chamber temperature.

- If no `temperature_sensor chamber` is defined, the hotend thermistor will be used.

### Configuration

_Use these exact device names in your configuration:_

```
[temperature_sensor chamber]
sensor_type: ATC Semitec 104GT-2
sensor_pin: PF4
```

## 2. Internal (Klipper/RatOS Controlled) Heater

Uses a PWM-controlled chamber heater, heater fan, and chamber thermistor.

- If no `temperature_sensor chamber` is defined, the hotend thermistor will be used
- If `temperature_sensor chamber` is defined, it will be used for automatic temperature control

### Configuration

<div className="text-amber-300 font-medium">
_Use these exact device names in your configuration:_
</div>

```
[temperature_sensor chamber]
sensor_type: ATC Semitec 104GT-2
sensor_pin: PF4

[heater_generic chamber_heater]
gcode_id: chamber_heater
heater_pin: PA2
smooth_time: 10
sensor_type: PT1000
sensor_pin: PF5
control: pid
pid_kp: 24.750
pid_ki: 2.578
pid_kd: 59.400
pwm_cycle_time: 0.25
min_temp: 0
max_temp: 200
max_power: 1.0

[heater_fan chamber_heater_fan]
pin: PE14
heater: chamber_heater
heater_temp: 40
```

## 3. External (stand-alone controller) heater

A dedicated heater device with or without its own temperature control that can be switched on/off by a relais/output_pin and a chamber thermistor to control the initial chamber temperature.

- if no `temperature_sensor chamber` is defined, the hotend thermistor will be used to wait for the initial chamber temperature.
- a `temperature_sensor chamber` can be used for the automatic chamber temperature control. In this case set `chamber_heater_control_external_heater` to `True`. This will turn the heater on/off when needed.

### Configuration

<div className="text-amber-300 font-medium">
_Use these exact device names in your configuration:_
</div>

```
[temperature_sensor chamber]
sensor_type: ATC Semitec 104GT-2
sensor_pin: PF4

[output_pin chamber_heater_pin]
pin: PE14
```

## Extra Chamber Heater Fan

A extra fan can be configured to support the chamber heating process, control its speed with the `chamber_heater_extra_fan_speed` variable.

```
[fan_generic chamber_heater_extra_fan]
```

To support more usecases the chamber heater extra fan control comes with two macro hooks that can be overwritten.

```
[gcode_macro _CHAMBER_HEATER_EXTRA_FAN_ON]
gcode:
	# config
	{% set chamber_heater_extra_fan_speed = printer["gcode_macro RatOS"].chamber_heater_extra_fan_speed|default(0.0)|float %}

	# turn chamber heater extra fan on
	{% if printer["fan_generic chamber_heater_extra_fan"] is defined %}
		{% if chamber_heater_extra_fan_speed > 0 %}
			SET_FAN_SPEED FAN=chamber_heater_extra_fan SPEED={chamber_heater_extra_fan_speed}
		{% endif %}
	{% endif %}
```

```
[gcode_macro _CHAMBER_HEATER_EXTRA_FAN_OFF]
gcode:
	# turn chamber heater extra fan off
	{% if printer["fan_generic chamber_heater_extra_fan"] is defined %}
		SET_FAN_SPEED FAN=chamber_heater_extra_fan SPEED=0
	{% endif %}
```

## User Macro Hooks

```
[gcode_macro _USER_CHAMBER_HEATER_BEFORE_PREHEATING]
description: Will be executed before chamber preheating, only if heating is needed.
gcode:

[gcode_macro _USER_CHAMBER_HEATER_AFTER_PREHEATING]
description: Will be executed after chamber preheating, only if heating was needed.
gcode:
```

### RatOS Configuration

```
[gcode_macro RatOS]
variable_chamber_heater_enable: True                      # Enable/disable chamber heater control
variable_chamber_heater_bed_temp: 115                     # Bed temperature during chamber preheating (°C)
variable_chamber_heater_preheating_temp: 150              # Generic heater temperature for chamber preheating (°C)
variable_chamber_heater_heating_temp_offset: 25           # Temperature offset for generic heater while printing (°C)
variable_chamber_heater_control_external_heater: False    # Enable automatic control for external heater devices
variable_chamber_heater_air_circulation_enable: True      # Use part cooling for air circulation during preheating
variable_chamber_heater_air_circulation_fan_speed: 0.35   # Part cooling fan speed for air circulation (0.0-1.0)
variable_chamber_heater_air_circulation_y_pos: 0          # Toolhead Y-position for air circulation
variable_chamber_heater_air_circulation_z_pos: 100        # Toolhead Z-position for air circulation
variable_chamber_heater_extra_fan_speed: 1.0              # Extra chamber heater fan speed (0.0-1.0)
variable_chamber_heater_filter_fan_speed: 1.0             # Filter fan speed for air circulation (0.0-1.0)
```
