# AXI-Lite GPIO over 100 Gbit Serial Aurora

Before I go too much into detail, no! I did not build a 100G AXI-Lite slave.
I merely routed AXI-Lite traffic over a 100G serial link for "evaluation purposes" and: because I can.

Imagine yourself having just arrived at home from a from a hugely unproductive and somewhat mentally exhausting day at the office because you were fighting Xilinx tools (entirely fictional experience).
Before you are getting comfortable on the sofa, you are want to cook this vegetarian chickpea curry and pull out the recipe on your phone.
Instead of getting straight to the instructions, the recipe reads more like a diary. 
The author goes to lengths about their experiences in India and how fantastic this particular recipe is. 
Before you know it you have scrolled the distance of a half-marathon to arrive at the bottom line containing the list of ingredients.
Welcome on my blog, this is not a technical documentation. I can ramble about Sweden first.

Back in 2022 I went on an Erasmus student exchange semester to the University of Linköping in Sweden.
I had the opportunity to briefly work as a research assistant at the institute for electronics and systems. 
At the begining I was given the choice between two projects:
- do I want to develop some Arduino libraries for LIDAR chips for the undergrad students to use?
- or do I want your very first FPGA project to be on this 9'000 USD Virtex UltraScale+ plattform to evaluate a 100G link?
Many things happened over the course of these 6 months. Me touching Arduino software was not one of them.
And so I evaluated various 100G capable protocols that are later to be used in 5G MIMO systems.

It turns out, the serial transceivers on FPGAs just do their transceiving thing. And even though they are bundled to a GT quad and routed to a Quad SFP (QSFP) connector, it is still up to the designer how to transport data over this physical layer.
There are a hand full of standards, all suited for such a task.

- _Interlaken_
- _CPRI_ (Common Public Radio Interface)
- _RoCE_ (RDMA Ethernet)
- _Aurora_ 

Now, Interlaken is a chip-to-chip interface not really made for long distances. CPRI seems cool because it is exactly for telecommunication infrastructure.
However, both of them are premium IP cores so I did not look into them further.
RoCE is an Ethernet standard protocol supporting RDMA transfers. Its main advantage is that Nvidia sells you cards that can speak RoCE at 100Gbps.
So instead of an FPGA-only network you could connect any old PC supporting PCIe Gen 3 x16. I had the painful experience to work with Xilinx's ERNIC IP for this purpose in a different project.
Back in Linköping it was not on my radar. We ended up pickin Aurora because it was so simple-stupid, free of charge and we were not sure yet, what the payload data would be exactly.
With close to zero FPGA experience, I spend a good amount of time reading into Xilinx User Guides and documentation - to the point where I was making memes about it.

[]

I went for a VHDL only approach, writing my own top level design, instantiating clk wizards and aurora IP example designs.
The VCU118 has a confusing amount of physical clocking resources attached. So many in fact that when I was quite bamboozled about the lack thereof on the VCU128. 


### AXI Memory Mapped to Stream Mapper (AXI MM2S Mapper)

This IP is as dumb and straight-forward as it can be. So it is easy to implement.
You have an AXI4-MM Slave "input" that 

### Aurora 64b66b

This one is fun.

### Is this the worst blinky ever?

Yes! It is an incredibly over engineered abomination if you inspect it on a CPU architecture level.
We are all familiar with the Von Neumann CPU architecture of one bus for instruction and data (incl. peripherals).
What I just did is: splitting a part of the CPU bus up, converting it to a duplexed data steam, serialize this data stream and send it over a physical cable.

When running the blinky software on the microblaze, I can unplug the QSFP connector and it stops blinking.
The entire CPU is halted until the AXI-Lite write transaction to the GPIO register is completed.
Aurora is robust enough to be hot-plugable. And as soon as the connector is plugged back in, the write transfer is transmitted and the CPU can continue operating.

### A word on latency

Multiple stages of bus translation and hacks each comes at the cost of at leas another clock cycle of latency for the CPU bus access.
Adding an entire serializer / deserializer blows the latency out of proportion. 

### Burst transfers?


