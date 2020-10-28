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

- `$00` Unmap the SD Card from SRAM area
- `$01` Map SD Card sector data to SRAM, data will be available at $A000-$A200 (unknown if this wraps)
- `$03` Map SD Card read status to SRAM, status is available at $A000, $01 indicates read is done. (usage seems to be to wait till reading is done)

### $7FB0, $7FB1, $7FB2, $7FB3: SD Card Sector number

These 4 registers combine to a 32bit sector number to read from the SD card. SD sectors are always read 512 bytes at a time.
$7FB0 is the LSB and $7FB3 is the MSB.

### $7FB4: SD Card trigger?

This seems to be written to $01 after a sector number is loaded. This could be a trigger to start, or an amount of sectors to load. Testing is required to be sure.

This is written to $81 to write a sector to the SD card. Data is stored in the SD Cart sector data mapping before this is initiated.


## SRAM/RTC related

### $7FC0: Switch SRAM

This register is used to map different things into the SRAM area.

Observed values:

- `$00` Unmap SRAM area
- `$02` Unknown. Used during `stage1`, but SRAM is not accessed afterwards.
- `$03` Maps SRAM into SRAM area. Contains multiple pages controlled by $4000, but more pages are available. See: [SRAM](#sram) for more details
- `$04` Maps the FW Version to SRAM area. `$A000` contains version
- `$05` Related to the firmware update. Firmware update data is stored in SD to RAM area, exact process unknown.
- `$06` Maps RTC registers to SRAM area. See [RTC](#rtc)

## ROM loading related

### $7F37: Configure MBC

This register configures the MBC to be used by the cart. Currently assume this is only prepared, and not applied, as the loader ROM is still running from ROM when this is written.

TODO: Note which values are which MBC.

### $7FC1, $7FC2: Configure ROM size/mask

$7FC1 and $7FC2 combine into a single 16bit mask value. $7FC1 is lower 8 bits. This configures the amount of available ROM space to address from the MBC.

For example $0003 is written if there are 4 ROM pages available.

### $7FC3: Header checksum?

For some magical reason the checksum of the target ROM header is written here. Reason is unknown.

### $7FC4: SRAM Size/mask

$7FC4 follows the pattern of $7FC1/$7FC2, but configures the mask for SRAM access. $00 is written if there is no SRAM available, else it sets up the amount of SRAM banks.

### $7FE0: Reset to loader ROM

This is written to $80 to reset the cart and load the new rom. This is done from WRAM by the `loader`.

### $7F36: ROM Loading commands

This maps ROM loading information to the SRAM area. This is used from WRAM as ROM will be changed.

Observed values:

- `$00` Unmap SRAM area
- `$01` Map `ROM Load command` data into SRAM area. See [ROM Load data](#rom-load-data)
- `$03` Map ROM Load status into SRAM area. $A000 reads as $02 when rom loading is done.

### $7FD2: Unknown

This is written to $00 and $01 during firmware update. $A000 is read after $01 is written. Potentially write status info on firmware update?

### $7FD4: Unknown

This is written to $00 during preperation to load a rom. Reason unknown.

### $7F31, $7F32: Unknown

These are written to $00 after a rom is loaded before a reset to this new rom. Reason unknown.

These are written to $00 and $80 by the `stage1` after the `loader` is loader. Reason unknown.

These are written to $00 and $80 by the firmware update. Reason unknown.

# SRAM

TODO... this is most likely loader specific, not Cart specific. And just SRAM for "general usage". Lower pages might actually just be normal SRAM? Just lots to still figure out.

First pages seem to be normal cart SRAM, and mapped to the SRAM area during normal cartidge, as well as backed up to save files.

- Page $11 seems to contain flash cart status (previously loaded rom for SRAM backup?)
- Page $12 is used as extra RAM during the loader.

- `$11:$A000` is `$AA` if there is SRAM data to be backed up.
- `$11:$A001` is size of the SRAM to bankup in number of SRAM banks.
- `$11:$A00F` length of the SRAM save file filename
- `$11:$A010+` contains sram save file if save needs to be backed up (in wchar_t)
- `$11:$A201` is written to `$88` to indicate cart initialization, if this isn't read back on boot of the loader it reports "Battery dry"
- `$11:$A300+` contains the last loaded rom filename (in wchar_t)

# RTC

The RTC maps a full date/time into SRAM. Unlike the MBC3 RTC it contains full date and is BCD encoded.

Known mapped data is:

- `$A008`: BCD encoded seconds
- `$A009`: BCD encoded minutes
- `$A00A`: BCD encoded hours (24H format)
- `$A00B`: BCD encoded day of the month
- `$A00C`: ??
- `$A00D`: BCD encoded month number
- `$A00E`: BCD encoded last 2 digits of the year

# ROM Load Data

To load a rom 512 bytes of data is loaded into the `ROM Load command` data. These are 128 32bit integers.
This data is used to load the ROM from the SD card. It contains a list of SD card sectors to read into the ROM. It is a list of start sector with amount of sectors to read. Depending on the fragmentation on the SD card it will only need one or a few entries.

(Note, when observing this in the `loader`, the used buffer isn't cleared and unwritten data still contain the first 512 bytes of the loaded ROM)

General structure is:

- `INT32[0]`: Value 0, reason unknown.
- `INT32[1..X]`: Pairs of 2 integers. First the start sector to start reading from, and next the amount of sectors to read. When there is no more fragmentation the last size is $FFFFFFFF
- `INT32[$7C]: Size of the to be loaded ROM in bytes.
- `INT32[$7D]: Value 1, reason unknown.
- `INT32[$7E]: Value 4, reason unknown.
- `INT32[$7F]: Value 0, reason unknown.
