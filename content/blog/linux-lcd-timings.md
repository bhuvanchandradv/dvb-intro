---
title: "Understanding Linux LCD Display Timings"
date: 2016-02-02T11:55:43+08:00
tags: ["linux", "embedded", "display", "device-tree", "toradex"]
---

### Mapping LCD/TFT display timings to Linux Kernel data structures

Most LCD/TFT display datasheets provide the following timing information:

- **Horizontal Back Porch (HBP)**: Number of pixel clk pulses between HSYNC signal and the first valid pixel data.
- **Horizontal Front Porch (HFP)**: Number of pixel clk pulses between the last valid pixel data in the line and the next hsync pulse.
- **Vertical Back Porch (VBP)**: Number of lines (HSYNC pulses) from a VSYNC signal to the first valid line.
- **Vertical Front Porch (VFP)**: Number of lines (HSYNC pulses) between the last valid line of the frame and the next VSYNC pulse.
- **VSYNC pulse width**: Number of HSYNC pulses when a VSYNC signal is active.
- **HSYNC pulse width**: Number of pixel clk pulses when a HSYNC signal is active.
- **Active frame width**: Horizontal resolution.
- **Active frame height**: Vertical resolution.
- **Screen width**: `Active frame width + HBP + HFP`
- **Screen height**: `Active frame height + VBP + VFP`
- **VSYNC polarity**: Value of VSYNC to indicate the start of a new frame (active LOW or HIGH)
- **HSYNC polarity**: Value of HSYNC to indicate the start of a new line (active LOW or HIGH)

Some datasheets provide horizontal and vertical blanking period instead:

```
Horizontal Blanking Period = HSYNC + HFP + HBP
Vertical Blanking Period   = VSYNC + VFP + VBP
```

e.g. If Vertical blanking period is given as 150, one can split it as:
```
VSYNC: 60, VFP: 45, VBP: 45
```

## Mapping to `fb_videomode`

```c
struct fb_videomode {
    const char *name;   // Descriptive name
    u32 refresh;        // Refresh rate in Hz
    u32 xres;           // Resolution in x (Active frame width)
    u32 yres;           // Resolution in y (Active frame height)
    u32 pixclock;       // Pixel clock in picoseconds
    u32 left_margin;    // Horizontal Back Porch (HBP)
    u32 right_margin;   // Horizontal Front Porch (HFP)
    u32 upper_margin;   // Vertical Back Porch (VBP)
    u32 lower_margin;   // Vertical Front Porch (VFP)
    u32 hsync_len;      // HSYNC pulse width
    u32 vsync_len;      // VSYNC pulse width
    u32 sync;           // Polarity on Data Enable
    u32 vmode;          // Video mode (e.g. FB_VMODE_NONINTERLACED)
    u32 flag;           // Usually 0
};
```

Note: `pixclock` is in picoseconds — convert MHz to ps: `pixclock_ps = 1e12 / freq_MHz`

![LCD Timing Reference](https://github.com/bhuvanchandra/images-repo/raw/master/images-linux-display-timings/101353-linux_lcd_timing.jpg)
