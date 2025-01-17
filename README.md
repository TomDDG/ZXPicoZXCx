# ZX PicoIF2Lite
Sinclair Interface 2 replacement including ROM Cartridge emulation using a Raspberry Pico.

The main purpose of this interface is to replicate the [Sinclair Interface 2](https://en.wikipedia.org/wiki/ZX_Interface_2), adapting the original Pico ROM design by [Derek Fountain](https://github.com/derekfountain/zx-spectrum-pico-rom). Derek's idea was to use the Pico's flash memory in place of the ROM cartridges and I adapted this slightly to use the PiO rather than bit-banging, to use through hole components (as easier to build), to control the ROMCS line using the Pico and to also to include a joystick port. 

The original design always has to have a ROM paged in as the ROMCS line is permanently attached to 5V. This made it difficult to use shadow ROMs like with an Interface 1, and basically meant adding functionality to the Pico to page in shadow ROMs when needed. It also wasn't great on a 128k machine as it stopped the editor being useable. I therefore altered the design to be able to control the ROMCS line using the Pico which meant the interface could be disabled. This did mean losing the M1 line input to the Pico but this mirrors the original Interface 2.

The big advantage of disabling the Interface is reverting the Spectrum back to original state which improves compatibility with games etc... As of v0.3 I added [Z80 and SNA snapshot](#z80--sna-snapshot-compatibility) support which takes advantage of this. Once the game is copied to RAM the interface will turn itself off switching in the original ROM so the Spectrum behaves exactly as it would if you loaded the game from tape.

## The Interface 

![image](./images/back_nocoveron.jpg "ZX PicoIF2Lite")

The Interface is very simple with the Pico doing most of the heavy lifting. I've used 74LVC245 bus transceivers (as per Derek's original) to level shift both the address line (A0-A13) and data line (D0-D7) between the Pico and the edge connector. These chips also have the advantage of a high impedance mode when they are not required, something used on data line when the ROM isn't being accessed. The other chip in the circuit is a 4075 tri-input OR chip which is used to combine the A14, A15 & MREQ signals, which is basically indicating a ROM access. As the 4075 has three logic units I've also used this for the joystick.

Power for the Pico is provided by the 5V of the Spectrum via the edge connector, protected by a diode. I've also wired up a run button to easily reset the Pico and a user button which is used to control the Pico behaviour. More details of how to use the interface are below.

## The Joystick
Includes a simple 9pin joystick connector (DE-9 often referred to as DB-9), wired to Atari pinout and mapped to Sinclair 1 (67890). Note this is a simple hardware implementation inspired by the [Shadow of the Unicorn Interface](https://www.pcbway.com/project/shareproject/Mikro_Gen_Shadow_of_the_Unicorn_interface_Modern_re_creation_for_Sinclair_Spectrum.html) using diodes rather than the Pico (not enough GPIOs) or an inverter chip which most other interfaces use. As a result 5V is not sent to the adapter and autofire functions of some joysticks do not work. This is most likely due to the lack of a constant ground signal, normal fire and directions work fine.

The joystick circuit is pretty basic, it uses the 4075 OR chip to pull the GND pin of the joystick low when RD, A0, A12 & IORQ are low, high when any of them isn't low. This is basically simulating when the Spectrum is reading 67890 on the keyboard. If the output from the OR chip is low then if you move the joystick or press fire it grounds one of the data lines, either D0, D1, D2, D3 or D4. This creates the correct bit pattern for an `IN 0xFE` read on the Spectrum. Diodes protect the data line so it only zeros the bits when they are needed. Have +5v on the ground pin, other than when input is requested, is probably why autofire circuits don't work.

## Version Control
- v0.6 Simplified ROM includes adding a header to each ROM to replace romName & compatMode. New versions of compressROM & Z80toROM. **Latest Version
  - picoif2lite_diagROMboot.uf2 is an alternate version of v0.6 for those that cannot boot into the Spectrum due to bad memory or for troubleshooting when building the device. If you hold the top button when switching on the Spectrum it will directly boot into the Retroleum DiagROM bypassing everything else. This runs in ROM only so is good for troubleshooting.
- v0.5 ZX Spectrum machine code refactoring, LED matches ROMCS on/off, attempt to fix crash on reset. New version of Z80toROM with bug fixes. 
- v0.4 fixed issue with snapshots not loading on earlier Spectrum models
- v0.3 added converted Z80 and SNA ROM compatibility  
- v0.2 added ZXC2 cartridge compatibility
- v0.1 initial release taken form PicoIF2ROM but with interrupt driven user button

## Usage
Usage is very simple. On every cold boot the Interface will be off meaning the Spectrum will boot as if nothing attached. To activate the interface press and hold the user button for >1second, the Spectrum will now boot into the ROM Explorer. If you just want to reset the Spectrum just press the user button and do not hold down. The ROM Explorer is very easy to use and is in the style of a standard File Explorer. Use the cursor/arrow keys (5-left, 6-down, 7-up, 8-right and no need to press shift) to navigate the ROMs and enter to select one. ROMs with icons to the right hand side indicate they will launch with [ZXC2 cartridge (ZXC)](#zxc2-cartridge-compatibility) or [Z80/SNA (Z80) compatibility](#z80--sna-snapshot-compatibility).

To indicate that ZX PicoIF2Lite is controlling the ROMCS line the LED on the Pico will be on. If it relinquishes control of ROMCS the LED will go out. You can see this with some ZXC compatible ROMS, converted snapshots and if you turn the unit off.

More details of the [ROM Explorer](#the-rom-selector) can be found below.

## Building an Interface
I've provided everything you need to build your own Interface below, including the Gerbers for getting the PCBs made. I've also provided all the code so you can add your own ROMs and compile the necessary files to load onto the Pico. For details on how to set-up a build environment please refer to the [Raspberry Pico SDK documentation](https://www.raspberrypi.com/documentation/pico-sdk/)

Once built, load the UF2 file onto the Pico and boot the Spectrum. Hopefully all works fine.

### The Schematic
![image](./images/picoif2lite.png "Schematic")

### The PCB
![image](./images/picoif2lite_brd.png "Board")

All the files needed to make your own PCB are in the [Gerbers folder](./gerbers/), zipped. I've also popped them on [PCBWay](https://www.pcbway.com/project/shareproject/ZX_PicoZXCx_Sinclair_ZX_Spectrum_Interface_2_Replacement_18716605.html) if you find that easier.

### Bill of Materials (BoM)
- 1x [Raspberry Pico](https://shop.pimoroni.com/products/raspberry-pi-pico?variant=40059364311123). Recommend H variant as header pre-soldered.
- 3x [SN74LVC245AN Octal Bus Transceiver With 3-State Outputs](https://www.mouser.co.uk/ProductDetail/595-SN74LVC245AN). Note these need to be the LVC variant to work with the 3v3 of the Pico.
  - Optional 2x10pin sockets
- 1x [CDx4HC4075 Triple 3-Input OR Gates](https://www.mouser.co.uk/ProductDetail/595-CD74HC4075EE4) or equivalent.
  - Optional 2x7pin socket
- 4x [100nF/0.1uF Capacitor](https://www.mouser.co.uk/ProductDetail/Vishay-BC-Components/K104K15X7RF5TH5?qs=CuWZN%2F5Vbiofhf%252BuZNGw%2Fg%3D%3D)
- 7x [1N4148 Diode](https://www.mouser.co.uk/ProductDetail/onsemi-Fairchild/1N4148?qs=i4Fj9T%2FoRm8RMUhj5DeFQg%3D%3D)
- 1x [1N4001 Diode](https://www.mouser.co.uk/ProductDetail/Rectron/1N4001-B?qs=%252BtLcN0raKGUACwkD5chVvg%3D%3D). Can also use Schottky equivalent and 1A probably overkill but 1N4001 are fine and easy to get hold of, voltage drop isn't an issue with this circuit.
- 1x [56pin PCB Edge Connector](https://www.ebay.co.uk/sch/i.html?_from=R40&_trksid=p2047675.m570.l1313&_nkw=zx+spectrum+edge+connector&_sacat=0). Needs both ends cutting off, the pins removing top & bottom at position 5 and a piece of plastic or metal inserting for the slot (see [ZX Spectrum Edge Connector](https://i0.wp.com/projectspeccy.com/documents/ZXSpectrum_Edge_Connector_Diagram_48K.jpg)). This ensures everything aligns when connecting.
- 2x [20pin female header sockets](https://www.ebay.co.uk/sch/i.html?_from=R40&_trksid=p2380057.m570.l1313&_nkw=20pin+female+header+socket&_sacat=0)
- 1x [6x6x9mm Right Angled PCB Momentary Push Button](https://www.ebay.co.uk/sch/i.html?_from=R40&_trksid=p2380057.m570.l1313&_nkw=6x6x9mm+Right+Angled+PCB+Momentary+Push+Button&_sacat=0)
- 1x [6x6x9mm PCB Momentary Push Button](https://www.ebay.co.uk/sch/i.html?_from=R40&_trksid=p2334524.m570.l1313&_nkw=6x6x9mm+PCB+Momentary+Push+Button&_sacat=0&LH_TitleDesc=0&_osacat=0&_odkw=6+x+6+x9mm+right+angled+pcb+momentary+push+button)
- 1x [DE9 9pin Male PCB Connector](https://www.ebay.co.uk/sch/i.html?_from=R40&_trksid=p2334524.m570.l1313&_nkw=db9+9pin+pcb+male+connector&_sacat=0&LH_TitleDesc=0&_osacat=0&_odkw=db9+9pin+pcb+male+connector)

## The ROMs
For demonstration purposes I have included the following ROMs, if you are the owner of any of these ROMs and do not want it including please just contact me and I'll remove it:
- Original ZX Spectrum ROM (copyright Amstrad)
- [Retroleum DiagROM v1.59 by Retroleum](http://blog.retroleum.co.uk/electronics-articles/a-diagnostic-rom-image-for-the-zx-spectrum/)
- [ZX Spectrum Diagnostics v0.37 by Brendan Alford](https://github.com/brendanalford/zx-diagnostics/releases/tag/v0.37)
- ZX Spectrum Test Cartridge (copyright Amstrad)
- [128k RAM Tester by Paul Farrow](http://www.fruitcake.plus.com/Sinclair/Interface2/Cartridges/Interface2_RC_New_RAM_Tester.htm)
- [Looking Glass ROM by Geoff Wearmouth as used on the Spectrum Next](https://gitlab.com/thesmog358/tbblue/-/tree/master/machines/next)
- [Spectrum 128k Emulator by Paul Farrow](http://www.fruitcake.plus.com/Sinclair/Interface2/Cartridges/Interface2_RC_New_Spectrum_128.htm)
- [Spanish 128k Emulator by Paul Farrow](http://www.fruitcake.plus.com/Sinclair/Interface2/Cartridges/Interface2_RC_New_Spanish_128.htm)
- [Spectrum ROM Tester by Paul Farrow](http://www.fruitcake.plus.com/Sinclair/Interface2/Cartridges/Interface2_RC_New_ROM_Tester.htm)

### Adding your own ROMs
To add your own ROMs you need to first create a binary dump of the ROM (or just download it) and convert that into a `uint8_t` array to put in a header file. I've written a little utility to do this called `compressROM`. This utility uses a very simple compression algorithm to reduce the size of the ROMs which helps if you want to add a loads of them (max 126 or ~1.5MB). As part of the compression you can specify if the ROM should have ZXC2 compatibility and also what the display ane shoule be. The utility outputs the appropriate header file to put into the `rominc` folder (or a folder of your choice). For Z80 or SNA snapshots see the section below.

Once you've created the header you then need to add details about it to the `picoif2lite_lite.h` header file. This is in two parts.
1. include the header file
2. add it to the `roms` array in the position you want it to show in the ROM Selector

You can use the provided `picoif2lite_lite.h` header file as a guide. 

## Z80 & SNA Snapshot Compatibility
As of v0.3 the interface supports Z80 & SNA snapshots that have been converted into a ROM cartridge. This works with 48k and 128k snapshots. I've included a small utility, [Z80toROM](https://github.com/TomDDG/ZXPicoIF2Lite/blob/main/z80torom.c), which converts snapshots into the correct format and outputs a header file to include in the `rominc` folder as per normal ROMs.

The conversion of the snapshot to ROM is relatively simple and takes advantage of ROM paging and ability to switch off the interface. It works as follows:
- ROM 0 has the loader and compressed Memory Bank 5 (memory lcoation 0x4000, the one with the screen)
  - Upon launch the ROM copies a simple copy program to RAM (@0x6000) and jumps to this location after the copy
  - The code accesses memory location 0x3fff which tells the Pico to switch to the next ROM
- ROM 1 contains Memory Bank 2 (memory location 0x8000)
  - The copy routine simply copies ROM 1 to location 0x8000 to 0xbfff
  - When the routine accesses memory location 0x3fff the Pico again knows to switch the ROM to the next one (after the memory contents have been written)
- ROM 2 contains Memory Bank 0
  - The copy routine simply copies ROM 2 to location 0xc000 to 0xffff
  - When the routine accesses memory location 0x3fff the Pico does one of the following:
    - If it is a 48k snapshot it switches back in ROM 0 and jumps into this to finish the snapshot load
    - If it is a 128k snapshot it continues copying the rest of the memory - Bank 1, 3, 4, 6 & 7 in turn. After bank 7 it switches back to ROM 0 as per 48k.
- Back in ROM 0 the routine decompresses Memory Bank 5 to 0x4000 to 0x7fff
  - It then sets the registers and jumps back into memory for the final part of the loader
  - The final part of the loader (7bytes long) is placed either in the screen or just under the stack, if possible, to avoid screen corruption.
- The final part tells the interface to turn off, enables interrupts (if needed) and jumps to the correct program counter. The snapshot is now fully loaded.

Even with 128k Snapshots the loading is near instant. While the Pico is copying the ROM to memory the LED will flash.

## ZXC2 Cartridge Compatibility
While researching how to get the 128k ROM editor working on the device, before the ROMCS change, I remembered [Paul Farrow's FruitCake website](http://www.fruitcake.plus.com/Sinclair/Interface2/Interface2_ResourceCentre.htm) and the numerous cartridges and ROMs he had created. Some of those ROMs require software based bank switching and also for the unit to be disabled. Now that I could control the ROMCS line it was relatively easy to adapt the Pico code so that it could be compatible with Paul's ZX2 cartridge. As ZX2 compatibility isn't always desirable, due to it constantly scanning the top 64kB of ROM until you tell it not to, I added a toggle so that you can chose whether you want ZX2 compatibility or just run the unit as originally intended.

Full details of Paul's ZXC2 design and how it works can be found on [Paul Farrow's website](http://www.fruitcake.plus.com/Sinclair/Interface2/Cartridges/Interface2_RC_ZXC2.htm). 

The following is a quick summary of how I've implemented it using the Pico. When in ZXC2 compatibility mode, the Pico will scan each ROM address memory request. If the memory request is in the top 64bytes (`0x3fc0-0x3fff`) the Pico will do one of the following:
- If address bit 5 is on (1) the Pico will prevent further paging, basically locking the interface. This lock cannot be reversed via software and needs the user (reset) button pressing. Note you can still get into the ROM swap menu by pressing and holding the user button, this simply stops the software paging from working.
- If address bit 4 is on (1) the Pico will page out the interface ROM paging in the Spectrum ROM
- If address bit 4 is off (0) the Pico will page in the interface ROM, paging out the Spectrum ROM
- Address bits 0-3 are used to determine which ROM to page in, 0 is ROM 0, 5 is ROM 5 etc... I've set-up the Pico to mimic a 128k EPROM which allows for 8 banks/pages. 
  - To use this you need to create a single ROM binary with all the ROMs you want to swap in/out using the ZXC2 commands. 
  - The Spectrum 128k emulator ROM on Paul's website is a 32kB or 48kB ROM as an example. You don't need to split this into 2 or 3 16kB ROMs, just load the whole thing.

ZX2 compatibility was tested with [Spectrum ROM Tester](http://www.fruitcake.plus.com/Sinclair/Interface2/Cartridges/Interface2_RC_New_ROM_Tester.htm), [Spectrum 128k Emulator (with & without IF1)](http://www.fruitcake.plus.com/Sinclair/Interface2/Cartridges/Interface2_RC_New_Spectrum_128.htm) and [Spanish 128k Emulator](http://www.fruitcake.plus.com/Sinclair/Interface2/Cartridges/Interface2_RC_New_Spanish_128.htm)

## The ROM Selector
In order to swap between all the different ROMs the interface needs a simple ROM Selector utility which runs on the Spectrum. Once this has launched the Pico will constantly monitor the top of ROM memory, so to pick a ROM all the Spectrum code needs to do is loop over a memory read at the correct location between `0x3f80` and `0x3fff`. For example `0x3f80` is ROM 0, `0x3f96` is ROM 22. If a ROM is selected which doesn't exist the code will just pick the last ROM.

I've provided a fully working ROM Explorer program, in the style of File Explorer, which does exactly this. You can easily replace this with your own if you wish and I've highlighted the relavent sections in the code which need replacing.

![image](./images/romswitchblank_v1_1a.png "ROM Explorer")

The icons to the right handside of the text indicate which mode is used to load the ROM. `Z80` is for Z80 or SNA converted snapshots, `ZXC` is for ZXC2 compatibility and no icon means just a normal ROM.

![image](./images/romswitch.png "ROM Switch")

You can also use the selector to turn the device off, which pages in the Spectrum ROM, especially useful on 128k machines or if you want to use external devices with shadow ROMs i.e. Interface 1.

![image](./images/off.png "Off")

## The Case
I've created a 3D Printed Case for the interface. This makes it a lot easier to attach and remove from the Spectrum. You can download the STLs on [Thingiverse](https://www.thingiverse.com/thing:6074475) or [Printables](https://www.printables.com/model/503915-zx-picoif2lite-interface-case)

![image](./images/front.jpg "Case Front")

![image](./images/back.jpg "Case Back")
