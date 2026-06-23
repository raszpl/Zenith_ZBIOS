# Zenith Data Systems (ZDS) Z-386 ZBIOS reverse engineering
Zenith build its own bespoke Bios firmware. In this repository I try to investigate and reverse engineer its idiosyncrasies.

ZBIOS 3.2C setup screen:
```
╔══════════════════════════════════════════════════════════════════════════════╗
║                  System Hardware Setup/Configuration Program                 ║
╠═══════════════════════════════════════╦══════════════════════════════════════╣
║  Time:  22:30:08    Date: 06/23/1989  ║  Video Display:  Enhanced Graphics   ║
║                                       ║  Video Refresh Rate: 50 Hz   60 Hz   ║
║                 BASE  EXTENDED  EMS   ║  								       ║
║    Main RAM:    640K       0K  -OFF-  ║  Boot Drive:     Hard Disk Drive 0   ║
║  Add-On RAM: 	    0K       0K   ---   ║  Floppy Drive 0:       3 1/2" 1.1M   ║
║       Total: 	  640K       0K   ---   ║  Floppy Drive 1:       Not Present   ║
║                                       ║                                      ║
║  Operating Speed:  Slow  Fast  Smart  ║  Hard Disk Drive 0:  Drive Type 44   ║
║  Cache Control:      Cache: ON Q: 16  ║  ┌────────────────────────────────┐  ║
║                                       ║  │Cylinders:   976  Heads:       5│  ║
║  Serial Port 1 (COM1):        Enable  ║  │Ship Zone:   976  Sectors:    17│  ║
║  Serial Port 2 (COM2):        Enable  ║  │Precomp:     Off  Capacity:  42M│  ║
║  Parallel Port Assignment:     LPT1:  ║  └────────────────────────────────┘  ║
║                                       ║                                      ║
║  Password Control:   Make No Changes  ║  Hard Disk Drive 1:  -Not Present-   ║
║  ┌─────────────────────────────────┐  ║  ┌────────────────────────────────┐  ║
║  │Current Password:        XXXXXXXX│  ║  │Cylinders:        Heads:        │  ║
║  │New Password:   XXXXXXXX XXXXXXXX│  ║  │Ship Zone:        Sectors:      │  ║
║  │Password Mode:  Prompt   Noprompt│  ║  │Precomp:          Capacity:     │  ║
║  └─────────────────────────────────┘  ║  └────────────────────────────────┘  ║
║               Enter Current Time As HH:MM:SS in 24 hour Format               ║
╚════Use Space/Backspace to select values, Arrows to move, Esc when finished═══╝
```
Quirks:
- Cant enter custom MFM/RLL/IDE HDD parameters, only drives on BIOS predefined list of 62 types are supported.
- Special provision for Zenith MFM controller, bios calls it TEMPEST, by checking for presence of 'DZ' signature at 0C800:1Bh standard window for MFM bios. I havent identified which card this was meant for yet.
- ESDI drive Type (hardcoded as 1 and 100) supports some form of capacity auto detection by executing IDE Command E2_STANDBY with Heads:15 / Sectors:63 and returning Cylinder number. ESDI option uses those hardcoded Heads:15 / Sectors:63 parameters and only changes number of Cylinders. This could only work with Controller performing CHS translation in hardware on the fly, and afaik normal ones didnt?
- There Is some support for custom HDD parameters after all hiding behind custom int13h AH=1d AL=0 and AL=1 functions. Both set Disk Type to 101 and build FDPT on a live system after issuing either IDE_CMD_E1_IDLE_IMMEDIATE or IDE_CMD_E2_STANDBY command.
- Hidden partial support for setting 50 column mode gated behind detection flag (manual mentions physical EGA 200/350 line switch) for mentioned in manual Z-549 or maybe Headland HT208 based [Zenith 3737V1](https://theretroweb.com/expansioncards/s/zenith-data-systems-240-7940) VGA card and non standard Int 10h AX=6502 (int10h_65_al_2_init_50_line_mode) call.
- Password managed by Keyboard Controller, called System Control Processor/SCP by Zenith, and stored in 1Kbit serial EEPROM 93C46.
- Manual: "Do not use the SHIFT or CAPS LOCK keys when entering a password. The computer differentiates between upper- and lowercase characters, making it difficult to remember the correct password." :-)
- Despite Setup fitting on single page ZBIOS has full support for multi page layouts.

# ZBIOS 'MFM-300 Monitor'
Multi-Function Monitor (MFM) was a build-in SoftIce/Periscope like Debugger/Monitor Bios extension user could enter with a press of a button (CTRL-ALT-ENTER) _at any time_.
> CTRL-ALT-ENTER displays the contents of the CPU's registers and flags, and then enters the Monitor program. Press G and ENTER to return to the operating system or your software program.

