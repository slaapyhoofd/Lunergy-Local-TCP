# Sunpura-Local-TCP
Control the sunpura s2400 over local tcp (No usage of the cloud)

Exposes following sensors: 
- AC charging power
- AC discharge power
- Battery SOC
- Power setpoint. (- is discharge, + is charging)


Internval update can be set to about 6 to 8 seconds for automations.


Look for the local battery IP adress in your router/AP. Input this in the setup window of the plugin.
The port for control is 8080.

Only ONE local device can control the battery at the same time.
