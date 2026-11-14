These are the dumped first stage boot roms of the EZ Flash Jr.
The first stage loads the 2nd stage loader (which provides file selection) from the SD card.

These ROMs are somehow updated by the firmware updater, exact mechanism is currently unknown.

These ROMs are 32KB, build with SDCC (which is obvious if you disassemble them)

## Version history

### FW1

Dumped by nitro2k01. This is the factory loaded version on a cartridge with manufacturing week 4619. It is reported to be version 1 by the update ROM. This version of stage 1 does not have error handling for a missing SD card. It is unknown whether this version is the first version shipped from factory and/or whether it was ever shipped with a firmware update.

### FW2

First shipped with version K1.01 according to the official changelog. Not dumped yet.

### FW3

First shipped with version K1.03 according to the official changelog. Not dumped yet.

### FW4

First shipped with version K1.04e according to the official changelog. According to changelog: "Fixed booting sequence halt in OS INIT which caused by disk fragmentation".

### FW5

First shipped with version K1.05RC according to the official changelog.


### FW5 (2021-09-18)

In `FW5-2021-09-18`. Shipped in the 2021-09-18 updater, and the same ROM is in the forSGB beta updater. It differs from the FW5 one above in three header bytes: the SGB flag, the old licensee code and the header checksum. Not dumped from a cartridge: taken from the block RAM contents of the FPGA bitstream inside the updater. The FW4 and FW5 ROMs above come out of their updaters the same way and match the dumps.
