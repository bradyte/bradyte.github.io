# Hardware Configuration for libavtp

AVTP implementation leverages quality-of-service (QoS) features from 802.1Q and the generalized Precision Time Protocol (PTP or gPTP) from 802.1AS.

Specifically, an AVTP end station that transmits or receives fixed latency streams need the following services:
* Time synchronization (802.1AS)
* Bandwidth reservation (802.1Qat)
* Traffic queueing and shaping (802.1Qav)

As cool as these sound, it requires specific Ethernet controllers that support these hardware features because most applications can tolerate milliseconds of latency. But for time critical applications, milliseconds is an eternity for needing to react to information.

Therefore, we can leverage the hardware to organize the data for transport and guarantee the latency required for the system. 

### Finding the Right Controller
As always, I like to try and use a Raspberry Pi from the myriad of boards lying around my office because more than likely, someone on the Raspberry Pi forums has a thread troubleshooting a bring up. 

So I looked into the Ethernet controllers on the Pi 4 and Pi 5. For both models, the Ethernet controller is integrated into the respective SoCs. They are fairly simple but [this github issue](https://github.com/raspberrypi/linux/issues/4151) details some more details on adding in more advanced features like IEEE1588 PTP. That's where my journey started thinking I could leverage the Pi 5 on my bench since it does support the hardware timestamping needed for TSN. Quickly realized that it's just one piece to the puzzle because it did not support the other QoS features needed.

However, this was one of the first rabbit holes I ventured down because two of my favorite engineers([Jeff Geerling](https://www.jeffgeerling.com/) and [Austin Pivarnik](https://austinsnerdythings.com/)) had some great details on implementing PTP with the standard Pi 5 hardware.

For more info, highly recommend these setups for a quick start to understanding PTP:

* [PTP and IEEE-1588 hardware timestamping on the Raspberry Pi CM4](https://www.jeffgeerling.com/blog/2022/ptp-and-ieee-1588-hardware-timestamping-on-raspberry-pi-cm4/)

* [Nanosecond accurate PTP server (grandmaster) and client tutorial for Raspberry Pi](https://austinsnerdythings.com/2025/02/18/nanosecond-accurate-ptp-server-grandmaster-and-client-tutorial-for-raspberry-pi/)

With the native Pi hardware not supporting what is needed, I took some time to slow down and read the TSN documentation which had a suggested controller right in the beginning...

> This tutorial has been validated on two desktop machines with **Intel(R) Ethernet Controller I210** connected back-to-back and Linux kernel version 4.19.

The I210 is a feature rich controller that's easy to acquire and fairly cheap. To me, this has always been important when finding hardware because it means that more people can have access to demo setups for learning.

I found two versions of this controller in both mini-PCIe and NIC form factor. The mini-PCIe would easily work on Pi 5 and the NIC form factor works on the CM4 and CM5 IO controller board

TODO: add links and photos

### Setting Up the Hardware
Always my favorite part of any demo setup is connecting all the hardware together, likely from growing up loving (and still loving) Legos.

I had a few ideas on how to connect in the I210 into the Pi. The first idea was to connect the controller via a PCIe card slot connected to PCIe connector on the Pi 5. I bought a cheap 2.5GbE Realtek 8125 controller to validate basic functionality. 

TODO: add photo

Once that worked, I naively thought anything would just work. Unfortunately, the first hat I bought was DOA. 

TODO: add photo

Rather than spending too much time debugging it, I moved to a different form factor with a mini-PCIe hat and the controller was detected right away.

TODO: add photo

Now I still had the I210 NIC and wanted to still try and use it, call me stubborn I guess. I knew it would work, just wanted some guidance before my next hardware procurement. Doing a little navigation through forums and blog posts, I stumbled upon a [GitHub discussion](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/204) from Jeff Geerling and Lassel Johnson from [Timebeat](https://www.timebeat.app/) showing the exact setup I was thinking of. Thanks both!

So if people exponentially smarter than me got it working, I have a chance. Plus, I've always wanted a reason to by a CM4 I/O board so it was rather serendipitous.

In fact, I was so excited, I bought the CM4 modules without wireless support which is typically no major issue but having wireless access allowed me to dedicate the Ethernet controlled to AVB traffic. One finally order getting the correct CM4 and my setup was ready to go.

TODO: add photo