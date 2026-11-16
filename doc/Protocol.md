This document documents the protocol/registers used by the EZFlashJr cart.

# Word of warning

There is a good chance this document will contain mistakes, and a lot will need extra verfication/tests to understand the exact workings.

Most of these functions only work in kernel mode, but a normal ROM can switch back to it. See [Access from a normal ROM](#access-from-a-normal-rom).

# General

General communication with the flash cart is done in 2 ways:

- Register writes
- Mapping to SRAM area

## Writing to registers

The registers are all mapped in the $7Fxx memory area. And usage need to unlocked with an unlock sequence.

Current stage1&kernel always unlock and lock after each usage, if this is required is unknown, and it might just be possible to unlock once and keep it unlocked.

Unlock sequence is:

1. write $E1 to $7F00
2. write $E2 to $7F10
3. write $E3 to $7F20

Lock sequence is:

1. write $E4 to $7FF0

Note:
- Writing anything in the $7F00-$7F2F range while the registers are unlocked will cause the registers to be locked. (This also causes the registers to be locked if the unlock sequence is done while the registers are unlocked)

## SRAM area

Different things can be mapped to the SRAM area ($A000-$C000) including:

- Nothing (default at cart start)
- SRAM
- SD card sector + status
- RTC registers
- ROM load command data

It is currently not known what happens if you map multiple at the same time, might generate a bus conflict on hardware level.

# Registers

Registers are write only. Reading information back is always done in SRAM mapping.

## SD Card related

### $7F30: SD Card window enables

Two bits that control what the SRAM area shows while $7FC0 is `$00`. Use the $7FBx registers to configure which SD Sector to read.

- Bit 0 makes the SD sector buffer readable at $A000+. Up to 4 sectors ($800 bytes) are available, and it wraps after that. This bit also has to be set for the "read done" flag to be set, which is why `stage1` and the `kernel` write `$01` here before starting a read.
- Bit 1 makes the SRAM area read `$01` instead of the sector data once the read is done.

So the observed values are:

- `$00` Unmap the SD Card from SRAM area
- `$01` Map SD Card sector data to SRAM.
- `$03` Map SD Card read status to SRAM. Any SRAM area byte reads `$E1` from the $7FB4 write until the data is in the buffer, then `$01`. (With `$01` the SRAM area also reads `$E1` while busy, then the sector data.)

### $7FB0, $7FB1, $7FB2, $7FB3: SD Card Sector number

These 4 registers combine to a 32bit sector number to read from the SD card. SD sectors are always read 512 bytes at a time.
$7FB0 is the LSB and $7FB3 is the MSB.

### $7FB4: SD Card read/write count + trigger.

This triggers an amount of sectors to read/write. Setting the highest bit triggers a write, else it will read. A maximum of 4 sectors can be read a time. Writing multiple sectors has not been tested yet.

Example:
- `$01` read 1 sector
- `$02` read 2 sectors
- `$81` write 1 sector


## SRAM/RTC related

### $7FC0: Switch SRAM

This register is used to map different things into the SRAM area.

Only the low 4 bits are used. Values:

- `$00` The SD sector buffer and status, as set by [$7F30](#7f30-sd-card-window-enables). With $7F30 at `$00` nothing from the SD card is mapped.
- `$01` The `rom area`. Not used by `stage1` or the `kernel`.
- `$02` The ROM Load command data, while $7F36 bit 0 is set. See [ROM Load data](#rom-load-data). Used by `stage1` to load the `kernel` and by the `kernel` to load a rom.
- `$03` Maps SRAM into SRAM area. Contains multiple pages controlled by $4000. See: [SRAM](#sram) for more details
- `$04` Maps the FW Version to SRAM area. Any byte in the SRAM area contains the version: `$04` on FW4, `$05` on FW5. The number is not stored anywhere, it is wired into the FPGA design.
- `$05` The firmware updater's busy flag, in bit 0. Used by the updater while it writes the FPGA config flash.
- `$06` and up: Maps RTC registers to SRAM area. See [RTC](#rtc)

Source: the FW4 and FW5 (2021-09-18) FPGA bitstreams, decoded into Verilog and proved equivalent. See [sd_mem_path.v](https://github.com/lukesau/ezgb/blob/main/stage0/fw4/rtl/sd_mem_path.v) and [gb_data_out.v](https://github.com/lukesau/ezgb/blob/main/stage0/fw4/rtl/gb_data_out.v) in the ezgb repo.

### $7FD0: RTC Update trigger

While having $7FC0 mapped to $06 (RTC) write new RTC values to the SRAM area, and write $01 to $7FD0 to update the RTC date/time.

## ROM loading related

### $7F37: Configure MBC

This register configures the MBC to be used by the cart. Currently assume this is only prepared, and not applied, as the kernel ROM is still running from ROM when this is written.

Known values:

- `$00` No MBC
- `$01` MBC1
- `$02` MBC2
- `$03` MBC3 (and gameboy camera)
- `$83` MBC3+TIMER
- `$04` MBC5
- `$05` MBC1m
- `$06` Fallback MBC, used for all others. (Seems to act like no MBC)


### $7FC1, $7FC2: Configure ROM size/mask

$7FC1 and $7FC2 combine into a single 16bit mask value. $7FC1 is lower 8 bits. This configures the amount of available ROM space to address from the MBC.

For example $0003 is written if there are 4 ROM pages available.

### $7FC3: Header checksum?

For some magical reason the checksum of the target ROM header is written here. Reason is unknown.

### $7FC4: SRAM Size/mask

$7FC4 follows the pattern of $7FC1/$7FC2, but configures the mask for SRAM access. $00 is written if there is no SRAM available, else it sets up the amount of SRAM banks.

### $7FE0: Reset to loaded ROM

This is written to $80 to reset the cart and load the new rom. This is done from WRAM by the `kernel`.

### $7F36: ROM Loading commands

This maps ROM loading information to the SRAM area. This is used from WRAM as ROM will be changed.

Observed values:

- `$00` Unmap SRAM area
- `$01` Map `ROM Load command` data into SRAM area. This area is write only. See [ROM Load data](#rom-load-data)
- `$03` Map ROM Load status into SRAM area. $A000 reads as $02 when rom loading is done.

### $7FD2: Map firmware update status

This is written to $01 to map firmware update status to SRAM area, the current firmware update command is still busy when address $A000 reads any non zero value.
This is written to $00 to unmap firmware update status.

### $7FD3, $7FD4: Fast load key (FW5 only)

A write to $7FD4 stores the sum of the byte written and the last byte written to $7FD3. When a ROM load starts, the FW5 loader reads that sum, and if it is `$11` it skips the long delay it normally waits after each SD read. Both start at 0. FW4 has neither register and always waits.

`stage1` writes the console's boot value of the A register to $7FD3: `$01` on a DMG or SGB, `$11` on a CGB or GBA, `$FF` on a GB Pocket or SGB2. The `kernel` (K1.05 and later) writes `$00` to $7FD4 before it loads a rom. So with the stock firmware only a CGB or GBA gets the fast load, which matches the K1.05RC changelog line "All device beyond CGB(included) have turbo loading speed". K1.04e never writes $7FD4, so it always gets the slow load, even on FW5. A kernel that writes `$00` to $7FD3 and then `$11` to $7FD4 gets the fast load on every console.

Each skipped delay is about 20 ms, roughly 0.7 s per MB of ROM. This comes from simulation and has not been timed on hardware.

Source: the FW5 (2021-09-18) FPGA bitstream, decoded into Verilog and proved equivalent. See [fw5_7fxx.v](https://github.com/lukesau/ezgb/blob/main/stage0/fw5-0918/fw5_7fxx.v) in the ezgb repo, and the fast-load notes in [firmware-diff.md](https://github.com/lukesau/ezgb/blob/main/stage0/docs/firmware-diff.md#the-fast-load-key).

### $7F32: Mode

Bits 7 and 3 of this register switch the whole cart between three modes. Both bits are set at power-on.

| Bit 7 | Bit 3 | Mode | ROM area ($0000-$7FFF) reads | Set by |
| ----- | ----- | ---- | ---------------------------- | ------ |
| 1 | 1 | stage1 | `stage1`, from the FPGA block RAM | power-on |
| 1 | 0 | kernel | the `rom area`, where the kernel was loaded | `stage1`, `$80` |
| 0 | 0 | game | the `rom area`, through the MBC set in $7F37 | `kernel` launch, `$00` |

In kernel mode:

- Writes to $2000-$2FFF set an 8 bit ROM bank and $3000-$3FFF its bit 8. There is no MBC behavior and the bank masks are not applied.
- $4000-$5FFF sets the SRAM page used by `$7FC0 = $03`.
- The SRAM area shows whatever $7FC0 selects.
- The SD card and the PicoBlaze processor that drives it (and the RTC) are running.

In game mode, writes to $0000-$7FFF go to the MBC set in $7F37 and the SRAM area is the normal cart SRAM. The SD card logic is stopped, and the PicoBlaze is stopped too unless $7F37 bit 7 (the RTC flag) is set. The SD sector buffer and status can't be seen in the SRAM area in this mode.

### $7F31: Nothing

There is no register at $7F31. A write to it keeps the registers unlocked but is stored nowhere. `stage1`, the `kernel` and the updater write `$00` to it next to every $7F32 write. Why is not known.

# Access from a normal ROM

The unlock sequence and the $7Fxx registers work in every mode. So a normal ROM can switch the cart back to kernel mode:

1. Unlock.
2. `$80` -> `$7F32`
3. Lock.

The ROM keeps running from the same `rom area` data, but the banking is now the kernel mode banking above. SD access, the RTC registers and all SRAM pages ($7FC0) then work as they do in the `kernel`. Switch back to game mode by writing `$00` to $7F32. Because the ROM bank register is shared between the two modes, write the game's ROM bank again after switching back. The MBC type, the masks and the SRAM enable are kept.

Notes:

- Don't write `$88` to $7F32 unless you want `stage1` back in the ROM area.
- The unlock and lock writes go to $7F00-$7FF0, which is also the $6000-$7FFF MBC register range in game mode. In an MBC1 game they change the banking mode bit and leave it at 0, so a game that uses mode 1 has to set it again.
- This is the sequence [VGM_player](https://github.com/untoxa/VGM_player/) uses to read the SD card (see issue #7).

Source: the FW4 and FW5 (2021-09-18) FPGA bitstreams, decoded into Verilog and proved equivalent. See [gb_regs.v](https://github.com/lukesau/ezgb/blob/main/stage0/fw4/rtl/gb_regs.v) and [unlock.v](https://github.com/lukesau/ezgb/blob/main/stage0/fw4/rtl/unlock.v) in the ezgb repo, and [gb_regs.v](https://github.com/lukesau/ezgb/blob/main/stage0/fw5-0918/gb_regs.v) for FW5. Switching back to game mode has not been tested on hardware.

# SRAM

There are 64 banks of SRAM available. The first 16 are the normal cart SRAM that you can access during a normal rom. Rest of the SRAM is mostly unused except for a few bits the ezgb.dat kernel puts in there.
Unused SRAM contains random garbage, but can be used normally otherwise.

First pages seem to be normal cart SRAM, and mapped to the SRAM area during normal cartidge, as well as backed up to save files.

- Page $11 seems to contain flash cart status (previously loaded rom for SRAM backup?)
- Page $12 and beyond is used as extra RAM during the kernel, mostly to cache the file list.

- `$11:$A000` is `$AA` if there is SRAM data to be backed up.
- `$11:$A001` is size of the SRAM to bankup in number of SRAM banks.
- `$11:$A00F` length of the SRAM save file filename
- `$11:$A010+` contains sram save file if save needs to be backed up (in wchar_t)
- `$11:$A200` auto save flag. $00 = no auto save, $01 = auto save
- `$11:$A201` is written to `$88` to indicate cart initialization, if this isn't read back on boot of the kernel it reports "Battery dry"
- `$11:$A300+` contains the last loaded rom filename (in wchar_t)

# RTC

The RTC maps a full date/time into SRAM. Unlike the MBC3 RTC it contains full date and is BCD encoded. Data is not latched, values keep updating.

Known mapped data is: (x is any value, mapping repeats every `$0100` bytes)

- `$Ax08`: BCD encoded seconds
- `$Ax09`: BCD encoded minutes
- `$Ax0A`: BCD encoded hours (24H format)
- `$Ax0B`: BCD encoded day of the month
- `$Ax0C`: Day of the week
- `$Ax0D`: BCD encoded month number
- `$Ax0E`: BCD encoded last 2 digits of the year
- All other addresses are zero (or open bus?)

# ROM Load Data

To load a rom 512 bytes of data is loaded into the `ROM Load command` data. These are 128 32bit integers.
This data is used to load the ROM from the SD card. It contains a list of SD card sectors to read into the ROM. It is a list of start sector with amount of sectors to read. Depending on the fragmentation on the SD card it will only need one or a few entries.

(Note, when observing this in the `kernel`, the used buffer isn't cleared and unwritten data still contain the first 512 bytes of the loaded ROM)

General structure is:

- `INT32[0]`: Value 0, reason unknown.
- `INT32[1..X]`: Pairs of 2 integers. First the start sector to start reading from, and next the amount of sectors to read. When there is no more fragmentation the last size is $FFFFFFFF
- `INT32[$7C]`: Size of the to be loaded ROM in bytes.
- `INT32[$7D]`: Value 1, reason unknown.
- `INT32[$7E]`: Value 1,4,8,16, reason unknown (maybe amount of sectors in a FAT cluster?)
