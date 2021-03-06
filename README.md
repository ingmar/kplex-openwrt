kplex for OpenWrt
=================

These files are for using the OpenWrt build system to build [kplex](http://www.stripydog.com/kplex/) for OpenWrt.

kplex is a program for combining and routing NMEA-0183 data to and from multiple sources and destinations.
By installing it on a USB-enabled router, it is possible to broadcast NMEA data over a local ethernet and/or wifi network, making the data available to computers, phones, and tablets connected to the network.

[For more information about kplex see http://www.stripydog.com/kplex/](http://www.stripydog.com/kplex/).


## Background

To run kplex on your router, it must be compiled for your router's architecture.
Rather than compiling software on the router directly, as we would on most computers, it is normal to use a process called cross-compiling whereby the software is compiled on a more powerful machine, but with your router's architecture as the target.

Fortunately, OpenWrt makes cross-compiling software very easy, using the [OpenWrt SDK](https://wiki.openwrt.org/doc/howto/obtain.firmware.sdk) which is pre-compiled for every supported architecture, saving you from having to build the whole system from scratch using the [OpenWrt Buildroot](https://wiki.openwrt.org/doc/howto/buildroot.exigence).

Please note that these instructions assume a certain amount of familiarity with the command line.


## Seting up the environment

1. Install the prerequisites required to run the OpenWrt build system.\
    This depends on your system. The requirements are the same as for the Buildroot and are [listed here](https://wiki.openwrt.org/doc/howto/buildroot.exigence).\
    The following will normally be sufficient:
    ```bash
    $ sudo apt-get update
    $ sudo apt-get install git build-essential ccache libssl-dev libncurses5-dev unzip gawk zlib1g-dev
    ```

2. Download and unpack the OpenWrt SDK.\
    You can find the SDK for your router's architecture from [the OpenWrt downloads page](https://downloads.openwrt.org/). The SDK download you need is found in the same directory as the firmware for your router.\
    For example, the OpenWrt 15.05.1 SDK for the common `ar71xx` architecture is found [here](https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/OpenWrt-SDK-15.05.1-ar71xx-generic_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64.tar.bz2).

3. Set up the SDK.\
    `cd` into the extracted `SDK` directory and execute the following command:
    ```bash
    $ make menuconfig
    ```
    A menu-based interface will appear. Choose the `<Exit>` option, then choose to `<Save>` the generated config.


## Building

1. Ensure you are in the root `SDK` directory.

2. Clone this repo into the `package/kplex` subdir:
    ```bash
    $ git clone https://github.com/caesar/kplex-openwrt.git package/kplex
    ```

3. Use `make` to build the `package/kplex/compile` target:
    ```bash
    $ make package/kplex/compile
    ```
    This will download the source code for kplex, cross-compile it for your router, and package it in an IPKG installer package.

    The package will be located in the `bin` subdirectory of the SDK. For my architecture the package is at `bin/ar71xx/packages/base/kplex_1.3.4-1_ar71xx.ipk`.


## Installing

1. Copy the package to your router: (change the filename and the IP address of your router as appropriate)
    ```bash
    $ scp bin/ar71xx/packages/base/kplex_1.3.4-1_ar71xx.ipk root@192.168.1.1:
    ```

2. SSH into your router and install the package using OPKG.\
    You have to run `opkg update` first to update the package lists, so that OPKG will know where to download the dependency `libpthreads`.
    ```bash
    $ ssh root@192.168.1.1
    $ opkg update
    $ opkg install kplex_1.3.4-1_ar71xx.ipk
    ```
    The commands above should result in `libpthreads` being downloaded and installed, and then kplex being installed.


## Running at Startup

If you want to run kplex automatically at startup, run the following command:
```bash
$ /etc/init.d/kplex enable
```
Likewise, to disable it, run:
```bash
$ /etc/init.d/kplex disable
```
If you have the LuCI web interface installed on your router, you can also enable and disable kplex from the `System > Startup` page.


## Configuration

The config file for kplex is located at `/etc/kplex.conf`.
You must edit this file to tell kplex what you want it to do.

The default config file is empty (all lines are commented out), so by default kplex will do nothing.

[For information on the config file syntax see the kplex website](http://www.stripydog.com/kplex/configuration.html).
