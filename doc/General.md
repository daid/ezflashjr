Here the general structure and process of the EZFlashJr flash cart is documented.

# Terminology

| Name   | Description |
| ------ | ----- |
| stage1 | This is a 32KB initial loader that is builtin to the cartridge. Dumped version is available in /boot/ |
| kernel | This is the code that is located on the SD card called ezgb.dat, which is just a rom file |
| rom area | This updatable area inside the cart that contains the current rom |

# Boot process (simplified)

- `stage1` is started from the gameboy bootrom.
- `stage1` loads the `kernel` rom from the SD card into the `rom area` and jumps to $0100 to start it.
- `kernel` shows the menu and allows file selection.
- On a selected file the flash cart is reconfigured for the rom, the rom is loaded from SD card into the `rom area`.
- The gameboy is reset to boot properly to the new rom as if it is started from power-on.

# Notes:

- `stage1` is updatable, the FW Update roms change this rom. Details are not known.
- `kernel` is easy to swap out for a different rom. But a bit of care needs to be taken as it is not started like a normal rom.
- `stage1` and `kernel` always run in DMG mode.
- `stage1` and `kernel` are created with sdcc.
- `stage1` clears WRAM, so WRAM is no longer uninitialized when starting the target rom.
- The SD is accessed raw and the `stage1` and `kernel` read the partition table and FAT32 to get file lists.
- Loading a ROM into the `rom area` is done by special commands while running from WRAM.
