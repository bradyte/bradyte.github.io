# Audio Video Transport Protocol on Raspberry Pi CM4

### Introduction

In an effort to better understand the IEEE1722-2016 protocol, I wanted a nice and simple setup to watch the packets flowing through the network. There are a lot of bits and pieces of this information scattered across the internet, so I wanted to document how to accomplish this in 2026.

The two pillars of this setup are the Avnu Alliance [ libavtp repo](https://github.com/Avnu/libavtp) and [The TSN Documentation Project for Linux](https://tsn.readthedocs.io/) from Intel. They're both fairly well documented but I personally needed some additional notes to get this working. 

There are three main portions of this setup are:
* [Hardware Configuration](https://bradyte.github.io/rpi/tsn/linux_tsn_hardware)
* [Kernel Configuration](https://bradyte.github.io/rpi/tsn/linux_tsn_cm4_kernel)
* [Software Installation](https://bradyte.github.io/rpi/tsn/linux_tsn_software)

Credit to:  
TSN project for linux  
libavtp  
ALSA  
gstreamer  
Jeff Geerling  
Lassel Johnson/Timebeat
Austin's Nerdy Things

### IEEE1722 Background

[add brief history]

