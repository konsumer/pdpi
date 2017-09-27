# pdpi

This is a raspberry pi puredata synth.

[![intro video](https://img.youtube.com/vi/vhotjWj_JhU/0.jpg)](https://www.youtube.com/watch?v=vhotjWj_JhU)

This project is unrelated to [this other pdpi](http://pd-la.info/pdpi/), which I found after I named it. If anyone has any suggestions for a better name, I'm so in!

It uses Puredata vanilla 0.46.2 (the version available in debian repos.)

The basic idea is that you can run it headless, and use your MIDI keyboard to control every aspect of it. The modules that get loaded for different `PROGRAM_CHANGE` MIDI messages are located in `modules/`.

So far, I have a simple demo `phasor~`, `osc~`, a bass-synth (like a 303,) a rhodes-like fm synth, and a super-saw washy witch-house type synth.

Making instruments is meant to be very simple.  There are 2 inlets and 2 outlets. top-left inlet is a note-pair (note-number and velocity) and the right inlet is mapped control pair messages (eg "knob1 100".) Knob values are MIDI scale (0-127) and buttons are 0 or 127 (127 when pressed.)

MIDI control messages are mapped, so you can make your own instrument map (knobs and buttons) for controls to make it easier to use with your MIDI controller. Open up `control_map.pd` in puredata to make a map for your own MIDI controller. I made one for my Oxygen 25.

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
knob8 - 0-127 basic instrument parameter
button1 - 0/127 - trigger switch
button2 - 0/127 - trigger switch
button3 - 0/127 - trigger switch
button4 - 0/127 - trigger switch
button5 - 0/127 - trigger switch
button6 - 0/127 - trigger switch
```

These are just the controls that made sense for my controller, but I'm happy to add more so they become a standard that others can depend on in their layout for their instrument modules. I try to use sensible defaults, so synths will sound good without having the parameter control available on any given hardware.

To use the synth, get MAIN.pd running on boot, and send it a `PROGRAM_CHANGE` message to switch instruments.

Open up the patch in puredata from `modules/{PROGRAM NUMBER}/main.pd` to see how it works.

If you don't have a MIDI keyboard handy, you can open `modules/vcontroller.pd` to send messages using your computer keyboard.

## module details

### 1. simple saw-wave synth

This is just a simple `[phasor~]`. It's made to load 8-voice polyphony, so you can see how that works.

### 2. simple sine-wave synth

This is just a simple `[osc~]` with 1 voice.

### 3. bass synth

This is sort of like a 303. It's an analog bass synth with 1 voice.

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

### 4. rhodey

Sort of like a rhodes, with 4 voice polyphony.

```
mod - modulation depth
knob1 - tone
knob2 - decay
knob3 - vibe frequency
knob4 - vibe depth
```

### 5. super-saw

Badass phat saw sound. Based on [this](http://www.ghostfact.com/jp-8000-supersaw/) rad breakdown. Setup with 4 voice polyphony.

```
knob1 - detune
```

## Installing on the raspberry pi

* Install [minibian](https://minibianpi.wordpress.com/download/). I am on a Mac, so I used [Apple Pi Baker](http://www.tweaking4all.com/software/macosx-software/macosx-apple-pi-baker/), but here are [instructions for others](https://www.raspberrypi.org/documentation/installation/installing-images/). Make a blank file in the root of the SD called `ssh` (with no extension,) the contents don't matter.
* If you don't have a monitor/keyboard, make a simple network with [just a cable](https://pihw.wordpress.com/guides/direct-network-connection/)
* Login with `root` (password: `raspberry`)
* `apt update && apt install raspi-config` and resize your partition with `raspi-config --expand-rootfs`
* Reboot and login to pi
* `apt upgrade`
* `apt install nano unzip alsa-utils git`
* Run `git clone https://github.com/konsumer/pdpi.git /pdpi`
* Run `apt install puredata gem puredata-utils puredata-extra puredata-import pd-3dp pd-arraysize pd-aubio pd-bassemu pd-beatpipe pd-boids pd-bsaylor pd-chaos pd-comport pd-csound pd-cxc pd-cyclone pd-earplug pd-ekext pd-ext13 pd-fftease pd-flite pd-freeverb pd-ggee pd-hcs pd-hid pd-iemambi pd-iemlib pd-iemmatrix pd-iemnet pd-jmmmp pd-libdir pd-list-abs pd-lua pd-lyonpotpourri pd-mapping pd-markex pd-maxlib pd-mjlib pd-moonlib pd-motex pd-osc pd-pan pd-pddp pd-pdogg pd-pdp pd-pdstring pd-plugin pd-pmpd pd-purepd pd-readanysf pd-sigpack pd-smlib pd-unauthorized pd-vbap pd-wiimote pd-windowing pd-zexy`


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

If you have a Pi 3, you can [setup wifi & bluetooth](https://minibianpi.wordpress.com/how-to/rpi3/), if you want. I didn't do this because I want it to boot faster, and I can plug it in when I need networking.


## additional plugins

Above, I instructed you to install `pd-plugin` which can load LADSPA plugins (using `plugin~` in puredata,) many of which are awesome building blocks for extremely complicated and efficient synth/effect modules. If you'd like a whole bunch of LADSPA plugins installed, run `apt install vco-plugins wah-plugins zam-plugins swh-plugins tap-plugins ste-plugins mcp-plugins omins liquidsoap-plugin-ladspa invada-studio-plugins-ladspa fil-plugins cmt caps bs2b-ladspa blop blepvco autotalent amb-plugins rev-plugins wah-plugins fil-plugins`. After this, you can run [this script](https://gist.github.com/konsumer/84ebf8837cdd80fde839) to generate some nice commented patches all ready for pdpi! You will probably have to tweak some of them a bit to make work right (toggle might be better than slider, range might not be good)but for the most part they just work.

Here is how to get it on the pi and run it:

```
cd /pdpi/lib
wget https://gist.githubusercontent.com/konsumer/84ebf8837cdd80fde839/raw/ca79c60c8a54f053f3c117b06392b0956f9cbdc3/wrap_ladspa.py
mkdir ladspa
apt install python
python wrap_ladspa.py
```

Now you can add `[ladpsa/whatever]` to your patches.

## speed tweaks

Get the fastest microSD card you can find. You can also make the pi boot faster by editing some files (with `nano`):

### /boot/config.txt

You can overclock. Here is what it looks like for a pi 2:

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

You may be able to get it higher. Check out [this for pi 3](https://www.raspberrypi.org/forums/viewtopic.php?f=66&t=138123) and [this for pi 2](https://haydenjames.io/raspberry-pi-2-overclock/). I managed to use these settings without a heatsink, or any change in the hardware.

### /boot/cmdline.txt
```
dwc_otg.lpm_enable=0 root=/dev/mmcblk0p2 rootfstype=ext4 rootflags=commit=120,data=writeback elevator=deadline noatime nodiratime data=writeback rootwait quiet
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

I also sped it up a little, and disabled the wait for network with this in my `/etc/network/interfaces`:

```
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
iface eth0 inet dhcp
```

## remote editing

You can install Xwindows on your desktop computer and edit puredata patches remotely (running on pi.) This ensures that you know exactly what puredata extensions are available. If `-X` doesn't work on OSX or Linux, try `-YC`.

### On Linux

In a terminal, run `ssh -X root@YOURPI pd -noadc -alsa -midiindev 1`

### On Mac

Install [XQuartz](https://www.xquartz.org/)

Now, in a terminal, run `ssh -X root@YOURPI pd -noadc -alsa -midiindev 1`

### On Windows

Install [Xming](http://www.straightrunning.com/XmingNotes/) and [putty](http://www.putty.org/). Ssh into your pi and run `pd -noadc -alsa -midiindev 1`. I don't have a Windows box handy, so I'd appreciate a Windows-person documenting this.


## non-pi usage

The pdpi was designed to be a cheap & easy computer-based synth, but you might have an old computer laying around that you're not using, and it'd be cheaper to run it there than buy a new pi. We got you covered! Install [debian](https://www.debian.org/distrib/) or [ubuntu server](http://www.ubuntu.com/download/server), Install puredata and extensions, as above, download this project to `/pdpi` the same as above, and edit `/etc/rc.local` the same as above, and you should be good to go!
