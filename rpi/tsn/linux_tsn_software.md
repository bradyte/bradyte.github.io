# Installing the Software
After feeling like I was riding down rainbow road, figuring our how to install the software brought me back down to Earth. Now granted, the [Linux TSN Documentation](https://tsn.readthedocs.io/) was a really great guide once I understood how to adopt it to my CM4 system.



### System Requirements
This tutorial has been validated on two desktop machines with Intel(R) Ethernet Controller I210 connected back-to-back and Raspberry Pi kernel version `Linux raspberrypi 6.12.47+rpt-rpi-v8 #1 SMP PREEMPT Debian 1:6.12.47-1+rpt1 (2025-09-16) aarch64 GNU/Linux`.

### My Install Journey
I will be walking through the [Linux TSN Documentation](https://tsn.readthedocs.io/) and updating the commands for sections I needed to update. Otherwise, the existing commands are correct.

As with any Pi project, update to with the latest security patches and software updates.

```shell
sudo apt update && sudo apt upgrade
```

Starting from the [Plugin Installation](https://tsn.readthedocs.io/avb.html#plugins-installation), I installed the necessary packages:
```shell
sudo apt install build-essential git meson flex bison \
        libcmocka-dev autoconf libtool autopoint libncurses-dev \
        libpulse-dev
```
**Info:**  *I received an error regarding the `glib2.0` that the system was unable to locate the package. The [notes from Debian](https://packages.debian.org/sid/libglib2.0-dev) encourage moving to the `libglib2.0-dev` package which is installed as a dependency package anyway.*

Next, it's required have the `libavtp` package installed when compiling ALSA and Gstreamer. I found a good explanation from [ÆSR Lab - Sound Project Labratory](https://www.mdw.ac.at/aesr-lab/docs/A-System/Networked-audio/automotive-avb/) about the correct order of operations.

I proceeded to install the `libavtp` package.
```shell
git clone https://github.com/Avnu/libavtp.git
cd libavtp/
meson setup build
sudo ninja -C build install
```
**Info:** *When running just `meson build`, I received an error saying that running the setup command without the word `setup` is deprecated so I updated the command.*

### ALSA Plugin
To install ALSA library (alsa-lib) on Debian/Ubuntu systems, use sudo apt-get install libasound2 for the runtime library or sudo apt-get install libasound2-dev for development files.

TODO: confirm alsa-lib vs libasound2

The ALSA project includes gitcompile script which is recommended to use by the INSTALL documentation so the commands were updated.

#### Step 1: Start with the core libraries from alsa-lib project:
```shell
git clone https://github.com/alsa-project/alsa-lib.git
cd alsa-lib/
./gitcompile --prefix=/usr/local
sudo make install
```

#### Step 2: Install the utility tools from alsa-utils project:

```shell
git clone https://github.com/alsa-project/alsa-utils.git
cd alsa-utils/
./gitcompile --prefix=/usr/local --disable-nls
sudo make install
```
**Info:** *For `alsa-utils` install, I added `--disable-nls` to `./gitcompile` which removed some NLS errors*


#### Step 3: Install the plugins from alsa-plugins project
```shell
git clone https://github.com/alsa-project/alsa-plugins.git
cd alsa-plugins/
./gitcompile 
./configure --prefix=/usr/local --enable-aaf
sudo make install
```
**Info:** *For `alsa-plugins` install, I added `--enable-aaf` to `./gitcompile`  to be able to install the plugins alongside system packages*

TODO: figure out why I need to
```shell
#sudo cp -a /usr/local/lib/alsa-lib/. /lib/aarch64-linux-gnu/alsa-lib/
# I may only need to do this one
sudo cp -a /usr/local/lib/. /lib/aarch64-linux-gnu/

```

You can confirm everything is correct so far looking at the console output and seeing: 
```shell
AAF plugin:         yes
```
#### Step 4: Regenerate the shared library cache after manually installing libraries
```shell
sudo ldconfig
```

#### VLAN Configuration
All the commands worked for me but on the Pi, the I210 NIC was the `eth1` interface so I changed all the commands to match. This caused a lot of debugging issues missing this detail.

```shell
sudo ip link add link eth1 name eth1.5 type vlan id 5 \
        egress-qos-map 2:2 3:3

sudo ip link set eth1.5 up
```

### Qdiscs Configuration
Note: these qdiscs enable a transmission algorithm and should be configured on transmitting end-stations (Talker systems). They are not required on the receiving end-stations (Listener systems).
#### MQPRIO Configuration
```shell
sudo tc qdisc add dev eth1 parent root handle 6666 mqprio \
        num_tc 3 \
        map 2 2 1 0 2 2 2 2 2 2 2 2 2 2 2 2 \
        queues 1@0 1@1 2@2 \
        hw 0
```
#### CBS Configuration
```shell
sudo tc qdisc replace dev eth1 parent 6666:1 handle 7777 cbs \
        idleslope 98688 sendslope -901312 hicredit 153 locredit -1389 \
        offload 1

sudo tc qdisc replace dev eth1 parent 6666:2 handle 8888 cbs \
        idleslope 3648 sendslope -996352 hicredit 12 locredit -113 \
        offload 1
```

#### ETF Configuration
**error** does not like this for some reason

```shell
sudo tc qdisc add dev eth1 parent 7777:1 etf \
        clockid CLOCK_TAI \
        delta 500000 \
        offload

sudo tc qdisc add dev eth1 parent 8888:1 etf \
        clockid CLOCK_TAI \
        delta 500000 \
        offload
````

#### Cyclic test
https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/rt-tests#compile-and-install

sudo apt install rt-tests

#### TAPRIO Configuration
````shell
sudo tc qdisc replace dev eth1 parent root handle 100 taprio \
        num_tc 3 \
        map 2 2 1 0 2 2 2 2 2 2 2 2 2 2 2 2 \
        queues 1@0 1@1 2@2 \
        base-time 1000000000 \
        sched-entry S 01 300000 \
        sched-entry S 03 300000 \
        sched-entry S 04 400000 \
        flags 0x1 \
        txtime-delay 500000 \
        clockid CLOCK_TAI
````

#### ETF Offload
````shell
sudo tc qdisc replace dev eth1 parent 100:1 etf \
        clockid CLOCK_TAI \
        delta 500000 \
        offload \
        skip_sock_check
````

note may need to restart ptp4l or wait until everything is configured

### Time Synchronization

```shell
# run on both
sudo ptp4l -i eth1 -f ~/src/linuxptp/configs/gPTP.cfg --step_threshold=1 -m

sudo timedatectl set-ntp false

sudo pmc -u -b 0 -t 1 "SET GRANDMASTER_SETTINGS_NP clockClass 248 \
        clockAccuracy 0xfe offsetScaledLogVariance 0xffff \
        currentUtcOffset 37 leap61 0 leap59 0 currentUtcOffsetValid 1 \
        ptpTimescale 1 timeTraceable 1 frequencyTraceable 0 \
        timeSource 0xa0"

sudo phc2sys -s eth1 -c CLOCK_REALTIME --step_threshold=1 \
        --transportSpecific=1 -w -m

sudo ptp4l -i eth1 -f ~/src/linuxptp/configs/automotive-master.cfg --step_threshold=1 -m
sudo ptp4l -i eth1 -f ~/src/linuxptp/configs/automotive-slave.cfg --step_threshold=1 -m
```


Confirm NTP is not running!!!
An NTP service may be running and changing the system clock. On systems with systemd, run:





### GStreamer examples

```shell
sudo gst-launch-1.0 clockselect. \( clock-id=realtime \
    audiotestsrc samplesperbuffer=12 is-live=true ! \
    audio/x-raw,format=S16BE,channels=2,rate=48000 ! \
    avtpaafpay mtt=50000000 tu=1000000 streamid=0xAABBCCDDEEFF000B processing-deadline=0 ! \
    avtpsink ifname=eth1.5 address=01:AA:AA:AA:AA:AA priority=2 processing-deadline=0 \)

sudo gst-launch-1.0 clockselect. \( clock-id=realtime \
    avtpsrc ifname=eth1.5 address=01:AA:AA:AA:AA:AA ! \
    queue max-size-buffers=0 max-size-time=0 ! \
    avtpaafdepay streamid=0xAABBCCDDEEFF000B ! audioconvert ! autoaudiosink \)


export GST_DEBUG=3
# Or a higher level for more detail:
export GST_DEBUG=4
# Then run your GStreamer command or application
gst-launch-1.0 [your pipeline description]

for utils add 
apt-get install gstreamer1.0-plugins-base-apps
```

The GStreamer Way
From a WAV File
To stream contents from a WAV file, use the filesrc element and run the command:

sudo gst-launch-1.0 clockselect. \( clock-id=realtime \
    filesrc location=~/Flamagra-01-004-More.wav ! wavparse ! audioconvert ! \
    audiobuffersplit output-buffer-duration=12/48000 ! \
    avtpaafpay mtt=50000000 tu=1000000 streamid=0xAABBCCDDEEFF000B processing-deadline=0 ! \
    avtpsink ifname=eth1.5 address=01:AA:AA:AA:AA:AA priority=2 processing-deadline=0 \)


## removing qdiscs
sudo tc qdisc del dev eth1 root  
sudo tc -d qdisc show dev eth1

