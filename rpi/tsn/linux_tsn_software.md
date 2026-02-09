# Installing the Software
After feeling like I was riding down rainbow road, figuring our how to install the software brought me back down to Earth. Now granted, the [Linux TSN Documentation](https://tsn.readthedocs.io/) was a really great guide once I understood how to adopt it to my CM4 system.



### System Requirements
This tutorial has been validated on two desktop machines with Intel(R) Ethernet Controller I210 connected back-to-back and Raspberry Pi kernel version XXXX.

### My Install Journey

For the first general install:
```shell
sudo apt install build-essential git meson flex bison glib2.0 \
        libcmocka-dev autoconf libtool autopoint libncurses-dev \
        libpulse-dev
```

I received an error regarding the `glib2.0` :
```shell
configure: error: the glib 2.0 library is required 
```

The [notes from Debian](https://packages.debian.org/sid/libglib2.0-dev) encourage moving to the `libglib2.0-dev` package.

So the updated command became:
```shell
sudo apt install build-essential git meson flex bison libglib2.0-dev \
        libcmocka-dev autoconf libtool autopoint libncurses-dev \
        libpulse-dev
```

To install ALSA library (alsa-lib) on Debian/Ubuntu systems, use sudo apt-get install libasound2 for the runtime library or sudo apt-get install libasound2-dev for development files.