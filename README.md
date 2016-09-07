# pdpi

This is a raspberry pi puredata synth.

[![intro video](https://img.youtube.com/vi/vhotjWj_JhU/0.jpg)](https://www.youtube.com/watch?v=vhotjWj_JhU)

This project is unrelated to [this other pdpi](http://pd-la.info/pdpi/), which I found after I named it.

It uses Puredata vanilla 0.46.2 (the version available in debian repos.)

The basic idea is that you can run it headless, and use your MIDI keyboard to control every aspect of it. The modules that get loaded for different `PROGRAM_CHANGE` MIDI messages are located in `modules/`. It's designed to run in pd-extended.

So far, I have a simple demo `phasor~`, `osc~`, and bass-synth (like a 303.)

Making instruments is meant to be very simple.  There are 2 inlets and 2 outlets. top-left inlet is a note-pair (note-number and velocity) and the right inlet is mapped control pairs (eg "knob1 100".) Knob values are MIDI scale (0-127) and buttons are 0 or 1 (1 when pressed.)

Each instrument is loaded polyphonically (8 voices) and MIDI control messages are mapped, so you can make your own instrument map (knobs and buttons) for controls to make it easier to use with your MIDI controller. Open up `control_map.pd` in puredata to make a map for your own MIDI controller. I made one for my Oxygen 25.

These are the control-types I currently have:

```
mod - 0-127 modulation wheel
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

These are just the controls that made sense for my controller, but I'm happy to add more so they become a standard that others can depend on in their layout for their instrument modules. I try to use sensible defaults, so synths will sound good without having the parameter control available on any given hardware.

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
button5 - enable an additional highpass filter
```


## Installing on the raspberry pi

* Install [minibian](https://minibianpi.wordpress.com/download/). I am on a Mac, so I used [Apple Pi Baker](http://www.tweaking4all.com/software/macosx-software/macosx-apple-pi-baker/), but here are [instructions for others](https://www.raspberrypi.org/documentation/installation/installing-images/).
* [Resize your partition](https://minibianpi.wordpress.com/how-to/resize-sd/)
* `apt-get update && apt-get upgrade`
* `apt-get install xauth nano unzip alsa-utils git`
* Run `git clone https://github.com/konsumer/pdpi.git /pdpi`
* Run `apt-get install puredata pd-zexy pd-bassemu pd-aubio pd-csound pd-cyclone puredata-utils puredata-extra pd-pdp pd-plugin`


I tested sound like this:

```
wget http://www.kozco.com/tech/piano2.wav
aplay piano2.wav
```

I heard a nice piano sound through the headphone jack.

After all this, let's get puredata running our `MAIN.pd` on boot. `nano /etc/rc.local` and make it look like this:

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

/usr/bin/printf "         My IP address is\033[0;31m `/sbin/ifconfig | grep "inet addr" | grep -v "127.0.0.1" | awk '{ print $2 }' | awk -F: '{ print $2 }'` \033[0m\n" > /dev/console

/usr/bin/printf "         Starting PdPi...\n" > /dev/console
/usr/bin/pd -nogui -noadc -midiindev 1 /pdpi/MAIN.pd &

exit 0
```

If you have a Pi 3, you can [setup wifi & bluetooth](https://minibianpi.wordpress.com/how-to/rpi3/), if you want. I disabled wifi & ethernet by putting a `#` in front of non-`lo` stuff in `/etc/network/interfaces`, so it boots faster and doesn't try to connect to a network when it's being used as an instrument.


## additional plugins

Above, I instructed you to install `pd-plugin` which can load LADSPA plugins (using `plugin~` in puredata,) many of which are awesome building blocks for extremely complicated and efficient synth/effect modules. If you'd like a whole bunch of LADSPA plugins installed, run `apt-get install vco-plugins wah-plugins zam-plugins swh-plugins tap-plugins ste-plugins mcp-plugins omins liquidsoap-plugin-ladspa invada-studio-plugins-ladspa fil-plugins cmt caps bs2b-ladspa blop blepvco autotalent amb-plugins rev-plugins`

## speed tweaks

Get the fastest microSD card you can find. You can also make the pi boot faster by editing some files (with `nano`):

### /boot/config.txt

You can pverclock. Here is what it looks like for a pi 2:

```
arm_freq=900
```

and here is a pi 3:
```
gpu_mem=16
arm_freq=1300
over_voltage=5
gpu_freq=500

# sdram overclock
sdram_freq=500
sdram_schmoo=0x02000020
over_voltage_sdram_p=6
over_voltage_sdram_i=4
over_voltage_sdram_c=4
```

You may be able to get it higher. Check out [this for pi 3](https://www.raspberrypi.org/forums/viewtopic.php?f=66&t=138123) and [this for pi 2](https://haydenjames.io/raspberry-pi-2-overclock/)

### /boot/cmdline.txt
```
dwc_otg.lpm_enable=0 root=/dev/mmcblk0p2 rootfstype=ext4 rootflags=commit=120,data=writeback elevator=deadline noatime nodiratime  data=writeback rootwait quiet
```

### /etc/sysctl.conf:
```
vm.dirty_background_ratio = 20
vm.dirty_expire_centisecs = 0
vm.dirty_ratio = 80
vm.dirty_writeback_centisecs = 1200
vm.overcommit_ratio = 2
vm.laptop_mode = 5
vm.swappiness = 10
```

## remote editing

You can install Xwindows on your desktop computer and edit puredata patches remotely (running on pi.) This ensures that you know exactly what extensions are available.

### On Mac

Install [XQuartz](https://www.xquartz.org/)

Now, in a terminal, run `ssh -X root@YOURPI pd -noadc -alsa -midiindev 1`

### On Windows

Install [Xming](http://www.straightrunning.com/XmingNotes/) and [putty](http://www.putty.org/). Ssh into your pi and run `pd -noadc -alsa -midiindev 1`


## non-pi usage

The pdpi was designed to be a cheap & easy computer-based synth, but you might have an old computer laying around that you're not using, and it'd be cheaper to run it there than buy a new pi. We got you covered! Install [debian](https://www.debian.org/distrib/) or [ubuntu server](http://www.ubuntu.com/download/server), [install pd vanilla 0.47-1](http://msp.ucsd.edu/software.html), download the project to `/pdpi` the same as above, and edit `/etc/rc.local` the same as above, and you should be good to go!
