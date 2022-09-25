---
comments: true
slug: index
---
# Coral M.2 Module

It has been a long-time driver issue that the Google TPU Coral module doesn't support PCIe connection on Raspberry Pi 4, the only remaining way for raspberry pi is the USB 2.0 connection which for some bandwidth-heavy usage it will cause some performance impact. 


## Where is my USB3 on the Coral Module????

If we look at the datasheet for Coral Module, there is a note:
> Note: USB 3.0 is also available but requires special design considerations and support—for details, contact Coral Sales.

And the rest of the datasheet only mentions PCIe or USB 2.0 connection on the module, the only other discussion happened here: https://forums.raspberrypi.com/viewtopic.php?t=322485

The wires for the USB 3.0 connection is two differential pair, one for TX and the other for RX, so given that it is quite similar to the PCIe just without the clock, what are the chances that the USB 3.0 support is actually using the same pinout?

Let's examine the [Gumstix Raspberry Pi CM4 PoE Smart Camera](https://www.gumstix.com/manufacturer/cm4-poe-smart-camera.html), which is a CM4 board with Coral Module connected via ASM1142 chipset:
![](https://i.imgur.com/oIwRXGZ.png)

Double-check both Coral Module and ASM1142 pinout from the datasheet:
![](https://i.imgur.com/JtBk2Gj.png)
![](https://i.imgur.com/pIeGWoI.png)

Bingo, it looks like the connection is:  
Coral Module <---> USB 3.0 Controller  
"PCIe" TX <---> RX  
"PCIe" RX <---> TX  

And you can connect the USB2.0 D+,D- too.  


## USB 3.0 controller - where is the FW

So the next step is to find a USB 3.0 controller and make an M.2 Card with it, but the problem is which USB 3.0 controller?   

My first initial thought is using the same ASMedia ASM1142 used on the [Gumstix Raspberry Pi CM4 PoE Smart Camera](https://www.gumstix.com/manufacturer/cm4-poe-smart-camera.html) board, something like this:
![](https://i.imgur.com/d0WAkc3.jpg)

But the issue is that the freshly baked board doesn't have any firmware loaded in the SPI flash, and since I don't have the access to FW download tools, there is no way to load the FW without unsoldering and resolder. Both windows and Raspberry Pi 4 can recognize the board but no USB3 function.

So then I tried VL805, given that Raspberry Pi 4 already has support and provides an FW loading tool.
![](https://i.imgur.com/iy771Yf.jpg)

BUT - The FW loading tool has the SPI flash ID locked, it would only flash if the ID is 0xEF 0x10, which is Winbond W25Q10EW, but I'm using the same SPI flash I have around for the Raspberry Pi Pico which is W25Q16, so I need to patch the FW loading tool.
![](https://i.imgur.com/ckpxLo9.png)

A side note here is that supposedly you can ask the CM4 bootloader to load the FW for you so you can drop the SPI flash, but it doesn't seem to be working for me.

## How is the power looks like

So now I have a working Coral Module on an M.2 2242 form factor with USB 3.0 connection, but the board seems quite hot so I wonder what the power looks like.  

And here I present my setup:
![](https://i.imgur.com/D5WkUr4.jpg)


The setup is based on my CM4 Cluster daughter card and its breakout board with the M.2 3.3V powered externally via [Power Profiler Kit II](https://www.nordicsemi.com/Products/Development-hardware/Power-Profiler-Kit-2) from Nordic.

Here is the Card in idle just after boot-up:
351mA @ 3.3V = 1.15W
![](https://i.imgur.com/UbNvLQU.jpg)

Here is the trace of when the example code started:
![](https://i.imgur.com/0yd6FaV.jpg)

And here is the trace when the example code is running with iterations:
0.53A average = 1.7W
![](https://i.imgur.com/hIjTaQt.jpg)

Looking closer to one of the iterations, the peak power can reach 0.8A, which is about 2.64W:
![](https://i.imgur.com/7YNx8zs.jpg)

As a side note, yes, you can enable ASPM to lower the power but idle power only drop 20mA on my setup:
![](https://i.imgur.com/fXXyud3.jpg)


So my conclusion is that this board probably needs some heatsink to dissipate the heat. 


## Conclusion

This time we tested that the coral Module really is capable of USB3 without special hardware and it uses the same pinout as the PCIe port, and it is also possible to flash VL805 using the flash loader tool from the Raspberry Pi forum. The only concern or problem is that the Coral module and VL805 take quite some power that definitely needs a heatsink to dissipate the heat.

The board is also released in Opensource license (MIT), so feel free to take it to any project.
[https://github.com/will127534/Coral-USB3-M2-Module](https://github.com/will127534/Coral-USB3-M2-Module)
I was quite frustrated that I couldn't find any USB3 to PCIe Kicad design that I can base on during the design, given how mundane this might be on PC or even Raspberry Pi world.

I also really hope that someday chip vendor can just release their FW loading tools and FW for their chipset on their website too...



{% include comments.html %}