Here the official firmware releases are archived.

These contain 2 files:

- Update_*.gb: Update rom which updates the FPGA/Boot rom.
- ezgb.dat: The loader rom which contains the file selection menu etc...


Both are roms, both are build with sdcc and gbdk. Used versions are unknown.


Update_*.gb seems to be a 32KB rom with the payload data attached after that.
Guess is that the payload is an FPGA bitstream, but this has not been confirmed.
What has been confirmed is that this update changes the `stage1` boot.
