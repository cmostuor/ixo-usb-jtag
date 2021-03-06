
nexys2prog - program a Digilent Nexys 2 FPGA board over USB
Copyright (c) 2009, Andrew Ross <andy@plausible.org>
http://ixo-jtag.sourceforge.net/

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License along
with this program as the file COPYING.txt; if not, write to the Free
Software Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston,
MA 02110-1301 USA.

News
====

- May 2010: Imported into the SourceForge project ixo-jtag by Hauke Daempfling
  http://ixo-jtag.sourceforge.net/
  I've updated the script with the latest version of the Cypress FX2 firmware,
  and updated the documentation a tiny bit. The original documentation
  follows. Note that some of the links no longer work, but the general idea is
  still the same. Also, note that the described fixes to the firmware have
  since been applied to the main source tree hosted at the SF project above.
  The nexys2prog script was originally announced here:
  http://groups.google.com/group/comp.arch.fpga/browse_thread/thread/c7557a7b4bc9900c

nexys2prog
==========

[Executive summary for Debian/Ubuntu users: after installing ISE and
 setting up your environment to use it, install fxload and libftdi1
 using apt-get, and build and install UrJTAG from http://urjtag.org.
 Then call this script with the name of the Xilinx bitstream file you
 would like to program.  That's it.  See the attached led.v verilog
 file and build.sh script for a minimal sample.]

The Digilent Nexys 2 is a great FPGA development board, with a ton of
I/O available on board, a very large (XC3S1200E) FPGA part available,
and an excellent selection of add-on devices for a very reasonable
price.

It is also, like most such devices, completely unsupported under
linux.  Nonetheless it can be made to work, via a distressingly long
chain of responsibility.  Bear with me, I think I have most of this
puzzled out:

Xilinx, of course, provides a Linux version of their free (as-in-beer)
ISE WebPack product [1].  But Digilent uses a proprietary protocol for
their USB interface, and they neither document it nor provide tools
other than a (decidedly clunky) Windows GUI [2] to use it.

Nonetheless the USB hardware on the device is open.  The Cypress FX2
USB chip on the board, which itself is a 8051 microcontroller, is
fully programmable from across the USB bus [3] and that interface is
supported from linux thanks to the work of the linux-hotplug project
[4].  The sdcc project provides a free C compiler [5] for the 8051,
which Kolja Waschk used to write a firmware suite named "usb_jtag" [6]
for a FX2-based USB/JTAG cable that allows it to be used as a
compatible replacement for an Altera USB Blaster -- a cable based on a
different USB interface part from FTDI, which is supported under linux
by the libftdi project [7].  The UrJTAG project [8], which is a
currently-maintained fork of the mostly-abandonware openwince-jtag
project, provides a high level JTAG interface (using libftdi as one of
many drivers) that can be used to program the FPGA using SVF files
from the Xilinx iMPACT tool.  The Nexys 2 board enters the picture at
last when Sune Mai, in posts on fpga4fun.com [9] and on the UrJTAG
mailing list [10], ported Waschk's usb_jtag firmware to the Nexys 2 by
simply changing the 8051 port assignments of the JTAG pins (the FX2 on
the Digilent board is wired differently than the usb_jtag cable, but
is otherwise compatible) and by fixing a integer overflow bug the
upstream code had with handling large bitstream files.  Neither change
has been merged into Kolja's code, unfortunately.

Finally, Morgan Delahaye-Prat collected the above into a single
walkthrough on his (French) blog [11], providing detailed instructions
and downloadable files and patches.  The language barrier for
non-French-speakers can be surmounted without too much difficulty via
google's language tools.  The Rube Goldberg-like complexity of the
process, however, took much longer to puzzle out and left me with a
tree full of tiny scripts, notes, and patched software trees.

So at last we come to my contribution.  This nexys2prog script tries
to eliminate as much complexity as possible.  The script will
automatically probe the USB bus to detect the Nexys 2 board and
reconfigure it if necessary using a pre-built firmware image stored in
the script itself.  It will automatically generate SVF files from
Xilinx bitstream images, and push them to the device using a
configuration automatically generated from data files in the ISE tree
and a built-in notion of how the JTAG bus on the board is configured.
The user, ultimately, is responsible only for invoking the script with
the name of a .bit file they would like to push to the FPGA.

External dependencies have been limited to just Xilinx ISE (required
for anyone doing development), fxload (available via apt-get from
Debian or Ubuntu), libftdi1 (likewise a Debian package), and UrJTAG
(source package buildable with a simple "configure;make;make
install").  A permissions issue with USB device files on Ubuntu
requires a small modification to the udev configuration, or simply
running the script as root.

For questions and comments please see the project page at
http://ixo-jtag.sourceforge.net/

[1] http://www.xilinx.com/ise/logic_design_prod/webpack.htm
[2] http://digilentinc.com/Products/Detail.cfm?Prod=ADEPT
[3] http://download.cypress.com.edgesuite.net/design_resources/datasheets/contents/cy7c68013a_8.pdf
[4] http://linux-hotplug.sourceforge.net
[5] http://sdcc.sourceforge.net
[6] http://www.ixo.de/info/usb_jtag/
[7] http://www.intra2net.com/en/developer/libftdi
[8] http://urjtag.org
[9] http://www.fpga4fun.com/forum/viewtopic.php?p=4779
[10] http://sourceforge.net/forum/forum.php?thread_id=2312452&forum_id=682993
[11] http://www.m-del.net/info.html
