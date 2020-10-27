This document documents the protocol/registers used by the EZFlashJr cart.

# Word of warning

There is a good chance this document will contain mistakes, and a lot will need extra verfication/tests to understand the exact workings.

It is unknown if/which of these functions are available during a normal ROM run. Effort has focused on the `loader` so far.

# General

General communication with the flash cart is done in 2 ways:

- Register writes
- Mapping to SRAM area

## Writting to registers

The registers are all mapped in the $7Fxx memory area. And usage need to unlocked with an unlock sequence.

Current stage1&loader always unlock and lock after each usage, if this is required is unknown, and it might just be possible to unlock once and keep it unlocked. But could also be that locking also triggers some execution.

Unlock sequence is:

1. write $E1 to $7F00
2. write $E2 to $7F10
3. write $E3 to $7F20

Lock sequence is:

1. write $E4 to $7FF0

## SRAM area

Different things can be mapped to the SRAM area ($A000-$C000) including:

- SRAM
- SD card sector + status
- RTC registers
- ROM load command data

It is currently not known what happens if you map multiple at the same time, might generate a bus conflict on hardware level.

# Registers

Registers are write only. Reading information back is always done in SRAM mapping.

## SD Card related

### $7F30: Switch SD Card SRAM

Writting to this register will configure the SRAM area as different things.
Use the $7FBx registers to configure which SD Sector to read.

Observed possible values:

- 0) Unmap the SD Card from SRAM
- 1) Map SD Card sector data to SRAM, data will be available at $A000-$A200 (unknown if this wraps)
- 3) Map SD Card read status to SRAM, status is available at $A000, $01 indicates read is done. (usage seems to be to wait till reading is done)

### $7FB0, $7FB1, $7FB2, $7FB3: SD Card Sector number

These 4 registers combine to a 32bit sector number to read from the SD card. SD sectors are always read 512 bytes at a time.
$7FB0 is the LSB and $7FB3 is the MSB.

### $7FB4: SD Card trigger?

This seems to be written to $01 after a sector number is loaded. This could be a trigger to start, or an amount of sectors to load. Testing is required to be sure.


## SRAM/RTC related

### $7FC0: Switch SRAM

This register is used to map different things into the SRAM area.

Observed values:

- 0) Unmap SRAM area
- 2) Unknown. Used during `stage1`, but SRAM is not accessed afterwards.
- 3) Maps some kind of SRAM into SRAM area. Contains multiple pages controlled by $4000, but more pages are available. See: (SRAM)[#sram] for more details
- 6) Maps RTC registers to SRAM area. See (RTC)[#rtc]


# SRAM

TODO... this is most likely loader specific, not Cart specific. And just SRAM for "general usage". Lower pages might actually just be normal SRAM? Just lots to still figure out.

Page $11 seems to contain flash cart status (previously loaded rom for SRAM backup?)
Page $12 is used as extra RAM during the loader.

`$11:$A000` is `$AA` if there is SRAM data to be backed up. Rest of workings not yet known.
`$11:$A201` is written to `$88` to indicate cart initialization, if this isn't read back on boot of the loader it reports "Battery dry"

# RTC

The RTC maps a full date/time into SRAM. Unlike the MBC3 RTC it contains full date and is BCD encoded.

Known mapped data is:

`$A008`: BCD encoded seconds
`$A009`: BCD encoded minutes
`$A00A`: BCD encoded hours (24H format)
`$A00B`: BCD encoded day of the month
`$A00C`: ??
`$A00D`: BCD encoded month number
`$A00E`: BCD encoded last 2 digits of the year
