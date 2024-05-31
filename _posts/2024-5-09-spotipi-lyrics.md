---
layout: post
title: Display lyrics from Spotify with a Raspberry Pi and LED&nbsp;Panel
description: A guide to build a display for showing the lyrics of the song playing on your Spotify account. It uses a Raspberry Pi 3, RGB matrix bonnet, and LED matrix panel from Adafruit.
image: assets/images/spotipi-lyrics/hero-2.jpg
hide_image: true
comments: true
---

<div class="row">
    <div class="6u 12u$(small)">
    <div class="image fit">
    <img src="{% link assets/images/spotipi-lyrics/hero-1.jpg %}" alt="" />
    </div>
    </div>
    <div class="6u$ 12u$(small)">
    <div class="image fit">
    <img src="{% link assets/images/spotipi-lyrics/hero-2.jpg %}" alt="" />
    </div>
    </div>
</div>

## Intro
I like how Spotify shows the lyrics to the song you're listening to in the app, but I often play music over a smart speaker using Alexa/Siri/etc so I don't have the app open. And even if I did, holding my phone to see the lyrics is kind of annoying. I wanted an easier way to view them so I decided to build my own solution and found [Ryan Ward's project](https://github.com/ryanwa18/spotipi){:target="_blank"} as a great starting point.

The project uses a Raspberry Pi to connect to your Spotify account, fetch the song you're currently listening to, and display those lyrics on a 64x32 LED panel.

## Hardware Parts

1. [Raspberry Pi 3 - Model B - ARMv8 with 1G RAM](https://www.adafruit.com/product/3055){:target="_blank"}
1. [64x32 RGB LED Matrix - 5mm pitch](https://www.adafruit.com/product/2277){:target="_blank"}
1. [Adafruit RGB Matrix Bonnet for Raspberry Pi](https://www.adafruit.com/product/3211){:target="_blank"}
1. [5V 4A (4000mA) switching power supply - UL Listed](https://www.adafruit.com/product/1466){:target="_blank"}
1. [SD/MicroSD Memory Card (8 GB SDHC)](https://www.adafruit.com/product/1294){:target="_blank"}

## (Optional) Panel Enclosure

<img src="{% link assets/images/spotipi-lyrics/case.jpg %}" alt="3d printed case" />

If you have access to a 3d printer you can print a plastic case to house the Raspberry Pi behind the led panel. IMO it makes it easier to keep all the parts contained. I used this one from TomHammond:

[https://www.thingiverse.com/thing:3779303](https://www.thingiverse.com/thing:3779303){:target="_blank"}

Note: you only need to print 4 of the files for this project (Cover*RPi_side.stl, Cover_Battery_Side.stl, Frame_Battery_Side_Slots3.stl, and Frame_Rasp_Pi_Side_Slots3*.stl)

## Assembly

[https://learn.adafruit.com/adafruit-rgb-matrix-bonnet-for-raspberry-pi/driving-matrices](https://learn.adafruit.com/adafruit-rgb-matrix-bonnet-for-raspberry-pi/driving-matrices){:target="_blank"}

### 1. Connect bonnet to Raspberry Pi

<img src="{% link assets/images/spotipi-lyrics/rpi-bonnet.jpg %}" alt="bonnet connected to raspberry pi" />

Shut down your Pi and remove power. Plug the bonnet on so all the 2x20 pins go into the GPIO header.

### 2. Connect matrix power cable to terminal block

<img src="{% link assets/images/spotipi-lyrics/terminal-block.jpg %}" alt="bonnet terminal block" />

Your RGB matrix came with a red & black power cable. Plug the 4-pin MOLEX connector into the matrix. Then plug the red wire into the + side, and the black wire into the - side. You can either use the attached spade connectors as-is or strip them off.

### 3. Connect RGB matrix data cable to IDC

<img src="{% link assets/images/spotipi-lyrics/data-cable.jpg %}" alt="data cable" />

Connect one end to the matrix's INPUT side and the other end to the IDC socket on the bonnet.

### 4. Plug in the 5V DC power for the matrix

<img src="{% link assets/images/spotipi-lyrics/power.jpg %}" alt="power cable" />

Plug your 5V 4A wall adapter into the bonnet.

### 5. Check that the matrix plugs are installed and in the right location

<img src="{% link assets/images/spotipi-lyrics/back.jpg %}" alt="assembled hardware" />

The green light on the bonnet should be on. IDC goes into the INPUT side (look for any arrows, arrows point from INPUT side to OUTPUT).

## Install Software

### 1. Create Spotify Developer Application

<img src="{% link assets/images/spotipi-lyrics/create-app.png %}" alt="spotify create app page" />

Log in to [your Spotify account](https://developer.spotify.com/dashboard){:target="_blank"} and create a new app. Make sure to set the Redirect URI as http://127.0.0.1/callback.

### 2. Install Raspberry Pi OS on SD card

[Follow this guide](https://www.raspberrypi.com/documentation/computers/getting-started.html#install-an-operating-system){:target="_blank"}

Make sure you customize settings (hostname, username/password, wifi) and enable SSH so you can log in to the RPi remotely.

### 3. Install spotipi-lyrics on Raspberry Pi

Unplug the power cable and insert the SD card into the RPi. Then plug the power back in and wait a minute for the device to connect to your wifi network.

On your main computer, ssh into the RPi (format is \_username\_@_hostname\_.local):

    $ ssh matthewsperry@rpi.local
    matthewsperry@rpi.local's password:
    Linux rpi 6.1.21-v7+ #1642 SMP Mon Apr  3 17:20:52 BST 2023 armv7l

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Thu May  9 01:14:38 2024
    matthewsperry@rpi:~ $

Clone the repo:

    matthewsperry@rpi:~ $ git clone https://github.com/sperrow/spotipi-lyrics

Run the generation token script and enter the prompted credentials (from the Spotify app you created):

    matthewsperry@rpi:~ $ cd spotipi-lyrics
    matthewsperry@rpi:~/spotipi-lyrics $ bash generate-token.sh

1. **Spotify Client ID:** the id created on the Spotify developer dashboard
1. **Spotify Client Secret:** the secret token created on the Spotify developer dashboard
1. **Spotify Redirect URI:** http://127.0.0.1/callback (fyi if you copy/paste this url from the app page you might need to remove a line break)
1. **Spotify username:** the username for your Spotify account

You should see this message:

    Go to the following URL: https://accounts.spotify.com/authorize?client_id=...
    Enter the URL you were redirected to:

Open the link in your browser to authorize the app - once you do you then need to paste the redirected url back here. If everything worked a .cache file should have been created.

Now run the installation script:

    matthewsperry@rpi:~/spotipi-lyrics $ sudo bash setup.sh

Enter the same information from the previous step, as well as these:

1. **Full path to your spotify token:** the path to the .cache file on your raspberry pi (for example: /home/matthewsperry/spotipi-lyrics/.cache)
1. **sp_dc cookie:** Syrics sp_dc cookie to authenticate against Spotify in order to have access to the required services, [use this guide](https://github.com/akashrchandran/syrics/wiki/Finding-sp_dc){:target="_blank"}

## Thanks

Much thanks to [Ryan Ward](https://www.rwardtech.com/){:target="_blank"} for the base project this repo is forked from, and to [Akash R Chandran](https://akashrchandran.in/){:target="_blank"} for building the lyrics fetcher.

## Video

{% include youtube.html id="PKM1I0vdbhE" %}
