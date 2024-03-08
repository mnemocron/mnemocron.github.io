---
layout: post
title: 7400 FPGA diary 5 - Hello World
subtitle: program single bits in the bitstream
gh-repo: mnemocron/my-discrete-fpga
gh-badge: [star, follow]
tags: [diy-fpga]
comments: true
---

Since the completion of the most complicated modular PCB module, the CLB, I have successfully assembled the surrounding interconnect PCBs.
As it turns out, you _can_ reflow solder PCBs on the stove in your kitchen.
In unrelated news, does anybody have tips on how to get rid of unwanted smells in your apartment?

In theory, my PCB design should match the previously designed VHDL model. 
In practive, I messed up the byte order on the switchbox PCB (which was a pain to layout precisely because of the byte order).
Other than that, all of the SW, CBh and CBv are working flawlessly as designed.
It is now time to verify previously generated and simulated bitstreams on actual hardware.
But how do you program an FPGA by bitstream directly?

## The Blinky Example

### 1. Routing

![https://mnemocron.github.io/assets/img/fpga-diary-5/fpga-arch-tile-1bit.png](https://mnemocron.github.io/assets/img/fpga-diary-5/fpga-arch-tile-1bit.png){: .mx-auto.d-block :}
**Fig 1:** _Schematic of the loopback routing of a single bit._

Let's start with the LSB of a counter which behaves like an inverter - or a blinky.
This signal will have to be routed from the output of the LUT back to one of the four inputs of the LUT as shown in _Fig. 1_.

I am going to pick LUT output `A` or `0` as my LSB.
This bit will exit the CLB on the 4-bit wide bus at position `[0]` and enter the vertical connection box (CBv) on its west input bus.
To enable this signal to the horizontal routing lines the crosspoint `xp_bus[0]` must be enabled. 
This is done by setting a `1` at the corresponding location in the bitstream. In the case of `xp_bus[0]` this is bit `7` as indicated by the bitstream documentation in _Fig. 2_.

![https://mnemocron.github.io/assets/img/fpga-diary-5/fbitstream_doc_cbv.png](https://mnemocron.github.io/assets/img/fpga-diary-5/fbitstream_doc_cbv.png){: .mx-auto.d-block :}
**Fig 2:** _Excerpt from the bitstream documentation containing the CBv settings (`bitstream_doc.xlsx`)._

The signal will now arrive at the south bus of the switch box (SW). To correctly route it to the horizontal connection box (CBh) three bits must be set.
Two bits are necessary to enable the bus signal to enter and leave the crosspoint matrix (`en_bus_south[0]` / `en_bus_west[0]`) and one more bit to enable the crosspoint bit `xp_bus[0]` on the matrix itself.

![https://mnemocron.github.io/assets/img/fpga-diary-5/presel_0.png](https://mnemocron.github.io/assets/img/fpga-diary-5/presel_0.png){: .mx-auto.d-block :}
**Fig 3:** _Input multiplexer to the LUT._

Next, the CBh creates the input selection for all the LUTs in the CLB. The unprogrammed value will default to `GND`.
Therefore, a corresponding value must be programmed to select the correct input signal from `bus[0]`.
This is done with three bits named `presel_0[2:0]` which will serve as an address to the 8-input multiplexer.
To select the `bus[0]` line, the multiplexer must be set to input 7 (as shown in _Fig. 3_) which translates to `presel_0[2:0] = "111"`.

### 2. LUT contents

Now that the signal is routed, a behaviour must be defined inside the LUT.
One LUT4 contains a 16-bit long truth table that must be programmed.

> And by the way, this can be done on modern FPGAs too by manually instantiating a LUT primitive and define its contents as a hex vector: [Xilinx UG953](https://docs.xilinx.com/r/en-US/ug953-vivado-7series-libraries/LUT6).

Since counting to 16 with your fingers, I created another table to look up the bits that need to be set to one for a given truth table. This is more of a visual aid than a complex tool. See _Fig. 4_ where the toggle bit at the output `Y` is reacting to the input `S0`. To map this truth table to `LUT A` one has to map the `1`-bits from the `Y` column to the bitstream address of the `LUT A` column.

![https://mnemocron.github.io/assets/img/fpga-diary-5/truth-table.png](https://mnemocron.github.io/assets/img/fpga-diary-5/truth-table.png){: .mx-auto.d-block :}
**Fig 4:** _Truth table for a  (`lut_truth_table.xlsx`)._

### 3. CLB options

Finally, the CLB itself has a few features that need to be enabled. 
For proper RTL operation, we enable clocking with the `en_reg_lut_a` bit (`15`).

### Code

Before uploading the bitstream to the actual hardware, it can be simulated with the previously built VHDL model.
A pyhton script can build the bitstream for the testbench from the bit addresses that need to be set to `1`.

```python
set_bits = []
set_bits.append(7) # CBv LUT A -> bus[0] enable
set_bits.append(111) # SW xpoint_1 south[0] to west [0]
set_bits.append(119) # SW en_bus_south[0]
set_bits.append(127) # SW en_bus_west[0]
set_bits.append(101) # CBh presel_3[2] = +4
set_bits.append(102) # CBh presel_3[1] = +2
set_bits.append(103) # CBh presel_3[0] = +1
set_bits.append(15) # CLB LUT en reg a

for b in [17,19,21,23,25,27,29,31] :
    set_bits.append(b) 
```

For the upload to the hardware, I still use an Arduino and the corresponding C-code looks similar:

```cpp
#define BITS_CONFIGURED 15

char conf_bits[BITS_CONFIGURED] = {
   7,   // CBv: enable LUT_a output
   111, // SW:  xp_bus[0]
   119, // SW:  en_bus_south[0]
   127, // SW:  en_bus_west[0]
   101, // CBh: presel_0[2] --> MUX addr +4
   102, // CBh: presel_0[1] --> MUX addr +2
   103, // CBh: presel_0[0] --> MUX addr +1
   15,
   17,19,21,23,25,27,29,31};
```

Here you have it: assembly for FPGA.

### Simulation

![https://mnemocron.github.io/assets/img/fpga-diary-5/simulation-workspace.png](https://mnemocron.github.io/assets/img/fpga-diary-5/simulation-workspace.png){: .mx-auto.d-block :}
**Fig 5:** _The VHDL simulation workspace programs the bitstream into one FPGA tile and simulates its behaviour: a toggling bit._

### Run on actual hardware

![https://mnemocron.github.io/assets/img/fpga-diary-5/routing-on-pcb.png](https://mnemocron.github.io/assets/img/fpga-diary-5/routing-on-pcb.png){: .mx-auto.d-block :}
**Fig 6:** _Visual aid of how the configuration is routed on the FPGA tile for a single toggle bit._

### F-max Estimation

![https://mnemocron.github.io/assets/img/fpga-diary-5/toggle-bit-fmax.png](https://mnemocron.github.io/assets/img/fpga-diary-5/toggle-bit-fmax.png){: .mx-auto.d-block :}
**Fig 7:** _Scope view of the ring oscillator with a frequency of 13.9 MHz._

With the hardware working, I can now test the maximum speed of my FPGA by building a ring-oscillator.
To do so, I just disable the output register (`en_reg_lut_a` / bit `15`) to switch from RTL to combinatorial logic.
My scope tells me a toggle rate of some 13.9 MHz (!). This makes me happy. 
Because in my wildest dreams, I am already considering adding some DSP capabilities with shared hardware.
Let's say, you have an audio signal, sampled at 8-bit / 44.1 kHz while the larger FPGA runs at a conservative 5 MHz.
Then you would have approximately 100 clock cycles per sample to perform some DSP. That may be enough for quite a few FIR taps.

## Conclusion

All-in-all a great first success.
I would say, the greatest challenge so far is not the hardware itself - 7400 logic is simple and straight forward - no, the challenge is keeping both the VHDL model up-to date with the PCB (or vice versa). For example with rev. 2 of the PCB for the CLB I decided last minute, I need a clock enable for the flip flop. 
Luckily I had some unused _"reserved"_ configuration bits left both on the CLB and the CBh.
But changing such a thing required me to go back to the VHDL and implement and test this feature as well.
Then of course, a lot of testing is required that the same bitstream produces the same results in the simulation and on the hardware.