Description in [Zenith Z386SX (3557V4) Owners Manual](https://archive.org/details/zenith-z-386-sx-owners-manual) page 88, [Zenith Z-386SX/20 (3797V1) Owners Manual](https://github.com/raszpl/Zenith_ZBIOS/blob/main/documentation/Zenith%20Z-386SX-20%20Owners%20Manual%20595-5036.pdf) page 106. REMark _'The Official Zenith/Heath Computer User Magazine'_ [volume10-issue10-1989](https://github.com/raszpl/Zenith_ZBIOS/blob/main/documentation/remark-volume10-issue10-1989.pdf) page 11 and [volume10-issue12-1989](https://github.com/raszpl/Zenith_ZBIOS/blob/main/documentation/remark-volume10-issue12-1989.pdf) page 25.


MFM Monitor menu:
```
				- MFM-300 Command Summary -

CMD:	Explanation				Syntax
----	-----------				------
?:		Help					?
B:		Boot from disk			B [{F|W}][{0|1|2|3}][:<partition>]
C:		Color bar				C
D:		Display memory			D [<range>]
E:		Examine memory			E <addr>
F:		Fill memory				F <range>,{<byte>|"<string>"}...
G:		Execute (Go)			G [=<addr>][,<breakpoint>]...
H:		Hex math				H <number1>,<number2>
I:		Input from port			I <port>
M:		Move memory block		M <range>,<dest>
O:		Output to port			O <port>,<value>
R:		Examine Registers		R [<register>]
S:		Search memory			S <range>,{<byte>|"<string>"}...
T:		Trace program			T [<count>]
U:		Unassemble program		U [<range>]
V:		Set Video/Scroll		V [M<mode>][S<scroll>]
		Where <range> is:		<addr>{,<addr>|L<length>}
TEST:	Extended diagnostics	TEST
SETUP:	Define hardware Setup	SETUP
```
TEST menu:
```
			CHOOSE ONE OF THE FOLLOWING:
			1. DISK READ TEST
			2. KEYBOARD TEST
			3. BASE MEMORY TEST
			4. EXTENDED MEMORY TEST
			5. POWER-UP TEST
			6. EXIT
			ENTER YOUR CHOICE:
```
MFM Monitor shipped in models from three generations of Zenith computers.

386:
- [Zenith Data Systems (ZDS) 3797V1](https://theretroweb.com/motherboards/s/zenith-data-systems-3797v1)
- [Zenith Data Systems (ZDS) 3557V4](https://theretroweb.com/motherboards/s/zenith-data-systems-3557v4) both use [MFM-300 Monitor, Version 2.9B](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-386%20MFM-300%20Monitor,%20Version%202.9B.bin) [MFM-300 Monitor, Version 3.2C](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-386%20MFM-300%20Monitor,%20Version%203.2C.bin) [MFM-300 Monitor, Version 3.6D](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-386%20MFM-300%20Monitor,%20Version%203.6D.bin)

286:
- [Zenith Data Systems (ZDS) 85-3261-01](https://theretroweb.com/motherboards/s/zenith-85-3261-01) [MFM-200 Monitor, 2.0F](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-286%20MFM-200%20Monitor,%20Version%202.0F.bin)
- [Zenith Data Systems (ZDS) Z-248/12](https://theretroweb.com/motherboards/s/zenith-data-syst-z-248-12) [MFM-200 Monitor, Version 2.2](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-248%20MFM-200%20Monitor,%20Version%202.2.bin)

8088:
- [Zenith Data Systems (ZDS) Z-159](https://theretroweb.com/motherboards/s/zenith-data-syst-z-159) [MFM-1200 Monitor, Version 2.9](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-159%20MFM-1200%20Monitor,%20Version%202.9.bin)
- Live [Zenith Z-150](https://www.pcjs.org/machines/pcx86/zenith/z150/cga/) running [MFM-150 Monitor v3.1E](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-150%20MFM-150%20Monitor,%20Version%203.1E.bin) in the browser.

Despite the number 300 in the name suggesting 32bit 386 compatibility MFM-300 supports only original 16bit 8086/8088 instruction set (no 80186 60h PUSHA etc).

# Z-386 ZBIOS 3.2C POST CODE list
[POST code list](https://github.com/raszpl/Zenith_ZBIOS/blob/main/POST%20codes.txt)

# Z-386 ZBIOS 3.2C source code
Dissasemble progress 100%, annotation at about 80% still with lot of unknowns and surely some errors. [Listing](https://github.com/raszpl/Zenith_ZBIOS/blob/main/Zenith%20Z-386%20MFM-300%20Monitor%2C%20Version%203.2C.lst), 
[IDA C style Header file](https://github.com/raszpl/Zenith_ZBIOS/blob/main/Zenith%20Z-386%20MFM-300%20Monitor%2C%20Version%203.2C.h). [IDA 9.3 Free i64 database](https://github.com/raszpl/Zenith_ZBIOS/raw/main/Zenith%20Z-386%20MFM-300%20Monitor,%20Version%203.2C.i64).
