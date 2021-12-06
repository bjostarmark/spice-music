# Spice Music
## Introduction
LTSpice is an analog electronics simulation tool. It can be used to simulate analog synthesizer modules on the transistor level and exports _.wav_ files. Any node voltage or branch current can be exported. This opens up for new possibilities not easily available in analog hardware. For instance, it is possible to listen to the sound of the base current of a particular transistor in a ladder filter. It is difficult to produce complicated sequences and therefore LTSpice is mostly suited for repetitive patterns.

Several modules that have been designed and tested are provided. They adopt the standard control voltage (CV) range of 0V to 10V. 

## Files
### %path%\spice-music\Basic
Contains basic modules. Some files of interest are:
sin_saw_tri_pulse.asc:      shows how to generate Sine, Saw, Triangular and Pulse waves using the _voltage_ source.
noise.asc:                  demo of white noise generation.
env_gen2.asc:               an envelope generator using the _EXP_ function in the _voltage_ source.
tb303_vco_normalized.asc:   example of a discrete VCO including normalization.

### %path%\spice-music\Advanced
Contains more advanced examples such as:
1-voice.asc                 Single Sawtooth voice with 4-pole ladder filter and VCA.
analog_sequencer_8.asc      An eight step analog sequencer.
chopperb.asc                A chopper-like sound effect.
droneb.asc                  A droning sound based on two detuned sawtooth VCO:s and a 4-pole ladder filter.
hihatb.asc                  A highhat sound.
spice_music_v0_1b.asc       Demo of Spice Music capabilities.

## Normalization
To simplify output mixing and module interconnection it is a good idea to normalize each module so it has a +/-1V signal and zero DC output voltage. An example of this is given in \Basic\tb303_vco_normalized.asc.

## Output mixing
A simple output mixer can be implmented by the _bv_ behavioral component. The LTSpice _.wave_ command assumes a +/-1V signal as Full Scale so the mixer should scale the output accordingly to avoid clipping. A waveform viewer is helpful here. The _Audacity_ software can highlight clipping, and was used extensively during development.

## Sequencing
Sequencing in LTSpice is difficult. To create simple sequences, a simple pulse _voltage_ source swinging from 0 to 1V can be used in conjunction with a _bv_ source to gate a module. More advanced sequences can be implemented by using multiple pulse sources as gates. It is helpful to think in logical terms here, with the _bv_ source multiplication acting as an AND gate on the pulse sources, masking out certain regions in a beat.

Analog sequences are even more difficult to implement. An example using a _voltage_ PWL source in PWL mode is shown in \Advanced\analog_sequencer_8.asc. PWL interpolates between points, so extra points need to be added to create flat voltage steps. This is very tedious. The aforementioned sequencer simplifies usage somewhat.

## Components
### Oscillators
#### General
LTSpice comes with fixed frequency voltage and current sources, and a VCO. These simulate very quickly. More general VCO:s can be built using discretes or op amps.
#### Fixed frequency oscillators
The _voltage_ component can simulate Sine, Saw, Triangle, Pulse and more complex waveforms. They simulate very quickly. Examples are found in \Basic\sin_saw_tri_pulse.asc.
#### _modulate_
LTSpice has a built-in sinusoidal voltage controlled oscillator called _modualate_. It comes with a VCA (AM input). This is useful for generating chirps. An example is Basic\vco.asc.
#### TB303 VCO
A simple sawtooth VCO is \Basic\tb303_vco_normalized.asc. It is based on the TB303 VCO and uses transistors and diodes only so it simulates quickly. Oscillators can have start-up problems and an Initial condition _.ic_ is used for that reason. It has also been implmented as a custom component in \Custom_Components\ltspice\ as vco.asy and vc0.sub and is used in the Spice Music Demo.
#### White Noise
LTSpice has a built-in white noise generator, _white()_. It takes time as an argument e.g. _white(k * time)_. The spectrum can be altered by varying the k factor.
### Voltage Controlled Filter
#### Diode Ladder Filter
A 4-pole diode ladder type filter is implmented in \Basic\vcf.asc. It produces nice sounds. It has also been implmented as a custom component in \Custom_Components\ltspice\ as vcf.asy and vcf.sub and is used in the Spice Music Demo.
#### TB-303 based filter
This is similar to the Diode Ladder Filter above and adds a "Gimmick", and some discretes taken from the schematic. It produces the 4-pole roll-off but the characteristic sounds cannot be reproduced currently. It does not implement the full schematic yet. More work required.
### Envelope Generator
The _voltage_ component has an _EXP_ mode which generates exponential waveforms suitable for an Attack-Release envelope generator. It is not repeatable out of the box but there are undocumented settings to enable this (see e.g. https://www.analog.com/en/technical-articles/ltspice-using-time-dependent-exponential-sources-to-model-transients.html). See \Basic\env_gen2.asc for an implmentation. Be careful not to right-click, editing the _EXP_ source and then saving it, this will cuase the undocumented values to be lost.
### The Behavioral Voltage Source
The Behavioral Voltage Source _bv_ is very useful. It can do advanced mathematical operations on circuit node voltages. Simple applications include VCA:s and mixers. A list of the available mathematical operations can be found in the LTSpice help. It is used extensively in the examples.
### Op Amps
While it can be tempting to use op amp models in simulations, these tend to slow down simulations considerably since their internal netlists can be large. Convergence is also an issue. Avoid them if possible. Buffers and amplifiers can be implmented using a Voltage Dependent Voltage Source _e_.
### DC blocking capacitors
To avoid startup settling transients avoid using DC blocking capacitors in the signal path. Settling transients can have much larger settling transients than the real signal, impacting dynamic range. It is better to remove the capacitor altogether and add an offset voltage source to get zero output DC voltage (see e.g. \Basic\tb303_vco_normalized.asc). Alternatively, if using capacitors, add intital conditions so the capacitors are charged from start.
## Notes on simulation
LTSpice saves all node voltages and branch currents in a _.raw_ file. The time resolution is typically much higher than the _.wave_ sampling frequency. This means that the _.raw_ file can grow very large (several GB). This file can safely be deleted since the result is stored in the_.wav_ file. The _.raw_ file is generated on every simulation run. 

The LTSpice simulation engine dynamically adjust the time stepping during simulation. The _.wave_ command resamples the waveforms to the desired sampling frequency.

Simulation time depends on circuit complexity. The Spice Music demo runs between 30-70 msec per second. Simulation can be sped up by altering the _trtol_ option. It is found in _Menu->Tools->Control Panel->SPICE_. A value of 10 can give 3 times faster simulations. Note that this has the downside of altering the characteristic sound of ladder filter resonances, as well as causing disturbances to discrete VCO:s (clicks, slight detuning and even halting). A value of 2 seems to be a good trade-off. This setting is remembered between Spice invocations. See Spice help for more information.
