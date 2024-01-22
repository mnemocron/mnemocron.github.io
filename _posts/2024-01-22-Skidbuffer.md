---
layout: post
title: Proper AXIS Pipelining using a Skid Buffer
subtitle: A quick & basic AXI bus template
gh-repo: mnemocron/axis-skidbuffer
gh-badge: [star, follow]
tags: [axi, axis, dsp]
comments: true
---

Yes, this is another blog post on the interwebz about the infamous _skid buffer_ or AXI pipeline stages (in particular the `tready` path).
One could think that something so essential to building AXI compliant IP is covered in detail everywhere you search.
And yet, this particular IP took me longer to develop than I would like to admit. This is the story of the **AXIS Skid Buffer**.

Before we dive deep into boolean logic structures, I want to mention the sources already on the internet.
There is an [article over at the ZipCPU blog](https://zipcpu.com/blog/2019/05/22/skidbuffer.html) which helped me understand __what__ a skid buffer does - on an abstracted higher level.
Unfortunately, it helped me little in understanding how to build a minimal, straight-forward skid buffer without a side quest in formal verification. 
Something that in hindsight would have been worth understanding - but here I am, with VHDL and no will to either circumvent the formal verification limitations of VHDL or properly learn Verilog.
Another **very useful article** is available as [part of some Red Pitaya documentation](https://pavel-demin.github.io/red-pitaya-notes/axi-interface-buffers/).
This article takes a different approach and goes straight into the details of how to implement AXI compliant register stages for both the `tready` (_"input"_) and `tdata`/`tvalid` (_"output"_) paths. It even includes pretty schematics :D
The design does however, have some flaws (or missing features) that I will explain further down.

The reason why I bothered to build my own skid buffer _slash_ AXIS pipeline stage is to have a fully registered AXI compliant template.
A template that can be used as a boilerplate to build quick bus modification functions.
I have used this in the past to build IP that is inserted into an AXI protocol connection to perform little adjustments to the signals.
For example:
- AXI4 to AXIS converter (write only) by dropping all address information (and emulating a `BRESP` response on the AXI4 interface)
- FIR filter stage (with Xilinx FIR compiler compatible AXIS interface)
- AXI4 virtual memory offset translator / base address register (I have not built that one yet)

It is also noteworthy, that the AXI specification requires that there are no combinatorial paths from input to output of a given IP.
Adding a simple skid buffer is an easy way to achieve this.

{: .box-note}
**What is a Skid Buffer?**
A skid buffer is used in a bus when you want to pipeline the feedback route from `S` to `M` (usually `tready`).
The problem when introducing latency into the feedback path is, that any upstream `M` devices will receive the actual _busy_ / _not ready_ signal too late.
Here, a skid buffer is introduced for the main data direction from `M` to `S` (usually `tdata`).
The bus transactions will _"skid"_ to a halt.
Therefore **the skid buffer is the shortest possible FIFO** of 0 stages in normal operation or 1 stage when halting.

### Minimal Skid Buffer (`tready` register)

The source in [Red Pitaya notes](https://pavel-demin.github.io/red-pitaya-notes/axi-interface-buffers/) splits the register paths into the _output buffer_ (`tdata`/`tvalid`) path and the _input buffer_ (`tready`) path.
_Fig. 1_ shows both registers combined. With `OPT_OUT_REG=False` it acts as a skidbuffer to pipeline `tready`.
The source code for this buffer is on Github: [skidbuffer.vhd](https://github.com/mnemocron/axis-skidbuffer/blob/master/vhdl/basic/skidbuffer.vhd).

![https://mnemocron.github.io/assets/img/skidbuffer/skidbuffer-schematic.png](https://mnemocron.github.io/assets/img/skidbuffer/skidbuffer-schematic.png){: .mx-auto.d-block :}
**Fig 1:** _Schematic of the skidbuffer / pipeline combo._

### The Problem with notifications

This chapter is bonus material. I struggled to verify a few properties that I needed in my AXIS IP.
These are:
1. Rising edge on `tvalid` can activate downstream IP
2. Rising edge on `tready` can activate upstream IP

The first property, I needed to support pipelining of a custom IP with variable sample rates. The upstream FIFO may be hold `tvalid=1` data at any given time but the downstream IP may only be `tready=1` every _Nth_ or so clock cycle. 
In this case `tvalid=1` must propagate to the downstream IP even when the downstream IP is not `tready=1` yet.
The same accounts for the upstream path using `tready=1`.
Through some simulations I could verify this behaviour to a satisfying degree.




