https://www.kicad.org/discover/spice/

# How to simulate?

Start by following [this](https://www.woolseyworkshop.com/2019/07/01/performing-a-circuit-simulation-in-kicad/) tutorial. No need to understand in depth what is being simulated here (I didn't!). The following is a more concise summarise of the steps, with some text being copied directly.

Note: the following is for KiCad 7.0. Recent 8.0 updates improve SPICE simulation significantly but some aspects have changed, toolbar icons, how simulations are reported etc.

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
	- This will perform a transient analysis - a type of simulation that examines the behavior of a circuit over time, particularly in response to changes in input signals
	- 1u is the maximum step time i.e. the time interval between simulation data points (1 microsecond)
	- 1m is the simulation time (1 millisecond)

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

# Audio-specific simulator examples

There's not much out there addressing the topic of simulation in KiCad with an audio focus. 

Best tips I found so far was in [this thread](https://www.reddit.com/r/KiCad/comments/1961s6p/using_the_spice_simulator_in_kicad_on_a_guitar/), this being:
- SPICE simulator doesn't handle open inputs, outputs, or connectors very well. So, you'll need to attach appropriate voltage sources for your circuit to the input connectors
- some load for the outputs is needed (sometimes a resistor is enough, but it depends on what you are trying to accomplish)
- each symbol needs a PSPICE model associated with it. Passive components are already included, active ones we need to find online..
- instead of potentiometers try using discrete resistors and vary the values between simulation passes
- AC Sweep is probably the analysis method you want

What simple 

- we can simulate audio-in by just putting a sine wave in

## Simple attenuator

I'm starting with this Doepfer [simple passive attenuator example](https://doepfer.de/DIY/a100_diy.htm)

I'm simulating now in KiCad 8.0

The circuit consists of:
- a `Vin` label
- a VSIN voltage source, the same as in the above example i.e. with the following simulation parameters
	- dc=0 ampl=2.5 f=10k td=0 theta=0 phase=0 ac=1
	- this will produce a 5V peak-to-peak sine wave that peaks at 2.5V and troughs at -2.5v and whose centre is 0V (no DC offset)
- a 5OΩ resistor
- a `Vout` label

With the following connections:
- the `Vin` label is connected to the + side of the VSIN voltage source
- the other side of VSIN is connected to GND
- another 'Vin' label is connected to one side of the resistor
- the other side of the resistor is connected to GND
- the `Vout` label is connected between the output of the resistor and GND

We place the `.tran 1u 1m` simulation command as text in the schematic and then use `Inspect -> Simulator`, then click the `Play/Run Simulation` button to begin

- Note: If `v2#branch` or `r1#branch` are negative, flip the resistor in the schematic

### Troubleshooting

There are some strange things in the output report

```
Background thread stopped with timeout = 0
Note: No compatibility mode selected!
Circuit: KiCad schematic
Doing analysis at TEMP = 27.000000 and TNOM = 27.000000
Warning: singular matrix:  check node probe_int_probe_int_vin_r1_r1_2
Note: Starting dynamic gmin stepping
Warning: singular matrix:  check node probe_int_probe_int_vin_r1_r1_2
Warning: Dynamic gmin stepping failed
Note: Starting true gmin stepping
Warning: singular matrix:  check node probe_int_probe_int_vin_r1_r1_2
Warning: singular matrix:  check node probe_int_probe_int_vin_r1_r1_2
Warning: singular matrix:  check node probe_int_probe_int_vin_r1_r1_2
Warning: singular matrix:  check node probe_int_probe_int_vin_r1_r1_2
Warning: True gmin stepping failed
Note: Starting source stepping
Warning: source stepping failed
Note: Transient op started
Note: Transient op finished successfully
Initial Transient Solution
--------------------------
Node                                   Voltage
----                                   -------
probe_int_vin_v2_1                           0
probe_int_vout_v2_2                          0
probe_int_vout_r1_1                          0
probe_int_probe_int_vin_r1_r1_2               0
v2probe_int_vref                             0
r1probe_int_vref                             0
r1:power                                     0
probe_int_vin_r1                             0
vout                                         0
v2:power                                     0
vin                                          0
bprobe_int_v2power#branch                    0
v2:probe_int_n1#branch                       0
v2:probe_int_n2#branch                       0
bprobe_int_r1power#branch                    0
r1:probe_int_n1#branch                       0
r1:probe_int_n2#branch                       0
bprobe_int_r1vref#branch                     0
bprobe_int_v2vref#branch                     0
r1#branch                                    0
v2#branch                                    0
 Reference value :  0.00000e+00
No. of Data Rows : 1008
```

- many warnings
- all zeros in the node/voltage list
- by changing `Simluation menu -> Edit analysis tab -> Compatibility mode` to `PSpice and LTSpice` we can avoid the `Note: No compatibility mode selected!` warning
- when graphing Vin and Vin from the right hand Signals pane, the Vout is the correct range (-2.5v to 2.5v) and the Vin is attenuated (-100mV to 100mv). Why?

## Non-inverting Amplifier

I sketched and simulated the _non-inverting amplifier_ example from the [Doepfer A-100 specs](https://doepfer.de/DIY/a100_diy.htm), using a TL071 as the op amp.

### Create the schematic:
1. Create two VDC's, with values of 12V and -12V respectively and + sides going to global labels named `+12V` and `-12V` respectively. Other sides go to GND
2. Create a VSIN as a test input signal, with the sim parameters `dc=0 ampl=5 f=10k td=0 theta=0 phase=0 ac=1`. + side goes to `Vin` global label, other side goes to GND
3. Place symbols for the TL071 op amp and two 47K resistors (R1 and R2)
4. Place labels for `Vin`, `Vout`, `+12V` and `-12V`
5. Hook up as follows:
- `+12V` label to pin 7 of op amp (V+)
- `Vin` label to pin 3 of of op amp (+ _non-inverting input_)
- Pin 2 of op amp (- _inverting input_) to one side of second resistor (R2)
- other side of second resistor to GND
- Pin 6 of op amp (output) to `Vout` label
- From pin 2 op amp wire, connect a wire to one side of the first resistor (R1)
- other side of the first resistor to pin 6 wire (-> `Vout`)

### Add the SPICE model for the TL071
1. Download the file [here](https://web.archive.org/web/20200215133745/http://s-audio.systems/blog/spicemodels/) and place in your KiCad project folder
2. Right click on the op amp schematic symbol, choose `Properties -> Simulation Model` and import the .lib file
3. In 'pin assignment', hook up the 5 pins according to the text in the .lib file (shown in the reference pane)
```
1 (NULL)	Not Connected
2 (-)	2
3 (+)	1
4 (V-)	4
5 (NULL)	Not Connected
6	5
7 (V+)	3
8 (NC)	Not Connected
```

### Run the simulation

From `Inspect menu -> Simulator`, click the blue play button. The run should complete ok, if not check your schematic and TL071 pin assignments.

According to the Doepfer page, using two 47k resistors should lead to a 2x amplification (`R1 = 47k, R2 = 47k -> A = 2`), and we can check this by ticking the Vin and Vout boxes in the right hand Signals pane. Indeed, the output (-10V to 10V) is double the magnitude of the input (-5V to 5V).

### Changing the amplification range

We can change the value of the resistors to those of the other example given (`R1 = 100k, R2 = 10k -> A = 11`) and see if it checks out.
- At first the output clips
- This is because 11 * 5V would be 55V which is way more than our op amp can output
- So we need to bring down the `Vin` to something like 0.8
- Once we do this we see an `Vin` of +-800mV and a `Vout` of something like +-8.7V

### Replacing the fixed resistor with a potentiometer

If R1 in the last example is replaced by a 100k potentiometer the amplification is adjustable in the range 1...11. We can simulate this using the simulators tune function, accessible from the screwdriver icon or `Simulation menu -> Add tuned value`.

When you click it, then you can click any resistor or capacitor in the schematic and it will be added to the “Tune” area. There you can enter maximum and minimum limits and then use the slider to adjust, or directly type a value in the middle box.

If we then know the min/max limits we want e.g. 50 to 100K, we can place a fixed 50K resistor to provide the initial offset (from 800mV to 5V), followed by a 50K potentiometer (to covers the 5V-9V range up until close to clipping).

Now how to add the potentiometer into the circuit to complete the process?
- I guess wire pin two to one side, but does it matter which?
- Do simulation models exist for potentiometers at all?

# Simple circuit resources

An LFO would be easy to interpret
https://forum.kicad.info/t/simulation-examples-for-kicad-eeschema-ngspice/34443
 (passive elements here)
https://github.com/thehiddendatacentre/Single_Supply_Synth_Circuits
https://electro-music.com/forum/forum-160.html
https://sdiy.info/wiki/Schematics_and_manuals
https://github.com/erica-synths/diy-eurorack
https://www.falstad.com/circuit/
https://github.com/mysticcircuits/0HP_Modular
https://musicfromouterspace.com/index.php?MAINTAB=SYNTHDIY&VPW=1430&VPH=628
https://electro-music.com/forum/topic-56660.html

# Other issues

KiCad is slow and/or crashes when simulation window is open at same time as schematic window. For now I [disabled cross probing](https://forum.kicad.info/t/pcbnew-slow-when-eeschema-open/43831/4)