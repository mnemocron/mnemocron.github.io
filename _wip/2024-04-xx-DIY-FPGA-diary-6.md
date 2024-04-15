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

![https://mnemocron.github.io/assets/img/fpga-diary-6/sorry-bro-compiler.png](https://mnemocron.github.io/assets/img/fpga-diary-6/sorry-bro-compiler.png){: .mx-auto.d-block :}
**Fig 1:** _Sorry Bro, keep scrolling._

## The Compiler

The disapointment first: No, I did not build a complete place and route algorithm that maps any VHDL/Verilog code to my architecture. 
And I do not intend to do that either.
Now to the update. My approach has always been to simplify the configuration of the bitstream into a slightly more intu.
I do not want to count bits with the method explained in the previous post.
What I want is an abstraction layer for the bitstream with mnemonics to enable certain features - sort of like an assembly language.

I will spare you the nightmarish details of ideas that I did not end up building. Ultimately, I opted for a C-library, using struct objects that can be interpreted as bit fields. The same code can be compiled with Arduino (to upload to the FPGA) or locally with GCC to export a bitstream.txt for VHDL simulations. (I do not yet have a testbench for an FPGA larger than 1x1 slices)

![https://mnemocron.github.io/assets/img/fpga-diary-6/2-slices-arduino.jpg](https://mnemocron.github.io/assets/img/fpga-diary-6/2-slices-arduino.jpg){: .mx-auto.d-block :}
**Fig 2:** _Programming the FPGA using the Arduino library._


![https://mnemocron.github.io/assets/img/fpga-diary-6/fizz-buzz-truthtable.png](https://mnemocron.github.io/assets/img/fpga-diary-6/fizz-buzz-truthtable.png){: .mx-auto.d-block :}
**Fig 3:** _Creating the LUT contents from the truth table of Fizz-Buzz._


