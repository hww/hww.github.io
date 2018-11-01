# Aleste 520EX

The Aleste 520EX computer, developed by me at 1993, is a backward compatible with Amstrad CPC 6128 and runs MSX DOS operation system.

In addition to the standard CPC6128 features, it includes a number of extra features: 64-color palette, double color depth in slightly decreased horizontal resolution mode (eg. 16 color at 256x200 resolution), battery-backed Real Time Clock chip, 512Kbyte RAM (of which 192K can be accessed as on 64K CPC with 128K dk'tronics memory expansion), 8bit printer port, expanded keyboard matrix with 10 additional keys, two software controlled LEDs, extended Expansion Port (with additional pins for DMA support).

![Aleste 520EX](/projects/aleste/aleste_520ex_512px.jpg)

## External References

[Aleste 520EX - CPCWiki](http://www.cpcwiki.eu/index.php/Aleste_520EX)

## Simulation of TMS9928 

Was implemented tiny TMS9938 simulator and after that original MSX games could be pached to run on Aleste. The patching process require several things:

- Some of portions of code patched by replacing by multiple `RET` 
- Replace `LD	(0A000H),A` to `CALL	0F56CH`
- Replace `OTIR` by `RST10`
- Fuctions with OUT to the port `LD	A,(DE); INC	DE; OUT	(C),A` replaced to `RST16`

Video below shows running on Aleste the _King Valley II_ game.

[Aleste 520EX - TMS9928 Simulation](https://www.youtube.com/watch?v=m0mpAffz5DQ)

## Screenshots

![Aleste 520EX - Boot Screen](/projects/aleste/aleste_boot_screen.png)

![Aleste 520EX - MSX DOS Screen](/projects/aleste/msx_dos_screenshot.png)

![Aleste 520EX - Test Screen](/projects/aleste/test_screenshot.png)

