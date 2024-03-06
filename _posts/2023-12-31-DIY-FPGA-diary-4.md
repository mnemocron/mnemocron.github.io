---
layout: post
title: 7400 FPGA diary 4 - Debugging the PCB
subtitle: and documentation of the bitstream
gh-repo: mnemocron/my-discrete-fpga
gh-badge: [star, follow]
tags: [diy-fpga]
comments: true
---

![https://mnemocron.github.io/assets/img/fpga-diary-4/pcb-comparison.jpg](https://mnemocron.github.io/assets/img/fpga-diary-4/pcb-comparison.jpg){: .mx-auto.d-block :}
**Fig 1:** _PCB before and after assembly and debugging._

A month or so after sausageing around in KiCad I have some fresh PCBs on my desk ready for prototyping.
(Not for the life of me! did I want to make this a 4-layer PCB even though we live in 2024 and there are virtually no good arguments against 4-layers anymore. anyways.)
The boards look pretty. And yellow. Just like in the drawings I made.

{: .box-note}
_In theory, theory and practice are the same. In practice, they are not._

Well, in theory my digital twin (in VHDL, see previous article) and whatever I assembled on the PCB are the same. 
The first fuckup that was noticeable was the wrong polarity on the output enable (`~OE`) of the shift registers.
This condition made itself noticable by being a classic implementation of a WOM (write only memory). 
I had to embark on a mission to attach a modwire on every single shift register `~OE` pin after which all lights started flashing and blinking like a disco.
So I went ahead and implemented the 1-bit counter (inverter) on the PCB. **It worked!**
The next logical thing to test is a 2-bit counter which I immediately uploaded and saw it blink the wrong way.
I went back to the VHDL model but it did not spark joy to build another testbench just to figure out the wrong bit.
Now here is a different approach: **Test your digital design with an Arduino as the testbench**.
Because I figured, 

- a) I am quicker with breadboard wires and with C-code on an Arduino (vs. generating incremental stimuli data in Python and write a new VHDL testbench)
- b) the error(s) are more likely due to bad soldering on the PCB anyways

What I came up with is shown in _Fig. 2_.

![https://mnemocron.github.io/assets/img/fpga-diary-4/arduino-dut.png](https://mnemocron.github.io/assets/img/fpga-diary-4/arduino-dut.png){: .mx-auto.d-block :}
**Fig 2:** _Testbench approach with an Arduino Uno to provide bitstream, clocking and stimuli data and read back any results._

Thechnically, I had documented the bitstream before in an excel sheet. 
Because I absolutely needed this for when I want to set bits according to a truth table.
Let's see how the second bit should behave when implemented in LUT.
Note that the counter input bits are in reversed order - bit 0 is marked `m` and bit 1 is marked `s2`.
Since we are now using `LUT_B` this table tells me that I need to set the bits `36` to `43`.

![https://mnemocron.github.io/assets/img/fpga-diary-4/truth-table-bit1.png](https://mnemocron.github.io/assets/img/fpga-diary-4/truth-table-bit1.png){: .mx-auto.d-block :}
**Fig 3:** _Truth table for bit[1] of a counter._

Well, here is where it blinked the wrong way. 
So I coded an Arduino script which does the following things:

```
for each LUT in LUTs:
   for each bit in conf_bit:
      program_bitstream(bit)
      for each stim in input_vector:
         write(stim)
         clock_pulse()
         read(result)
         if(result)
            print("1")
         else
            print(".")
```

This will print a matrix of how the LUT behave to all input stimuli given all single bit configurations.
The output looked a bit like this:

![https://mnemocron.github.io/assets/img/fpga-diary-4/lut-a.png](https://mnemocron.github.io/assets/img/fpga-diary-4/lut-a.png){: .mx-auto.d-block :}
**Fig 4:** _Measured LUT output behaviour for LUT_A._

Now each configuration bit should act upon exactly one single input vector to produce a 1 at the output.
What we see here is some type of bit flip. And promptly, if I check back with the KiCad schematics, I produced a bug all by myself. 
The VHDL simulation would not have caught this.

![https://mnemocron.github.io/assets/img/fpga-diary-4/kicad-s0s1-bug.png](https://mnemocron.github.io/assets/img/fpga-diary-4/kicad-s0s1-bug.png){: .mx-auto.d-block :}
**Fig 5:** _Concept view of the proposed CLB._

Another journey into _modwire-land_ later, we have this problem sorted. However, there was more trouble.

![https://mnemocron.github.io/assets/img/fpga-diary-4/lut-d.png](https://mnemocron.github.io/assets/img/fpga-diary-4/lut-d.png){: .mx-auto.d-block :}
**Fig 4:** _Measured LUT output behaviour for LUT_D._

I'll spare you the details (because I don't actually know them myself) but this problem was caused by several bad (cold) solder connections.
After resoldering a bit of everything, it worked and produced a clean diagonal line of 1's.
Now I can use the above truth table in Excel to create the remaining LUT functions of an entire 4-bit counter.
And it works!!

![https://mnemocron.github.io/assets/img/fpga-diary-4/4-bit-counter-gif.gif](https://mnemocron.github.io/assets/img/fpga-diary-4/4-bit-counter-gif.gif){: .mx-auto.d-block :}
**Fig 5:** _4 bits counting ahead..._


We end with a first conclusion: Yes, it is possible to waste dozzens of 7400-family ICs to recreate half of a single 7400-family IC. I am thrilled to expand this project further.

![https://mnemocron.github.io/assets/img/fpga-diary-4/fraction-power-meme.jpg](https://mnemocron.github.io/assets/img/fpga-diary-4/fraction-power-meme.jpg){: .mx-auto.d-block :}
