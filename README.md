# pdpi

This is a raspberry pi puredata synth.

The basic idea is that you can run it headless, and use your MIDI keyboard to control every aspect of it. The modules that get loaded for different `PROGRAM` MIDI messages are located in `modules/`. It's designed to run in pd-extended.

So far, I have a simple demo `phasor~`, `osc~`, and bass-synth (like a 303.)

Making instruments is meant to be very simple.  There are 2 inlets and 2 outlets. top-left inlet is a note-pair (note-number and velocity) and the right inlet is mapped control pairs (eg "knob1 100".) Knob values are MIDI scale (0-127) and buttons are 0 or 1 (1 when pressed.)

Each instrument is loaded polyphonically (8 voices) and MIDI control messages are mapped, so you can make your own instrument map (knobs and buttons) for controls to make it easier to use with your MIDI controller. Open up `control_map.pd` in puredata to make a map for your own MIDI controller. I made one for my Oxygen 25.

Eventually, I'll include more complete instructions for setting it all up on a raspberry pi, and making a custom module.