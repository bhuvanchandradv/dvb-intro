---
title: "MicroPython running on NXP VF6XX M4 Core"
date: 2017-07-19T08:15:33+08:00
tags: ["embedded", "micropython", "arm", "linux", "toradex"]
---

This is an experimental port of MicroPython for NXP VF6XX M4 Core.

[Source code](https://github.com/bhuvanchandra/micropython/tree/vf6xx_m4)

Supported features include:
- REPL (Python prompt) over UART2
- Garbage collector
- GPIO, UART support

## Build

The toolchain required is the Linaro ARM Embedded toolchain (e.g. 4.9 2015 Q3):

```bash
tar xjf ${HOME}/gcc-arm-none-eabi-4_9-2015q3-20150921-linux.tar.bz2
export PATH=$PATH:$HOME/gcc-arm-none-eabi-4_9-2015q3/bin/
```

Then build:

```bash
cd ${HOME}/micropython/vf6xx
make
```

## Booting the M4 Core

Two methods are supported:

**1. From U-Boot using `bootaux`:**

```bash
Colibri VFxx # fatload mmc 0:1 ${loadaddr} firmware.elf
Colibri VFxx # bootaux ${loadaddr}
```

**2. From Linux using remoteproc:**

Deploy the `.elf` to `/lib/firmware` and load the remoteproc modules:

```bash
root@colibri-vf:~# ln -s firmware.elf /lib/firmware/freertos-rpmsg.elf
root@colibri-vf:~# modprobe vf610_cm4_rproc
```

## REPL

Access the Python prompt over UART_B at 115200 baud:

```
MicroPython v1.9.1-166-ge190711f on 2017-07-19; Colibri VF61 with VF6XX_M4
>>> print("Hello World!")
Hello World!
```

![Screenshot](https://raw.githubusercontent.com/bhuvanchandra/images-repo/master/micropython-vf6xx_m4-screenshot.png)
