## External interfaces

8x MMCX inputs
	Use edge launch connector and reinforce with epoxy to provide mechanical support

Provide plenty of ESD protection

Consider conformal coating

## Power budget

XC7A50T-1FTG256
4x AD9251
S27KS0642GABHI030

MCU: use STM32L031 in UQFN28 because easier to obtain than F031, and can do LVCMOS18 I/O which simplifies FPGA banking

## MCU I/O budget

	Bus to FPGA (5)
		CS#
		SI
		SO
		SCK
		INT

	Additional SPI chip select lines for ADCs (4)

## FPGA I/O budget

We have a total of 170 I/Os in the Artix-7 FTG256 package. Going to be tight!

Primary shared SPI bus between key chips with MCU as host and FPGA + 4x ADC as devices.

FPGA boot flash uses second dedicated QSPI bus at higher speed.

Clocks (2 LVDS inputs)
	One LVDS oscillator input (same clock domain as used for driving ADCs)

ADCs (144 LVCMOS18)

	Dedicated per converter, 4x (1 LVCMOS18):
		SYNC

	Dedicated per channel, 8x (16 LVCMOS18):
		OR (clip alert)
		DCO (source sync capture clock)
		D[13:0]

	Not going to FPGA:
		Differential clock to from oscillator through external 5-way fanout buffer

Boot flash (3-5, LVCMOS18):

	Hard minimum (3):
		SI/DQ0
		SO/DQ1
		CS#
		CLK (dedicated pin, does not count against GPIO usage)
	Nice to have (2):
		DQ[3:2]

HyperRAM (12 LVCMOS18):
	CS#
	Differential clock
	DQ[7:0]
	RDWS
	(Not using reset)

FPGA to MCU (5x LVCMOS18):
	CS#
	SI
	SO
	SCK
	INT

FPGA to outside world: we have 16 pins available at LVCMOS18 levels

	To FT232H (14x LVCMOS18 at FPGA -> LVCMOS33 at FTDI via external level shifter)
		ADBUS[7:0]
		RXF#
		TXE#
		RD#
		WR#
		CLKOUT
		OE#

	To level shifter
		1x LVCMOS18 direction control signal

	(SIWU# not used)

We have ONE free pin on the entire FPGA if we think of something else we might need!

## Memory bandwidth

Requirement: 10 Msps * 14 bits * 8 channels = 1.12 Gbps continuous write. 1.28 Gbps if we pad to 16 bits beforehand.

HyperRAM S27KS0642GABHI030
	64 Mbit (8 MB x8)
	LVCMOS18 interface, 200 MHz, DDR = 400 MT/s = 3.2 Gbps raw throughput

Single channel should be plenty. Which is good as we don't have the pins for a second channel.

## PC interface bandwidth

A single set of samples is 14 bits * 10 bits, padded to 16 (160 bits) for transfer to PC.
100K point memory depth gives +/- 50K points or +/- 5ms of centered trigger event.
This is 16 Mbits of data

Use FT232HL. Claims 40 MB/s = 320 Mbps transfer rates in datasheet. This is likely optimistic.

Assuming 250 Mbps achievable transfer rate, ~15 WFM/s streaming should be doable
