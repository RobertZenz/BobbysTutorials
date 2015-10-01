Cubietruck
==========


0. About this
-------------

This tutorial is licensed under [Creative Commons Attribution-ShareAlike][CC-BY-SA].


1. What is the Cubietruck?
--------------------------

The [Cubietruck][wiki-cubietruck] is the third version of the Cubieboard,
a System-on-a-cip board, like the [Raspberry Pi][raspberry-pi]. is fitted with
an Allwinner A20, 2GiB DDR3 RAM and most importantly it has a SATA 2.0 port.
That allows to attach storage devices to this little board.

The board itself is roughly 12x8cm in size.


2. The basics
-------------

There are some basic things that you should know about the board.

### 2.1 The LEDs

The board itself is fitted with five LEDs to display the state.

The most important one is the red LED right besides the power plug. This LEDs
is not, as I assumed, indicating that power is available, no it signals that
the system is running.

At the front it has a row of four additional LEDs, [I'll quote the
documentation][cubieboard-faq]:

 * Blue, hearbeat, signals that it is running.
 * Orange, CPU0, signals CPU0 activity.
 * White, CPU1, signals CPU1 activity.
 * Green, MMC, signals I/O on the card.

In my experience this is mostly true, except that the blue LED is only lighting
up for me during the boot process. After that it is quiet.

### 2.2 Power supply

You can either supply power via the power plug on the backside, or via
the [USB OTG][wiki-usb-otg] plug.

A "normal" USB connection should deliver enough power for powering the
Cubietruck. In my experience this must not be true, you should use a dedicated
power supply.

### 2.3 SATA power supply

An attached SATA device can draw its power directly from the board. For this
there are two white power "connectors" on the board right next to the SATA port.
One is labeled "12V SATA" and the other "5V SATA". The SATA cable that came
with the Cubietruck has besides the SATA plug itself also two plugs that are
fitting into those connectors, however they are not labeled but color coded:

 * 12V SATA == Yellow
 * 5V SATA == Red

Also, obviously, your power supply must also power the attached SATA device.


3. Flashing the NAND
--------------------

It is possible to flash the NAND with pre-made images, for this the tool called
"[LiveSuit][livesuit-download]" is necessary which allows to direcly flash
an (by USB) attached Cubietruck. [Ready images for flashing are available][images-download].


4. Installing Cubian to NAND
----------------------------

[Cubian][cubian] is a Debian version for the Cubietruck which can be run from
the memory card or installed to the NAND. However there are now pre-made images
available, the installation to the NAND is happening from inside a running
Cubian system. The [official tutorial for installine Cubian to NAND][cubian-nand]
is missing a possible problem you might run into. After installing Cubian to
NAND it is possible that the Cubietruck will not boor the Cubian system.
I was unable to identify the root cause but found a workaround. One has to flash
the official [Lubuntu images][images-download] to NAND first and then install
Cubian over it.

Some people assume that the Lubuntu image writes an bootloader (or some such)
to the NAND which Cubian is simply missing. So flash it first with Lubuntu,
then install Cubian.


5. Issues I've run into
-----------------------

### 5.1 Cubietruck is shutting down or constantly rebooting during boot

Your power supply is insufficient. Either it is not delivering enough power
or there is something wrong with it. Try a short cable first, and if that does
not help switch the power supply.


### 5.2 Cubietruck is screeching (especially with attached SATA device)

Your power supply is insufficient, see the tips above.

I could not pinpoint why the Cubietruck starts to "screech" if that is the case,
but it sounded like a harddisk that is trying to spin up.


6. Conclusion
-------------

Everything else is pretty much straightforward, the installation process of
Cubian is easy and the overall system behaves like any other Linux installation.



 [CC-BY-SA]: http://creativecommons.org/licenses/by-sa/4.0/
 [cubian]: http://cubian.org/
 [cubian-nand]: https://github.com/cubieplayer/cubian/wiki/Install-Cubian
 [cubieboard-faq]: http://docs.cubieboard.org/faq/faqs
 [images-download]: http://dl.cubieboard.org/model/cubietruck/Image/
 [livesuit-download]: http://dl.cubieboard.org/model/cubietruck/Tools/
 [raspberry-pi]: https://www.raspberrypi.org/
 [wiki-cubietruck]: https://en.wikipedia.org/wiki/Cubieboard#Cubietruck_.28Cubieboard3.29
 [wiki-usb-otg]: https://en.wikipedia.org/wiki/USB_On-The-Go

