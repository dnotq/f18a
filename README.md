# F18A

  A pin-compatible enhanced replacement for the TMS9918A VDP family.

  https://dnotq.io

---

## Version History

### F18A V1.9 Dec 31, 2018 (CRC: 99/4A-disk: A374, 99/4A-rom: 147A, CV-rom:  )

  Oct 14, 2018 WIP

  * Prepare for open source release.
  * Split up the original "core" to create a top-module for the stand-alone
    F18A, and a "main core" that can be used as part of a larger SoC.
  * Fixed the VGA horizontal timing error caused by treating the pixel time as
    40ns instead of 39.68ns.  Because events were being counted in "pixels",
    this caused the horizontal sync pulse to be slightly off, and the overall
    line time to be 32us instead of 31.746us.  This error meant each line was
    around 6.4 pixels too long, and pushed the total frame rate to 59.2Hz.
    This error was enough to cause games to fail (Pole Position on the 99/4A),
    and some monitors to not sync properly when run through video converters.
    The timing error also caused many problems for the PAL ColecoVision.
  * Removed sprite-linking.  This was an unused feature and helped free up
    FPGA resources to allow the core to better fit in the Spartan-3E 250K.
  * Removed programmable GROMCLK divisor.  Unused feature, free up resources.
  * Register mode and cd_i inputs to CPU component.

  Apr 23, 2017 WIP

  * Working on PAL ColecoVision problem.  Changed status bit to always clear
    on read instead of handling a possible hazard where the end of frame is
    trying to set the flag at the same time it is being read.  It will always
    be cleared.


### F18A V1.8 Aug 24, 2016 (CRC: F981)

  * Fixed sprite collision bug where sprite collisions were being incorrectly
    detected outside of the active display, after line 191 or 239 depending on
    the line mode.
  * Added hybrid VR write restriction to mask VR writes to three-bits when the
    F18A is locked, like the real 9918A does.  However, if mode bit M4 is set
    (80-columns), writes to VRs over VR7 are *ignored* instead of masked to
    three-bits.  This allows various 9938 programs to work (or continue to
    work), as well as continue to support TurboForth that writes to VRs 0..15
    to set up 80-columns (if straight masking was used, VRs 8..15 would over-
    write VR 0..7).


### F18A V1.7 Jan 1, 2016 (CRC: A3B5)

  * Fixed Bitmap-Layer (BML) display bug.
  * Fixed GPU’s PIX instruction to properly calculate BML addresses.
  * Added power-on graphic banner that shows the current firmware version.


### F18A V1.6 May 3, 2014 .. Apr 26, 2015 (CRC: 40CC)

  A very significant release with a lot of changes based on feedback and
  attempts to use the F18A as originally designed.  Some of the original
  features were uninformed, which became painfully obvious when people started
  trying to use some of the features.

  * Removed fixed tile functionality
  * Removed border scroll limit functionality
  * Removed banner functionality
  * Removed host-side 32-bit counter
  * Removed host-side 32-bit RNG
  * Removed GPU 32-bit counter
  * Removed GPU 32-bit RNG
  * Removed the sprite "disable value" (>F8) in the sprite Y-location when
    ROW30 is enabled.
  * Added second tile layer with its own NTBA, CTBA, h/v page sizes, and h/v
    scroll regs
  * Added ECM2/3 pattern table size selections for tiles and sprites.
  * Added host-side segmented counter with 10ns accuracy.
  * Added configurable HSYNC and VSYNC GPU triggers.
  * Added fat-pixel (2x1) with 16-color support to the bitmap layer (BML).
  * Added 1x1 page scroll support for T40 and T80 modes.
  * Added option to reset most VDP registers to their power-on values.
  * Added option to disable Tile Layer 1, which includes GM1, GM2, MCM, T40,
    and T80.  Sprites, the BML, and TL2 are still active and can be enabled or
    disabled independently.
  * Added option to allow attribute byte to be foreground / background color
    select in T40 and T80.
  * Added per-position tile attribute support.
  * Added DMA capability to the GPU:
    - 8xx0 - MSB src
    - 8xx1 - LSB src
    - 8xx2 - MSB dst
    - 8xx3 - LSB dst
    - 8xx4 - width
    - 8xx5 - height
    - 8xx6 - stride
    - 8xx7 - 0..5 | !INC/DEC | !COPY/FILL
    - 8xx8 - trigger
    - FILL (active high) will read a single byte at the src address and fill
      the destination with that byte.
    - src, dst, width, height, and stride are copied to dedicated counters
      when the DMA is triggered, thus the original values remain unchanged.
  * Added USR3 jumper to control GROMCLK/CPUCLK output on pin37 to provide
    support for 9128/29
  * Added USR2 jumper to disable/enable simulated scan lines (every other VGA
    scan line has its color reduced by 50%).  Also controllable via a new VDP
    register bit.
  * Added a 5th sprite reporting option instead of reporting the max-sprite,
    which on the F18A might be different than the original VDP because all 32
    sprites can be on a single scan line.
  * Added a new register  (VR51) to limit the maximum sprite processed.  This
    has nothing to do with the number of sprites that can be visible on a scan
    line, which is controlled by a separate register (VR30).  This register is
    always active and can be used instead of the >D0 byte in the sprite
    Y-location, and is the only way to limit sprite processing early when
    ROW30 is enabled.
  * Changed the GPU interlock so that polling the VDP status register will not
    cause the GPU to pause.  This should greatly increase GPU performance
    during heavy VDP interrupt polling.
  * Fixed T80 NTBA two LSbit problem.  They are ignored (set to "00") when the
    F18A is locked to provide compatibility with the 9938 and avoid problem
    with software that set the two LSbits of the NTBA to other than "11" as
    the 9938 documentation specifies they should be.  This limits the T80 name
    table to 4K boundaries.  When the F18A is unlocked, all 4-bits of the NTBA
    are used and the T80 name table can be located on 1K boundaries.
  * Fixed the 5th number update during a scan line.  As long as the 5S flag is
    zero, the 5th number register follows the sprite scanning sequence.  Seems
    to be a transparent latch that follows the input (current sprite being
    scanned) until latched by the 5S flag.  If the status register is being
    polled and 5S is reset mid frame, then the 5th number begins following the
    scanned sprites again.  This bug is known to have affected Miner49er on
    the 99/4A.

### In-System Updater for the TI-99/4A, Dec 18, 2013

  Announcement and code posted in AtariAge forum.  Many thanks to Tursi and
  Rasmus for making this possible.


### F18A V1.5 July 23, 2013

Not really a *bug* fix since the problem it corrects exists on the real 9918A,
and only has to do with sporadic collision bit reporting during heavy polling
of the original 9918A VDP status register. This was discovered while Rasmus
was writing Titanium. The 9918A was not designed to have its status register
polled (that is why it provides an interrupt output).

I don't think the original 9918A designers took the hazard into consideration,
but I decided to make this correction because it is what the original designers
would have done given their preference (and I asked Karl Guttag about it).
Thus, the F18A implements what could be considered the "expected behavior",
and will work as expected where the original 9918A might not. I did not make
this decision lightly.


### F18A V1.4 Mar 20, 2013, April 26, 2013

  * Fixed the sprite collision bug.  Now only sprites with *on-screen pixels*
    are considered.  The posted XB games are working correctly with the update.
  * Fixed the 40-column text mode problem when the F18A 30-row option was
    enabled.
  * Added more user vectors to the default GPU code.
  * Fixed GPU DIV, small separate update (April) without a revision number
    change.


### F18A V1.3 Release firmware, July 2012

  Initial F18A release.