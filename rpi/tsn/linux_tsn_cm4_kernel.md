# Updating the Raspberry Pi Kernel

A recent side quest in my embedded development journey has been learning how to compile the Linux kernel. For this project, it would not be possible to proceed without some minor modifications to the configuration enabling the QoS features.

As I was working through the Linux TSN documentation, I was trying to enable the queueing disciplines (qdisc) and kept encountering an error:

TODO: get exact verbiage

tom@raspberrypi: ~ $ Error: specified qdisc not found

The previous qdisc MQPRIO worked no issue and the TSN documentation made it seem everything was installed. However after doing some digging and finding this [Raspberry Pi forum post](https://forums.raspberrypi.com/viewtopic.php?t=292448#p1767932) gave me a super quick way to check out the kernel configuration on my running system. Some quick Googling helped find the QoS options.

> The core kernel configuration option for enabling "QoS and/or fair queueing" (the subsystem that handles queuing disciplines like NET_SCH) is CONFIG_NET_SCHED.  

> Individual queuing disciplines (sch_* modules) have their own specific configuration options, such as CONFIG_NET_SCH_NETEM for the Network Emulator. 

So a quick `grep` search of the file for `NET_SCH_` helped immediately find the issue.

TODO: add photo

The three QoS features outlined in the TSN docs (CBS, ETF, and TAPRIO) all were disabled, cool! This led to a sanity check to my Linux guru coworkers who confirmed I did need to rebuild the kernel to enable these. It seemed simple enough and frankly it was, I just needed to find the settings in `menuconfig`.

### Configuring the Kernel

I navigated over the the Raspberry Pi [cross-compiling the kernel](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compile-the-kernel) documentation for a friendly reminder of the process. Also, I highly recommend finding yourself a Linux machine (Facebook Marketplace and eBay always have some good finds) and cross-compiling on that machine. I used an old work laptop and it's been an invaluable addition to Raspberry Pi development.

Walking through the cross-compile kernel instructions, I needed to enable the QoS features required by this setup.

TODO: add config photo