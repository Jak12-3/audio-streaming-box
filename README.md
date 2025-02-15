# network-streamer
Create an audiophile streaming music device with [moOde audio](https://moodeaudio.org/), controls and an LCD

<img src="images/player-front.png">

## About
This project started as a means for adding a network audio streaming device to my stereo system. These devices are typically [awesome but expensive](https://www.crutchfield.com/shopsearch/network_streamer.html?&pg=2) so I decided to build one using a Raspberry Pi, an old analog AM/FM tuner's case, and the open-source moOde software. Front panel buttons can be used to select playlists, start/stop music and move to the previous and next track. When integrated into an old stereo component, it fits nicely into a stereo system. I used an old Sony ST-JX411/412/421 series of tuners from the 1990s that are a nice size and have rotary tuning knob.

I hope to add more details and instructions in the future, but the below should get ambitious makers started!

This is an advanced project that requires soldering, tinkering, and some prior experience working with the Raspberry Pi (Python and Docker experience is helpful but not required). This document outlines the basics to get started and create a minumum build. However, if you get stuck or find an issue, feel free to add an issue to this repo!

## Hardware required

- A Raspberry Pi 4-based SBC (2, 4, or 8 GB of RAM) or a CM-4
- Pi accessories such as a MicroSD card and power supply
- An audio DAC HAT for the Pi such as [this](https://www.raspberrypi.com/products/dac-pro/) or [this](https://www.raspberrypi.com/products/dac-plus/) (I used a [digital output HAT](https://www.hifiberry.com/shop/boards/hifiberry-digi2-pro/) with an outboard DAC)
- An LCD alphanumeric display like [this one](https://www.adafruit.com/product/181)
- [i2c / SPI character LCD backpack](https://www.adafruit.com/product/292) for connecting the LCD display to the Pi
- To control the device with switches, you'll need one or more momentary SPST switches, wired into a matrix (see below)

## Setup

### Hardware

- Connect the LCD display to the backpack, and the backpack to the Pi using SPI. ([instructions](https://learn.adafruit.com/i2c-spi-lcd-backpack/python-circuitpython))
- If you have the room, the backpack can be soldered right onto the LCD.
- Connect the DAC HAT to the Pi.
- Wire up your switches in a [key matrix pattern](https://pcbheaven.com/wikipages/How_Key_Matrices_Works/) (see the use of diodes in the link to improve performance)
- Wire the rows and columns of the key matrix to the Pi's GPIO pins (which may be on the DAC HAT if using one of those)
- See the [pad4pi](https://pypi.org/project/pad4pi/) library for more information about the key matrix.
- If you want a "power on" button, add a momentary pushbutton that sends GPIO 3 to ground.
  
### Software

- Download a Moode Audio image for the Pi 4, flash it to a MicroSD card and insert it into the Pi.
- Power up the Pi and make sure Moode works and you get audio out (LCD and buttons will not initially be functional)
- If using LCD, use `raspi-config` to turn on SPI for the screen.
- In the moode UI, under "Local Services" in the "System" menu, turn on "Metadata file" and "LCD updater".
- SSH into your Moode Pi (usually `ssh pi@moode.local` with password `moodeaudio`)
- Install Docker on the pi: https://raspberrytips.com/docker-on-raspberry-pi/ - make sure it starts automatically
- Clone this repo to `/usr/src/app` on device. (You can delete the "alternate" folder.)
- Adjust [the code](https://github.com/alanb128/moode-box/blob/main/controller/flask-api.py#L83) in flask-api on the device to correspond to the GPIO pins you used for your key matrix rows and columns 
- Update `/var/local/www/commandw/lcd_updater.py` and `restart.sh` with the included modified files.
- Issue `docker compose up -d` to start the container. It should always load on its own going forward.

If you make a change to any of the files on the device after this process, issue `docker compose down` then `docker compose build` then `docker compose up`.

<img src="images/player-back.png">

## Reference files

These may be helpful for further programming in the future:

FYI: output of `cat  /var/local/www/currentsong.txt` for both playing a file then playing a stream:

```
file=NAS/Flacs/The Temptations/Vintage Gold/04 - Since I Lost My Baby.flac
artist=The Temptations
album=Vintage Gold
title=Since I Lost My Baby
coverurl=/coverart.php/NAS%2FFlacs%2FThe%20Temptations%2FVintage%20Gold%2F04%20-%20Since%20I%20Lost%20My%20Baby.flac
track=4
date=
composer=
encoded=File does not exist
bitrate=0 bps
outrate=0 bps
volume=100
mute=0
state=stop
```

```
/var/local/www/commandw/lcd_updater.py
/var/local/www/currentsong.txt
pi@moode-dev:~/moodisp $ cat  /var/local/www/currentsong.txt
file=https://ice.cr1.streamzilla.xlcdn.com:8000/sz=RCOLiveWebradio=mp3-192
artist=Radio station
album=RCO Live
title=Robert Schumann: Symphony No. 2 in C major, Op. 61 - RCO Amsterdam - John Eliot Gardiner (7.3.2010)
coverurl=imagesw%2Fradio-logos%2FRCO%20Live.jpg
track=
date=
composer=
encoded=VBR
bitrate=192 kbps
outrate=16 bit, 44.1 kHz, Stereo, 1.411 Mbps
volume=60
mute=0
state=play
```
