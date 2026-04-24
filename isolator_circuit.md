Recommended one-channel circuit
HVAC SIDE (isolated)                     PI SIDE (isolated from HVAC)

W / Y / G / R call line ---- R1 10k ----+----[ LTV-814S input ]
                                        |
C (thermostat common) ------------------+----[ LTV-814S input ]


Pi 3.3V ---- R2 10k ----+----> to Schmitt inverter input (74LVC14 / 74HC14)
                        |
                        +---- C1 2.2uF ---- Pi GND
                        |
                  opto collector
                  opto emitter ----------- Pi GND

Schmitt inverter output ---------------> Raspberry Pi GPIO

Replicate that once each for:
W to C = heat call detect
Y to C = cool call detect
G to C = fan call detect
optionally R to C = system 24 VAC present
Do not connect thermostat C to Pi ground. The whole point here is to keep the HVAC side isolated.
Why this works
With 24 VAC present, the AC-input optocoupler turns on in both half-cycles, so the transistor repeatedly pulls the RC node low. Because C1 is fairly large, the node stays low between zero crossings instead of bouncing at 60 Hz. When the thermostat signal disappears, the opto stops pulling down and R2 charges the node back up. The Schmitt input turns that slow RC edge into a crisp GPIO transition.
That gives you hardware filtering instead of software debounce.
Values I’d use
Per channel:
R1 = 10 kΩ, 1/4 W
R2 = 10 kΩ
C1 = 2.2 µF
U2 = 74LVC14A or 74HC14 powered from 3.3 V
Why 10 kΩ on the 24 VAC side
At nominal 24 VAC, the peak is about 34 V, so 10 kΩ gives a few mA peak through the opto input, which is a comfortable operating point for this part. Power in the resistor is about:
P≈ 
10000
24 
2
 
​	
 ≈58 mW
So 1/4 W is comfortably fine.
Why 10 kΩ + 2.2 µF on the Pi side
That gives a time constant of about:
τ=RC=10000×2.2μF≈22 ms
That is long enough to ride through the AC gaps, but still fast enough for thermostat signals. Expect roughly:
assert time: essentially immediate
release time: about 20–40 ms
For HVAC calls, that delay is trivial.
Logic polarity
With the circuit exactly as drawn:
24 VAC present on W/Y/G → opto pulls RC node low → Schmitt inverter output goes HIGH
24 VAC absent → RC node rises → Schmitt inverter output goes LOW
So this version gives you a nice active-high GPIO for each thermostat call.