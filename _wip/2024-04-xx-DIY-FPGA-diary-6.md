---
layout: post
title: 7400 FPGA diary 6 - Fizz Buzz!
subtitle: Building a "compiler" and adding another slice to the FPGA
gh-repo: mnemocron/my-discrete-fpga
gh-badge: [star, follow]
tags: [diy-fpga]
comments: true
---

_I love it when a plan comes together._ This blog post is about three topics:
- expand the FPGA by another slice, forming a 1x2 FPGA matrix
- programing a rudimentary assembly "compiler" so I do not have to code bits by hand
- runing a 4-bit counter AND a fizz-buzz detector

![https://mnemocron.github.io/assets/img/fpga-diary-6/compiler-design.png](https://mnemocron.github.io/assets/img/fpga-diary-6/compiler-design.png){: .mx-auto.d-block :}
**Fig 1:** _Single source - multiple targets._

## The Compiler

The disapointment first: No, I did not build a complete place and route algorithm that maps any VHDL/Verilog code to my architecture. 
And I do not intend to do that either.
Now to the update. My approach has always been to simplify the configuration of the bitstream into a slightly more intu.
I do not want to count bits with the method explained in the previous post.
What I want is an abstraction layer for the bitstream with mnemonics to enable certain features - sort of like an assembly language.

I will spare you the nightmarish details of ideas that I did not end up building. Ultimately, I opted for a C-library, using struct objects that can be interpreted as bit fields. The same code can be compiled with Arduino (to upload to the FPGA) or locally with GCC to export a bitstream.txt for VHDL simulations. (I do not yet have a testbench for an FPGA larger than 1x1 slices)


## How the compiler works

![https://mnemocron.github.io/assets/img/fpga-diary-6/fpga-arduino-1.png](https://mnemocron.github.io/assets/img/fpga-diary-6/fpga-arduino-1.png){: .mx-auto.d-block :}
**Fig 2:** _How different lines of code map to features on a single FPGA tile._

Consider the relationship between the lines of code in _Fig. 2_ to the enabled bits in the bitstream. At this point it should be evident why I picked green LEDs to highlight the bitstream configuration. It reminds me of the green wires in the _Implementation_ view in Vivado.
The gist of this way of coding is, that every bit in the bitstream now has a descriptive (although not very short) mnemonic to it.

Also note that I still use the Excel sheet to give me an instantiation vector for the LUT.
In this example I looped back the `bus[0]` signal and let the LUT invert it to achieve the good ol' blinky. Note that my signal LEDs have the color blue.

![https://mnemocron.github.io/assets/img/fpga-diary-6/lut-inverter.png](https://mnemocron.github.io/assets/img/fpga-diary-6/lut-inverter.png){: .mx-auto.d-block :}
**Fig 3:** _`0x00FF` is the LUT contents for an inverter bit acting on `S3`._

## Blinky

With this approach the C-code can grow quite a bit longer to configure the necessary bitstream for the previous 4-bit counter.

![https://mnemocron.github.io/assets/img/fpga-diary-6/fpga-arduino-2.png](https://mnemocron.github.io/assets/img/fpga-diary-6/fpga-arduino-2.png){: .mx-auto.d-block :}
**Fig 4:** _4-bit counter ._

## Fizz-Buzz

{: .box-warning}
**Warning:** Using multiple FPGA tiles in a matrix configuration inherently provides you with multiple `OUTPUT`s from CLBs. The compilation process using C cannot detect issues where multiple drivers are connected through the selected routing options. This creates the potential to connect two outputs together wich **can damage the hardware**.
Caution and/or bitstream simulation in VHDL is advised.

![https://mnemocron.github.io/assets/img/fpga-diary-6/fizz-buzz-truthtable.png](https://mnemocron.github.io/assets/img/fpga-diary-6/fizz-buzz-truthtable.png){: .mx-auto.d-block :}
**Fig 5:** _Creating the LUT contents from the truth table of Fizz-Buzz._

![https://mnemocron.github.io/assets/img/fpga-diary-6/2-slices-arduino.jpg](https://mnemocron.github.io/assets/img/fpga-diary-6/2-slices-arduino.jpg){: .mx-auto.d-block :}
**Fig 6:** _Programming the FPGA using the Arduino library._




