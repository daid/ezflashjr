This document notes the loading sequence the default `stage1` and `kernel` use. This is done in great detail.

See General.md for basic info on how things are loaded.

See Protocol.md for detailed information about all the used registers.


Information about lock/unlock is left out. Generally each register write is preceded by an unlock and followed by a lock. Only a few registers are written during a single lock&unlock sequence.

- `stage1` is active.
- `$01` -> `$7fd3`: Reason unknown (only in FW5)
- `$00` -> `$7fc0`: Unmap SRAM (guess: per default SRAM is mapped, or random mapping, and we need to unmap before we can map SD in there)
- Following is repeated multiple times to read the partition information, and FAT tables to find ezgb.dat and which sectors it is stored in. Note that the same sector is repeatedly read multiple times for some reason.
  - `$01` -> `$7f30`: Map SD data (reason unknown, might be required before initiating data read?)
  - Sector number -> `$7fb0` `$7fb1` `$7fb2` `$7fb3`: Prepare to load sector
  - `$01` -> `$7fd4`: Read SD sector from SD card
  - `$03` -> `$7f30`: Map SD read status
  - `$A000` is checked to see if SD data is ready
  - `$01` -> `$7f30`: Map SD sector data, reason unknown, as status is not read...
  - SD Sector data is accessed
  - `$00` -> `$7f30`: Unmap SD status
- `$02` -> `$7fc0`: Reason unknown.
- `$01` -> `$7f36`: Map the ROMLoadInfo
- Fill the ROMLoadInfo with sector data to load
- Start running code from WRAM instead of ROM.
- `$03` -> `$7f36`: Initiate the load of ROM from SD sectors.
- `$00` -> `$7f36`: Unmap ROMLoadInfo
- Set ROM bank number to $0001 by writting $2000 and $3000 (MBC5 style)
- `$00` -> `$7f31`: Reason unknown
- `$80` -> `$7f32`: Reason unknown
- Stop running from WRAM and jump to $0100 to start the `kernel`. Kernel is now loaded and running.
- `$03` -> `$7fc0`: Enable SRAM.
- Read data from SRAM to see if save data needs to be backed up. (assuming no data to backup)
- `$00` -> `$7fc0`: Disable SRAM.
- Read the SD card, same way as the `stage1` process, except that some data is store to SRAM for caching reasons.
- User selects a file in the menu.
- `$xx` -> `$7f37`: to select the right MBC for the selected rom
- `$00` -> `$7fd4`: Reason unknown (as of kernel version 1.05)
- `$xx` -> `$7fc4`: Set SRAM bank mask (as of kernel version 1.01)
- `$xx` -> `$7fc1`: Set ROM bank mask low byte
- `$xx` -> `$7fc2`: Set ROM bank mask high byte
- `$xx` -> `$7fc3`: Write the header checksum for some reason...
- `$01` -> `$7f36`: Map the ROMLoadInfo
- Fill the ROMLoadInfo with sector data to load.
- Start running code from WRAM instead of ROM.
- `$03` -> `$7f36`: Initiate the load of ROM from SD sectors.
- `$00` -> `$7f36`: Unmap ROMLoadInfo
- `$00` -> `$7f31`: Reason unknown
- `$00` -> `$7f32`: Reason unknown
- `$80` -> `$7fe0`: Trigger a reset to start the selected from fresh.
