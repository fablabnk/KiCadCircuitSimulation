https://www.kicad.org/discover/spice/

# How to simulate?

Start by following [this](https://www.woolseyworkshop.com/2019/07/01/performing-a-circuit-simulation-in-kicad/) tutorial. No need to understand in depth what is being simulated here (I didn't!). The following is a more concise summarise of the steps, with some text being copied directly.

## Open the schematic

Open the `TransistorSwitchSimulation_Before` project given in this repo, then the schematic `TransistorSwitchSimulation.kicad_sch` from within the project. This gives us a simple circuit schematic as a starting point but without any simulation. Now follow these steps:

## Add simulatable voltage sources

Add the first of two simulatable DC voltage sources, as follows:
1. Place a new VDC voltage source component (located within the `Simulation_SPICE` component library) into the schematic. Note this is done with `Add a symbol (A)` not `Add a power symbol (P)`
2. Wire a global output label named Vcc to the positive side of the VDC component
3. Wire a GND power port to the negative side of the VDC component.
4. Now change the value of VDC to 5, meaning it will provide a 5 V DC supply.
- Repeat the above steps to create another VDC, this time with a global output label named Vin and a value of 0

## Add a SPICE model for the transistor

Note: for the passive components the schematic (such as resistors) the models are built-in. We can see this when right click on them, choose `Properties` and click `Simulation Model`. `Built-in SPICE model` is already chosen for us and `Device` and `Type` are set.

- For the transistor, we create the SPICE model by saving the following text as a PN2222A.lib file in the root KiCad project directory
```
* Modified from LTspice BJT standard library (2N2222)
.model PN2222A NPN (IS=1E-14 VAF=100
+ BF=200 IKF=0.3 XTB=1.5 BR=3
+ CJC=8E-12 CJE=25E-12 TR=100E-9 TF=400E-12
+ ITF=1 VTF=2 XTF=3 RB=10 RC=.3 RE=.2)
```
- Now right click on the transisor in the schematic, choose `Properties` and click `Simulation Model`
- In the `Model` tab choose `SPICE model from file` and navigate to and double click on the `PN2222A.lib` file
- In the `Pin Assignments` tab make sure the `Symbol Pin` and `Model Pin` letters match i.e.
	- E and E in the same row
	- B and B in the same row
	- C and C in the same row
- Click OK

## Simulate the transistor in the off state

- Add the text command `.tran 1u 1m` to the schematic
	- This will perform a transient analysis simulating the circuit from 0 to 1 ms using 1 us for each step

Our transistor is in the off state because we previously set the Vin voltage source value to 0

- Choose `Simulator` from the `Inspect` menu
- Check the setup in the `Sim Command` (gear icon) section
	- In the `Custom` tab the text command `.tran 1u 1m` should be shown
	- At the bottom of the window, compatibility mode should be set to `PSpice and LTSpice`
	- Click OK
- Click `Run/Stop Simulation`

You should see some text output 
- `vcc` should show 5 and `vin` should show 0 (as we set in the value section of each voltage source)
- 

## Simulate the transistor in the on state

- Change the value of the VIN voltage source to 5
- Choose `Simulator` from the `Inspect` menu and click `Run/Stop Simulation`
- In this output we should see:
	- `Vin` is now equal to 5 V
	- Vc is now 57.1 mV, meaning the transistor is now “on” and current is flowing through the 150 Ω (R2) resistor.

- the r1#branch and r2#branch represent the current flowing through resistor R1 (into the base of the transistor) and through resistor R2 (into the collector of the transistor) respectively.

The schematic with the simulation settings already added can be found a project called `TransistorSwitchSimulation_After`

## Viewing Waveforms

- In `Inspect -> Simulator`, click the `Add Signals` icon in the toolbar and select a signal you want to view in the popup window
	- e.g. I(R1) gives the current flowing through resistor R1
	- Note: the simulation has to have been run once first
- To remove a signal from the graph, double click it in the `Signals` panel

### More interesting Waveforms

Here we add a 100 mV ripple to the input voltage

Setup the new voltage source as follows:
- Add a new voltage source as before, except instead of VDC choose VSIN
- In `Properties -> Simulation Model -> Model Tab -> Built-in SPICE model` enter the following parameters:
	- DC offset:		5V
	- Amplitude:		100m
	- Frequency:		10k
- Add a global label 'Vin2' to the schematic and hook it up to the + terminal of the new voltage source
- Change also the label in the main circuit to `Vin2`
- Add a ground symbol and connect the other side of the new voltage source to it

Now perform the simulation again, and add the signal `Vin2`. Ten cycles of a a sinusoidal waveform should be shown, varying from 4.9V to 5.1V 
Try adding other signals

# Eurorack-specific examples
