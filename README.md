# pci_mini-FPGA-IP
Parallel PCI-bus to Wishbone bus bridge VHDL FPGA IP core

Originally hosted (in 2008) at:
https://opencores.org/projects/pci_mini
Since the OpenCores.org admins have abandoned their website, and don't allow new users to register and download old projects (like this one), I, Istvan Nagy, the sole author, decided to re-release the project on GitHub. The new release is with a new, less restrictive license: "BSD zero clause", instead of the old LGPL. This will allow companies to use it in their product without restrictions or obligations.
The BSD 0-clause license: Copyright (C) 2024 by Istvan Nagy buenoshun@gmail.com 
Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted. THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE.

This is a very simple PCI-target to Wishbone-master bridge.
PCI-Target only, the bandwidth is quite low, fixed memory-image size (16MB), but it has a very low FPGA logic resource need.
This is a single VHDL (old version was verilog) file, so easy to use.

The original PCI module is from: Ben Jackson http:www.ben.com/minipci/verilog.php
completely redesigned for wishbone and VHDL : Istvan Nagy, Hardware Design Engineer. www.buenos.extra.hu

Code variants/versions:
-----------------------
- Up till v3.3 (verilog) the code was tested on Xilinx Spartan-2/3 FPGAs using XST to synthesise. These versions don't have the automatic read-retry feature implemented, so the software developer has to read every location twice. Files: pci-33.v, sample_timing_constraints.ucf.txt.
- V3.4 (verilog) contains modifications to make it work on Actel/Microsemi ProASIC3 FPGAs using Synplify for synthesis, and also implements the automatic read-retry PCI feature. It means that the read transactions are terminated by the target with "retry" so next time the host accesses the device with read from the same address will return the right read data and the target terminates with successful access. This effects the software development in a way that we don't have to read every location twice just once. v3.4 was not tested on Xilinx FPGAs. Files: pci_mini-34.v, pci_mini-34_timing_constraints.sdc
- v4.0 (VHDL) is completely rewritten from scratch in VHDL. The reason for that was to improve static timing and to avoid strange behaviour of the v3.4 seen on Actel FPGAs with a few certain configurations. For example the input setup time and clock-to-output-delay was reduced from ~10ns to ~3ns. A few features were removed, like address remapping and user-reset-control. This version has to be used with the pci_mini_40_timing_constraints.sdc timing constraints file. This was only tested on Actel ProAsic3 FPGAs.

Test results:
-------------
Tested on hardware:
-PCI card (with SP2 FPGA) plugged into an old PC with Pentium-II CPU and VIA VT82C693A+VT82C596B chipset
-Custom motherboard designed by me, with the AMD Geode-LX processor, Geode chipset, and a Xilinx Spartan-3 FPGA.
-Com-Express carrier board with Intel Atom Z520 processor (on the COMe), and Actel ProASIC3 FPGA (on the carrier).
Test software: Hardware-Direct, Read-Write-Everything.
FPGA project: a peripheral block, consisting: Wishbone intercon module, CAN controller, some custom peripherals, and the PCI2WB bridge.
Although it was working in the 3 test systems, it was not validated against all the specifications of the PCI standard, it means it was not tested with standard compliance test methods (like Agilent PCI excercisers and analyzers). So no guarantee for anything, if you want to use it, you should test it. The 33MHz PCI 2.0 I/O-timing specifications were met (can be seen on the timing reports of the ISE development tool.), guaranteed by the timing constraints in the ucf/sdc files and the development tools. On the Spartan-2 FPGA board, the maximum frequency in the Xilinx-ISE STA result was just a littlebit above the 33MHz target (it was 39MHz) for the whole PCI-wishsbone system (the wishbone system used the PCI clcok too, so it was a single clock-domain synchronous system). The v4.0 on the A3P400 device meets the timing requirements of the PCI-66MHz (post P&R STA report), although it was never tested on 66MHz.

Synthesis:
----------
v3.3: 279 Slices on Xilinx Spartan-3 FPGA. (14.5% logic on SP3-200k)
v4.0: 719 Core Cells on Actel ProAsic3 FPGA. (8% logic on A3P400)


For simulating the core: generate 2 pci config-write transactions for initializing the bridge: write a base address (multiple of 16MB) to BAR0 at 14h , then enable by writing 7 to address 4h. after this, your transactions (single memory read/write at the address range specified by the base address) will go through the bridge. (devsel timing = "fast")

Features
- PCI target interface
- 16MB memory image
- Wishbone master interface
- Configurable address translation through a config space user register
- Single dword buffering (only 32bit mode is supported)
- No burst transactions (mem r/w multiple) supported.
- Delayed read request and posted write data transfers. (on v3.3 the reads are not retried automatically, so the user software has to initiate two reads from the same address, for every Dword. v3.4/v4.0 implements auto-retry)
- Configurable address translation through a config space user register (v3.3/v3.4 only)
- Downstream system control by 8 GPIO signals, throug config space (reset, low power mode..., v3.3/v3.4 only)

Status
- finished, tested
