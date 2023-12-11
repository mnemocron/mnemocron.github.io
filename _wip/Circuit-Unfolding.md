---
layout: post
title: Circuit Unfolding
subtitle: How to parallelize recursive DSP functions
gh-repo: mnemocron/my-discrete-fpga
gh-badge: [star, follow]
tags: [theory, dsp, vhdl]
comments: true
author: Simon
---

# WIP

![https://mnemocron.github.io/assets/img/unfolding/.png](https://mnemocron.github.io/assets/img/unfolding/.png){: .mx-auto.d-block :}
**Fig 1:** _Example of a 2-way unfolded (2x polyphasic) IIR filter processing two samples per clock cycle._

We are all familiar with the growing demand in computaional power in DSP applications. Radio frequency applications require that several gigasamples are processed on an FPGA that can be clocked at maximum of several hundred of MHz.
This facilitates the need to use massive parallelization of DSP algorithms.
For _non-recursive_ algorithms, like FIR filters, this is a no-brainer. See the bottom of this article for a demonstration.
The main focus shall be on _recursive_ algorithms. The theory behind circuit unfolding can be applied to any FPGA algorithm with internal state variables - not just IIR filters.

A bit of an introduction first. The theory I am about to present is a sumary of a digital design course I was so lucky to attend during my exchange semester at the Linköpings Universitet in Sweden.
With a background in applied sciences working on real industry problems, it was refreshing to see a new approach to digital design.
Just as an example: The institute in Sweden built a demonstration of a 100 GSps FFT on FPGA while colleagues at the school for applied sciences built an FFT design at 2 GSps that was actually used in industrial printers later. Both working on cutting edge performance but with a different goal in mind.

The course I took provided me with many valuable tools that I can throw at FPGA algorithms.
One of which is _Circuit Unfolding_ which has an intuitive and a systematic approach to it.

### Motivational Example

Let us start with a very basic recursive algorithm, the IIR filter with a single tap in _Fig. 2_.

![https://mnemocron.github.io/assets/img/unfolding/simple-iir.png](https://mnemocron.github.io/assets/img/unfolding/simple-iir.png){: .mx-auto.d-block :}
**Fig 2:** _Simple, single tap IIR filter._

This can be unfolded like in _Fig. 3_. the details behind unfolding are explained further below.

![https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded.png](https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded.png){: .mx-auto.d-block :}
**Fig 3:** _IIR filter unfolded to a 2-way parallel circuit which process 2 samples per clock cycle._

As you can see, the unfolded circuit can now process 2 samples per clock cycle. This however, comes at the cost of a much greater critical path.
What was before a single multiplication and addition is now twice as much - without a single pipeline stage (!). 
This will seriously impact the f-max of the circuit.
The mathematical operations can be split and rearanged to match the circuit in _Fig. 4_.

![https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded-expand.png](https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded-expand.png){: .mx-auto.d-block :}
**Fig 4:** _Unfolded IIR filter with expanded operations._

Here we still have a longer critical path than the original IIR filter. Using two additions (or a ternary addition) which is not supported by a DSP48 slice.
It will still result in worse timing than the non-unfolded circuit.
Now however, some operations are in the forward path without recursive elements and can be pipelined as in _Fig. 5_.

![https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded-expand-pipeline.png](https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded-expand-pipeline.png){: .mx-auto.d-block :}
**Fig 5:** _Final 2x unfolded, rearanged and pipelined IIR filter._

Theoretically, we now have the same critical path as the single IIR filter but it will process twice the samples per clock cycles.

Sidenote: the true critital path may vary. The single IIR filter could be implemented in a single DSP48 instance. The second multiplier with two subsequent additions will not fit into a DSP48 slice and may require either another DSP48 slice or an adder chain in LUT fabric.

---

### Circuit Unfolding Theory




| from block `i` | using `L` delays | to block `j` | using `K` delays |
|:----|:----|:----|:----|

| `i` | `L` | `j` | `K` |
|:----|:----|:----|:----|
| `0` | `1` | `1` | `0` |
| `0` | `2` | `0` | `1` |
| `0` | `1` | `0` | `1` |
| `0` | `2` | `1` | `1` |







