---
layout: post
title: AXIS Pipelining using a Skidbuffer
subtitle: A quick & basic AXI bus template
gh-repo: mnemocron/my-discrete-fpga
gh-badge: [star, follow]
tags: [axi, axis, dsp]
comments: true
author: Simon
---

Yehees, another blog post explaining the concept of the skidbuffer.
Before we start, I want to mention the [article over at the ZipCPU blog](https://zipcpu.com/blog/2019/05/22/skidbuffer.html) which helped me understand the basics of a skidbuffer. At least after a while. 
He goes a long way to explain the inner workings in an abstract way to then properly verify the design.
The focus of this article is to give the absolute basics as simple as possible - in VHDL.

So the reason why I bothered to my own skidbuffer _slash_ AXIS pipeline stage is to have a fully registered AXI compliant template.
A template that can be used as a boilerplate to build quick bus modification functions.
I have used this in the past to build IP that is inserted into an AXI protocol connection to perform little adjustments to the signals.
For example:
- AXI4 (write only) to AXIS converter by dropping all address information (and emulating `BRESP` response)
- AXI4 virtual memory offset translator (have not built that one yet)

{: .box-note}
**What is a Skidbuffer?**
A skidbuffer is used in a bus when you want to pipeline the feedback route from `S` to `M` (usually `tready`).
The problem when introducing latency into the feedback path is, that any upstream `M` devices will receive the actual _busy_ / _not ready_ signal too late.
Here a skid buffer is introduced for the main data direction from `M` to `S` (usually `tdata`).
The bus transactions will _"skid"_ to a halt.
Therefore **the skidbuffer is the shortest possible FIFO** of 0 stages in normal operation or 1 stage when halting.

### Minimal Skidbuffer

Note that the following skidbuffer is intentionally kept generic.
Any bus or channel using the `data`/`valid`/`ready` paradigm can be routed through this buffer.
This is especially true for all the channels of AXI4 or the AXI Stream bus. 
Note that any additional (optional) signals like `tkeep` / `tuser` etc. can be merged into the main data bus in a top level wrapper as shown further down.

_Fig. 1_ gives the theoretical waveforms for how a skidbuffer must react in various situations.
`tready` can arrive at any moment (before or while `tvalid` is asserted).

![https://mnemocron.github.io/assets/img/skidbuffer/skidbuf_passthru_wave.svg](https://mnemocron.github.io/assets/img/skidbuffer/skidbuf_passthru_wave.svg){: .mx-auto.d-block :}
**Fig 1:** _Waveform for an AXIS skidbuffer._

_Fig. 2_ gives a block diagram version of how this skidbuffer can be implemented using multiplexers and registers.
Note that only the `ready` signal is registered. This can be used to ease timing problems arising from the `ready` signal. 
However, the additional logic in the `data`/`valid` path may complicate timing. 
Adding another register stage at the output completes the pipelining of all AXI signals:

![https://mnemocron.github.io/assets/img/skidbuffer/simple-skidbuffer.png](https://mnemocron.github.io/assets/img/skidbuffer/simple-skidbuffer.png){: .mx-auto.d-block :}
**Fig 2:** _Schematic block diagram of the minimal skidbuffer._

### Additional output register stage

_Fig. 3_ shows similar waveforms as before with the downstream `data`/`ready` signals delayed by one clock cycle. 
This changes some of the interactions. However, all valid input (upstream) transactions return valid output (downstream) transactions.

![https://mnemocron.github.io/assets/img/skidbuffer/skidbuf_fullreg_wave.svg](https://mnemocron.github.io/assets/img/skidbuffer/skidbuf_fullreg_wave.svg){: .mx-auto.d-block :}
**Fig 3:** _Waveform for an AXIS register pipeline stage._

Adding another register output stage at the output is shown in _Fig. 4_. Note that it will only change the output state once the downstream slave has accepted the data in the output register stage. This mechanism is guaranteed to work given that the upstream master is stalled by the deasserted `ready`.

![https://mnemocron.github.io/assets/img/skidbuffer/full-skidbuffer.png](https://mnemocron.github.io/assets/img/skidbuffer/full-skidbuffer.png){: .mx-auto.d-block :}
**Fig 4:** _Schematic block diagram of the minimal skidbuffer._




