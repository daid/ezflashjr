Here the official firmware releases are archived.

These contain the files included in the original zip file:

- Update_*.gb: Update rom which updates the FPGA/Boot rom.
- ezgb.dat: The "kernel" ROM which contains the file selection menu etc...
- Some versions contain readme and changelog files in English and/or Chinese.
- bitstream-dump.bin: The original release (2019-12-25_K1.00) did not come with a update ROM containing the FPGA bitstream. This file is instead a dump of the flash chip from a cartridge known to contain firmware version 1 so the bitstream can be preserved.

Both the updater and kernel are GB ROMs. Both are built with sdcc and gbdk. Used versions are unknown.


Update_*.gb seems to be a 32KB rom with the payload data attached after that.
Guess is that the payload is an FPGA bitstream, but this has not been confirmed.
What has been confirmed is that this update changes the `stage1` boot.


## Official changelog

### FW5 K1.05RC

Released 2020-07-31.

* Supported CGB without CPU suffix("Micro SD Initial Error!" issue)
* RTC codes are rewritten, now working properly with all official RTC games
* All device beyond CGB(included) have turbo loading speed

### FW4 K1.04e

Released 2020-03-10.

* Removed Chinese support and filename sorting
* Fixed booting sequence halt in OS INIT which caused by disk fragmentation
* Fixed random File System Error
* Added a battery dry notice
* Some interface rearrangement
* Some scroll timing tweak
* The total length of the filename is 254 characters
* The maximum number of files in the folder is 7000

### FW3 K1.03

Released 2020-01-11.

* Improved game compatibility
* Added the support of MBC1M Multicart

### K1.02

Released 2020-01-07.

* Fixed some logical error in last played game.

### FW2 K1.01

Released 2020-01-04.

* Fixed some game compatibility
* Optimize the limited number of files and directories
* Fixed some file display issues
* Added a Reading interface
* Added a Loading interface
* Added the support of MBC30 64KB SAVE
* Added AUTO SAVE in SET tab<br>Checked = Backup the save to SD every time automatically when kernel booting up <br>Unchecked = Kernel will ask every time when kernel booting up, cancel backup may cause you to lose the last play record
* Added last played game Press START in file browser to activate, Press A to launch, B to cancel.

### K1.00

Released 2019-12-25.

The very first release.