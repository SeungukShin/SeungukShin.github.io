---
title: Switch
---

# Switch
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Enable Microphone form Earset

Find out audio devices
```sh
$ aplay --list-devices
**** List of PLAYBACK Hardware Devices ****
card 0: PCH [HDA Intel PCH], device 0: ALC295 Analog [ALC295 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
$ arecord --list-devices
**** List of CAPTURE Hardware Devices ****
card 0: PCH [HDA Intel PCH], device 0: ALC295 Analog [ALC295 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

Find out model name in `Documentation/sound/alsa/HD-Audio-Models.txt` of linux
```
ALC22x/23x/25x/269/27x/28x/29x (and vendor-specific ALC3xxx models)
======
  laptop-amic		Laptops with analog-mic input
  laptop-dmic		Laptops with digital-mic input
  alc269-dmic		Enable ALC269(VA) digital mic workaround
  alc271-dmic		Enable ALC271X digital mic workaround
  inv-dmic		Inverted internal mic workaround
  headset-mic		Indicates a combined headset (headphone+mic) jack
  headset-mode		More comprehensive headset support for ALC269 & co
  headset-mode-no-hp-mic Headset mode support without headphone mic
  lenovo-dock   	Enables docking station I/O for some Lenovos
  hp-gpio-led		GPIO LED support on HP laptops
  dell-headset-multi	Headset jack, which can also be used as mic-in
  dell-headset-dock	Headset jack (without mic-in), and also dock I/O
  alc283-dac-wcaps	Fixups for Chromebook with ALC283
  alc283-sense-combo	Combo jack sensing on ALC283
  tpt440-dock		Pin configs for Lenovo Thinkpad Dock support
```

Create `/etc/modprobe.d/alsa-base.conf` to describe module options
```conf
# options snd-hda-intel model=<model>
options snd-hda-intel model=dell-headset-multi
```

## Noise Reduction

Append following on `/etc/pulse/default.pa`
```
#Active Noise Removal
.ifexists module-echo-cancel.so
load-module module-echo-cancel aec_method=webrtc source_name=mic source_properties=device.description=MicHD
set-default-source "mic"
.endif
```

Restart pulseaudio
```sh
$ pulseaudio -k
```

# References

* [Ubuntu Documentation](https://help.ubuntu.com/community/HdaIntelSoundHowto)
* [Gentoo Forums](https://forums.gentoo.org/viewtopic-t-1065266-start-0.html)
* [linux v4.9.141](https://elixir.bootlin.com/linux/v4.9.141/source/Documentation/sound/alsa/HD-Audio-Models.txt)
* [Ask Ubuntu](https://askubuntu.com/questions/859104/echo-cancellation-and-noise-reduction-for-ubuntu-16-04-lts)
