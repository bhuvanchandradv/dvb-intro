---
title: "libsoc Examples on Colibri VF61"
date: 2017-05-18T11:55:50+08:00
tags: ["embedded", "linux", "gpio", "spi", "i2c", "toradex", "yocto"]
---

Examples using [libsoc](https://github.com/jackmitch/libsoc) on a Colibri VF61 module with an Aster carrier board and Pioneer600 Raspberry Pi expansion cape.

## Hardware

- [Colibri VF61](https://www.toradex.com/computer-on-modules/colibri-arm-family/nxp-freescale-vybrid-vf6xx) — Cortex-A5 + Cortex-M4, SODIMM form factor
- [Aster Carrier Board](https://www.toradex.com/products/carrier-boards/aster-carrier-board) — Arduino Uno and Raspberry Pi headers
- [Pioneer600](http://www.waveshare.com/wiki/Pioneer600) — RPi expansion cape

![Hardware Setup](https://github.com/bhuvanchandra/images-repo/raw/master/images-aster-pioneer600/aster-pioneer600.jpg)

## Cross-compiling libsoc

Build and install the Yocto SDK first, then:

```bash
git clone -b master https://github.com/jackmitch/libsoc.git
cd libsoc
. /usr/local/oecore-x86_64/environment-setup-armv7at2hf-neon-angstrom-linux-gnueabi
autoreconf -i
./configure --host=arm-angstrom-linux-gnueabi \
    --prefix=/usr/local/oecore-x86_64/sysroots/armv7at2hf-neon-angstrom-linux-gnueabi/
make -j$(nproc)
sudo make install
```

## Cross-compiling the examples

```bash
git clone https://github.com/bhuvanchandra/libsoc-examples.git
cd libsoc-examples/ssd1306/
MACHINE=colibri-vf make
```

Copy the binary to the target via SSH and run.

> **Note:** A Yocto recipe for libsoc is also available in [OpenEmbedded layers](https://layers.openembedded.org/layerindex/recipe/20464/).
