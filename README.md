tinySA - tiny Spectrum Analyzer
==========================================================

[![GitHub release](http://img.shields.io/github/release/erikkaashoek/tinySA.svg?style=flat)][release]
[![CircleCI](https://circleci.com/gh/erikkaashoek/tinySA.svg?style=shield)](https://circleci.com/gh/erikkaashoek/tinySA)

[release]: https://github.com/erikkaashoek/tinySA/releases

<div align="center">
<img src="/doc/tinySA.jpg" width="480px">
</div>

# This fork: tinySA Ultra+ ZS407 (STM32F303CBT6) speed build

Branch `zs407-speed` of this fork is the tinySA firmware built for speed on the STM32F303 (Cortex-M4F) of the tinySA Ultra / Ultra+ (ZS405, ZS406, ZS407). All functions of the original firmware are kept.

**Get the firmware:** every push is built by GitHub Actions with the same compiler as the official firmware (Arm GNU Toolchain 11.3.Rel1). Open *Actions*, select the latest run of *Build firmware* and download the artifact `tinySA4-zs407-firmware`. It contains `tinySA4.bin` (SD card update or `dfu-util`), `.hex`, `.elf` and `.map`.

**What is changed compared to the original firmware**

* The measurement loop (`perform()` and `sweep()` in `sa_core.c`) and the trace and cell rendering (`plot.c`) are compiled with `-O2` instead of `-Os`.
* Trace averaging (AVER 4, AVER 16, linear averaging) and the search for the trace minimum use the single precision FPU of the Cortex-M4F. The original code did these per point in software double precision.
* Nothing is changed in the programming of the PLL and the Si4468, in the settle delays, in the level calibration or in the layout of the stored configuration. Settings, calibration and presets are kept when this firmware is installed.

**What limits the sweep speed**

For every measured step the firmware programs the first LO PLL, waits for the PLL and the RSSI detector to settle (150 us at an RBW of 300 kHz and more, up to 18.7 ms at an RBW of 300 Hz) and then reads the RSSI from the Si4468. The largest RBW is 850 kHz, so a wide span needs many steps per displayed point. These times belong to the RF hardware. The MCU spends most of a sweep waiting for it, so faster code can only gain a little: the estimate from the timing constants in the source is a few percent. The gain is not measured on a device yet, compare the sweep time shown on the screen with the original firmware.

Settings that make the sweep faster, at the cost of level accuracy:

* `DISPLAY` > `SWEEP ACCURACY` > `FAST`: half the number of steps on wide spans.
* `DISPLAY` > `SWEEP ACCURACY` > `WSPEEDUP` (2..20): divides the settle delay. Check the level of a known signal after changing it.
* `DISPLAY` > `SWEEP POINTS`: fewer points, proportionally faster.
* A larger RBW when it is set manually.

# About

tinySA is very tiny handheld Spectrum Analyzer (SA). It is
standalone with lcd display, portable device with battery. This
project aim to provide an useful instrument for the RF 
enthusiast.

This repository contains source of tinySA firmware.

# Support

General tinySA support questions should be posted here: https://groups.io/g/tinysa/messages

Use github issue list only for firmware bugs and preferrably cross post to: https://groups.io/g/tinysa/messages

## Prepare ARM Cross Tools

**UPDATE**: Recent gcc version works to build tinySA, no need old version.

### MacOSX

Install cross tools and firmware updating tool.

    $ brew tap px4/px4
    $ brew install gcc-arm-none-eabi-80
    $ brew install dfu-util

### Linux (ubuntu)

Download arm cross tools from [here](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads).

    $ wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/8-2018q4/gcc-arm-none-eabi-8-2018-q4-major-linux.tar.bz2
    $ sudo tar xfj gcc-arm-none-eabi-8-2018-q4-major-linux.tar.bz2 -C /usr/local
    $ PATH=/usr/local/gcc-arm-none-eabi-8-2018-q4-major/bin:$PATH
    $ sudo apt install -y dfu-util

## Fetch source code

Fetch source and submodule.

    $ git clone https://github.com/erikkaashoek/tinySA.git
    $ cd tinySA
    $ git submodule update --init --recursive

## Build

Just make in the directory.

    $ make

For tinySA Ultra use this command

    $ make TARGET="F303"

### Build firmware using docker

Using [this docker image](https://hub.docker.com/r/edy555/arm-embedded) and without installing arm toolchain, you can build the firmware.

    $ cd tinySA
    $ docker run -it --rm -v $(PWD):/work edy555/arm-embedded:8.2 make

## Flash firmware

First, make device enter DFU mode by one of following methods.

* Jumper BOOT0 pin at powering device
* Select menu Config->DFU (needs recent firmware)

Then, flash firmware using dfu-util via USB.

    $ dfu-util -d 0483:df11 -a 0 -s 0x08000000:leave -D build/ch.bin

Or simply use make.

    $ make flash

### Update from the SD card (tinySA Ultra)

The tinySA Ultra can install a new firmware from its SD card, without DFU:

1. Copy the firmware `.bin` file to the root folder of the SD card, for example with STORAGE→USB DISK. Eject the drive when the copy is done.
2. Select STORAGE→UPDATE FIRMWARE, pick the file and confirm with UPDATE.
3. The tinySA checks the file, programs it in about 10 seconds and restarts. Do not switch it off or remove the SD card during the update. If the card is removed anyway, the update waits and continues when it is inserted again.

Settings, calibration and presets are kept. The shell command `fwupdate {file}` checks a file without changing anything, `fwupdate {file} flash` installs it.

Only firmware that contains the update header (from this version on) can be installed from the SD card; older versions and the tinySA basic firmware are refused. If an update is interrupted, the tinySA does not start: hold the jog button while switching on to enter DFU mode and flash the firmware with a PC as above.

## Companion Tools

There are several numbers of great companion PC tools from third-party.

* [Soon to come](https://github.com/erikkaashoek/tinySA-Win) by Erik
* [tinySASaver](https://github.com/erikkaashoek/tinySA-saver) by Erik

## Documentation

* [tinySA User Guide](https://tinySA.org/wiki/)

## Reference

* [Specification](https://tinysa.org/wiki/pmwiki.php?n=Main.Specification)
* [Technical info](https://tinysa.org/wiki/pmwiki.php?n=Main.TechnicalDescription)

## Note

tinySA is a trademark owned by its respective owner. Unauthorized use the the name tinySA not permitted

## Authorized Distributor

* [See the Wiki](https://tinysa.org/wiki/pmwiki.php?n=Main.Buying)

## Credit

* [@erikkaashoek](https://github.com/erikkaashoek)

### Contributors

* [@edy555](https://github.com/edy555)
* [@hugen79](https://github.com/hugen79)
* [@cho45](https://github.com/cho45)
* [@DiSlord](https://github.com/DiSlord/)
