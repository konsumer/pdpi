# pdpi

This is a raspberry pi puredata synth.

The basic idea is that you can run it headless, and use your MIDI keyboard to control every aspect of it. The modules that get loaded for different `PROGRAM_CHANGE` MIDI messages are located in `modules/`. It's designed to run in pd-extended.

So far, I have a simple demo `phasor~`, `osc~`, and bass-synth (like a 303.)

Making instruments is meant to be very simple.  There are 2 inlets and 2 outlets. top-left inlet is a note-pair (note-number and velocity) and the right inlet is mapped control pairs (eg "knob1 100".) Knob values are MIDI scale (0-127) and buttons are 0 or 1 (1 when pressed.)

Each instrument is loaded polyphonically (8 voices) and MIDI control messages are mapped, so you can make your own instrument map (knobs and buttons) for controls to make it easier to use with your MIDI controller. Open up `control_map.pd` in puredata to make a map for your own MIDI controller. I made one for my Oxygen 25.

These are the control-types I currently have:

```
mod - 0-127 modwheel
volume - 0-127 slider that makes sense to control main volume
knob1 - 0-127 basic instrument parameter
knob2 - 0-127 basic instrument parameter
knob3 - 0-127 basic instrument parameter
knob4 - 0-127 basic instrument parameter
knob5 - 0-127 basic instrument parameter
knob6 - 0-127 basic instrument parameter
knob7 - 0-127 basic instrument parameter
knob7 - 0-127 basic instrument parameter
button1 - 0/1 - trigger switch
button2 - 0/1 - trigger switch
button3 - 0/1 - trigger switch
button4 - 0/1 - trigger switch
button5 - 0/1 - trigger switch
button6 - 0/1 - trigger switch
```

These are just the controls that made sense for my controller, but I'm happy to add more so they become a standard that others can depend on in their layout for their instrument modules.

To use the synth, get MAIN.pd running on boot, and send it a `PROGRAM_CHANGE` message to switch instruments.

Open up the patch in Pd-extended from `modules/{PROGRAM NUMBER}.pd` to see how it works.

## module details

### 1. simple square-wave synth

This is just a simple `[phasor~]`.

### 2. simple sine-wave synth

This is just a simple `[osc~]`.

### 3. bass synth

This is sort of like a 303. It's an analog bass synth.

```
knob1 - filter cutoff
knob2 - filter resonance
knob3 - envelope mod
knob4 - rect width
knob5 - glide
button1 - select SAW wave
button2 - select SQR wave
button3 - select TRI wave
button4 - select SIN wave
button5 - enable n additional highpass filter
```


Eventually, I'll include more complete instructions for setting it all up on a raspberry pi, and making a custom module.

