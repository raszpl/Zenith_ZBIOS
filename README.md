# Zenith Data Systems (ZDS) ZBIOS 'MFM-300 Monitor' reverse engineering.
Multi-Function Monitor (MFM) was a build-in SoftIce/Periscope like Debugger/Monitor Bios extension user could enter with a press of a button (CTRL-ALT-ENTER) _at any time_.
> CTRL-ALT-ENTER displays the contents of the CPU's registers and flags, and then enters the Monitor program. Press G and ENTER to return to the operating system or your software program.

Description in [Zenith Z386SX (3557V4) Owners Manual](https://archive.org/details/zenith-z-386-sx-owners-manual) page 88, [Zenith Z-386SX/20 (3797V1) Owners Manual](https://github.com/raszpl/Zenith_ZBIOS/blob/main/documentation/Zenith%20Z-386SX-20%20Owners%20Manual%20595-5036.pdf) page 106. REMark _'The Official Zenith/Heath Computer User Magazine'_ [volume10-issue10-1989](https://github.com/raszpl/Zenith_ZBIOS/blob/main/documentation/remark-volume10-issue10-1989.pdf) page 11 and [volume10-issue12-1989](https://github.com/raszpl/Zenith_ZBIOS/blob/main/documentation/remark-volume10-issue12-1989.pdf) page 25.


MFM Monitor menu:

			ENTER YOUR CHOICE:

		- MFM-300 Command Summary -
	CMD:	Explanation		Syntax
	----	-----------		------
	?:	Help			?
	B:	Boot from disk		B [{F|W}][{0|1|2|3}][:<partition>]
	C:	Color bar		C
	D:	Display memory		D [<range>]
	E:	Examine memory		E <addr>
	F:	Fill memory		F <range>,{<byte>|"<string>"}...
	G:	Execute (Go)		G [=<addr>][,<breakpoint>]...
	H:	Hex math		H <number1>,<number2>
	I:	Input from port		I <port>
	M:	Move memory block	M <range>,<dest>
	O:	Output to port		O <port>,<value>
	R:	Examine Registers	R [<register>]
	S:	Search memory		S <range>,{<byte>|"<string>"}...
	T:	Trace program		T [<count>]
	U:	Unassemble program	U [<range>]
	V:	Set Video/Scroll	V [M<mode>][S<scroll>]
		Where <range> is:	<addr>{,<addr>|L<length>}
	TEST:	Extended diagnostics	TEST
	SETUP:	Define hardware Setup	SETUP

TEST menu:

 			CHOOSE ONE OF THE FOLLOWING:

			1. DISK READ TEST
			2. KEYBOARD TEST
			3. BASE MEMORY TEST
			4. EXTENDED MEMORY TEST
			5. POWER-UP TEST
			6. EXIT
   
Z-386 MFM-300 Monitor, Version 3.2C [POST code list](https://github.com/raszpl/Zenith_ZBIOS/blob/main/POST%20codes.txt), crude decompiled [listing](https://github.com/raszpl/Zenith_ZBIOS/blob/main/zenith-386sx-bios-v3-2c.lst) and [IDA 6.1 dump](https://github.com/raszpl/Zenith_ZBIOS/raw/main/Zenith%20Z-386%20MFM-300%20Monitor,%20Version%203.2C.i64). I forgot to relocate binary to F000 before starting dissasembly and now I dont know how to change it post facto :(, means few jumps to raw F000 pointers show up as red. If you know how to fix it please drop me a hint :)

MFM Monitor shipped in models from three generations of Zenith computers.

386:
- [Zenith Data Systems (ZDS) 3797V1](https://theretroweb.com/motherboards/s/zenith-data-systems-3797v1)
- [Zenith Data Systems (ZDS) 3557V4](https://theretroweb.com/motherboards/s/zenith-data-systems-3557v4) both use [MFM-300 Monitor, Version 2.9B](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-386%20MFM-300%20Monitor,%20Version%202.9B.bin) [MFM-300 Monitor, Version 3.2C](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-386%20MFM-300%20Monitor,%20Version%203.2C.bin) [MFM-300 Monitor, Version 3.6D](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-386%20MFM-300%20Monitor,%20Version%203.6D.bin)

286:
- [Zenith Data Systems (ZDS) 85-3261-01](https://theretroweb.com/motherboards/s/zenith-85-3261-01) [MFM-200 Monitor, 2.0F](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-286%20MFM-200%20Monitor,%20Version%202.0F.bin)
- [Zenith Data Systems (ZDS) Z-248/12](https://theretroweb.com/motherboards/s/zenith-data-syst-z-248-12) [MFM-200 Monitor, Version 2.2](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-248%20MFM-200%20Monitor,%20Version%202.2.bin)

8080:
- [Zenith Data Systems (ZDS) Z-159](https://theretroweb.com/motherboards/s/zenith-data-syst-z-159) [MFM-1200 Monitor, Version 2.9](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-159%20MFM-1200%20Monitor,%20Version%202.9.bin)
- Live [Zenith Z-150](https://www.pcjs.org/machines/pcx86/zenith/z150/cga/) running in the browser [MFM-150 Monitor v3.1E](https://github.com/raszpl/Zenith_ZBIOS/raw/main/BIOSes/Zenith%20Z-150%20MFM-150%20Monitor,%20Version%203.1E.bin)
