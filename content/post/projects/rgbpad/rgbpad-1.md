+++
authors = "kbeckmann"
categories = [ "projects" ]
date = "2016-08-08T21:00:00+02:00"
description = ""
title = "RGB Pad project"
+++

Xil.se is more than just a CTF team. We're doing some hardware hacks and will start posting about them now. First out is the RGB Pad. A quite simple and dirty project, but fun nonetheless.

[![alt tag](/images/projects/rgbpad/board.gif)](/images/projects/rgbpad/board.gif)

RGBPad is a button pad with fully addressable RGB LEDs. It is acting as an I2C slave and is designed to be connected in an array of multiple devices.

### Architecture
- Each RGBPad has an STM32F030C8T6 on board that acts as an I2C slave with a unique ID. This STM is dirt cheap and can be run without a crystal or passive components.
- The RGBPads are connected to each other on a I2C bus.
- When a button is pressed, the shared line NINT will be pulled down and the I2C master gets interrupted. This is to prevent the master from having to poll for button presses constantly. It may be a premature optimization though and will probably not be used.
- Each RGBPad has its own framebuffer for the LEDs. By sending a specific command, the framebuffer may be updated and pushed out to the LEDs.
- The LEDs used are the APA102. They have their own PWM and keep their state.

[![alt tag](/images/projects/rgbpad/board1.png)](/images/projects/rgbpad/board1.png)

### Why
Because it's fun.

The idea is to make another board (or use a raspberry pi or similar) that acts as a MIDI interface that uses these boards as keys/pads.

### Who
kbeckmann, arturo182

### Where's the code
- [Schematic and PCB design](http://git.xil.se/kbeckmann/rgbpad-hw)
- [Firmware for the RGBPad](http://git.xil.se/kbeckmann/rgbpad-fw)
- [Schematics in PDF](rgbpad-rev1-schematics.pdf)

### TODO
- [ ] Write all the code
- [ ] Implement I2C slave
  - [X] 0x01: Request button states. Reply with a uint16 with all the button states in row major order.
  - [X] 0x02: Set LED colors. Payload is a full framebuffer in snake order.
  - [ ] 0x03: Fade to framebuffer. Sets a new framebuffer and blend colors to this using s-curve interpolation/ease inout. Should take a uint16 with length of animation in milliseconds.
  - [ ] Use higher speed. 400 kHz should work, but we should try to go even faster.
  - [ ] Make it more reliable. Recover after faliures after connecting more boards while running.
- [ ] Implement I2C master on a STM32 for test that sends LED data and prints pressed buttons on UART.
  - Partly done, implemented a quick hack in lua for the NodeMCU.
- [ ] Implement some simple game (why not?).
- [ ] Finish the MIDI controller. But that's probably a project of its own.

### Project log

#### 2016-08-07
- PCB rev1 design is done.
- 10 PCBs have been ordered.
- 5 boards have been populated with components.
- Firmware for I2C slave is in a demoable state in 100kHz mode.
- Tested the firmware using a quick lua hack on a NodeMCU (ESP8266).
- Got a 3d printed bezel from arturo182.

![alt tag](/images/projects/rgbpad/board.jpg)
