Here the official firmware releases are archived.

These contain 2 files:

- Update_*.gb: Update rom which updates the FPGA/Boot rom.
- ezgb.dat: The loader rom which contains the file selection menu etc...


Both are roms, both are build with sdcc and gbdk. Used versions are unknown.


Update_*.gb seems to be a 32KB rom with the payload data attached after that.
Guess is that the payload is an FPGA bitstream, but this has not been confirmed.
What has been confirmed is that this update changes the `stage1` boot.


## Official changelog

### FW5 K1.05RC

* Supported CGB without CPU suffix("Micro SD Initial Error!" issue)
* RTC codes are rewritten, now working properly with all official RTC games
* All device beyond CGB(included) have turbo loading speed

### FW4 K1.04e

* Removed Chinese support and filename sorting
* Fixed booting sequence halt in OS INIT which caused by disk fragmentation
* Fixed random File System Error
* Added a battery dry notice
* Some interface rearrangement
* Some scroll timing tweak
* The total length of the filename is 254 characters
* The maximum number of files in the folder is 7000

### FW3 K1.03

* Improved game compatibility
* Added the support of MBC1M Multicart

### K1.02

* Fixed some logical error in last played game.

### FW2 K1.01

* Fixed some game compatibility
* Optimize the limited number of files and directories
* Fixed some file display issues
* Added a Reading interface
* Added a Loading interface
* Added the support of MBC30 64KB SAVE
* Added AUTO SAVE in SET tab<br>Checked = Backup the save to SD every time automatically when kernel booting up <br>Unchecked = Kernel will ask every time when kernel booting up, cancel backup may cause you to lose the last play record
* Added last played game Press START in file browser to activate, Press A to launch, B to cancel.
