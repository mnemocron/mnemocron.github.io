---
layout: post
title: Circuit Unfolding
subtitle: How to parallelize recursive DSP functions
gh-badge: [star, follow]
tags: [theory, dsp, vhdl]
comments: true
author: Simon
---

![https://mnemocron.github.io/assets/img/unfolding/5tap-iir-2phase-unfolded.png](https://mnemocron.github.io/assets/img/unfolding/5tap-iir-2phase-unfolded.png){: .mx-auto.d-block :}

We are all familiar with the growing demand in computaional power in DSP applications. Radio frequency applications require processing of several gigasamples on an FPGA that can only be clocked at a maximum of several hundred of MHz.
This facilitates the need to use massive parallelization of DSP algorithms - a so called **multi rate** system. 
For _non-recursive_ algorithms like FIR filters this is a trivial no-brainer. See the end of this article for a demonstration.
The main focus here is on _recursive_ algorithms like IIR filters. 
Note that the theory behind **circuit unfolding** can be applied to any FPGA algorithm with internal state variables - not just IIR filters.

A bit of backstory (I am a ~~food~~ _tech_ blogger and you will read about my travel diary before I actually show the ~~recipe~~ _tutorial_).
The theory I am about to present is a summary of a digital design course I was lucky to have attended during my exchange semester at the Linköpings Universitet in Sweden (TSTE87).
For my experience in applied sciences, working on real industry problems, it was refreshing to see a more theoretical approach to digital design.

> An anecdote: The institute in Linköping built a demonstration of a 100 GSps FFT on FPGA while my colleagues at the school for applied sciences in Switzerland built an FFT design at 2 GSps that was actually used in industrial printers. 
Both teams are working on cutting edge performance but each with a different goal in mind.

The afformentioned class provided me with many valuable tools that I can throw at FPGA algorithms.
One of which is _Circuit Unfolding_ which has an intuitive and a systematic approach to it.

### Motivational Example: Single Tap IIR

Let us start with a very basic recursive algorithm, the IIR filter with a single tap in _Fig. 2_.
Note that it has a **critical path** of one multiplication and one addition. Since loops cannot be pipelined, this is the path that defines the timing for the maximum achievable clock frequency.

![https://mnemocron.github.io/assets/img/unfolding/simple-iir.png](https://mnemocron.github.io/assets/img/unfolding/simple-iir.png){: .mx-auto.d-block :}
**Fig 2:** _Simple, single tap IIR filter._

An application may demand that multiple samples (e.g. `N=2`) are processed per clock cycle.
All of the processing elements are doubled and the challenge at hand is managing the feedback path (memory elements).
The IIR tap can be unfolded as presented in _Fig. 3_. 

![https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded.png](https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded.png){: .mx-auto.d-block :}
**Fig 3:** _IIR filter unfolded to a 2-way parallel circuit which process 2 samples per clock cycle._

This however, comes at the cost of a much greater critical path.
What was before a single multiplication and addition is now twice as much - without a single pipeline stage (!). 
This will seriously degrade the maximum achievable clock frequency of the circuit.

To improve the timing again the mathematical operations can be split and rearanged to match the circuit in _Fig. 4_.

![https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded-expand.png](https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded-expand.png){: .mx-auto.d-block :}
**Fig 4:** _Unfolded IIR filter with expanded operations._

Here we still have a critical path that is longer than the original IIR filter. Futhermore there are now two additions in series (or a ternary addition) which may not map efficiently to a given FPGA architecture.
It will still result in worse timing than the original circuit.
Now however, some operations are in the forward path without recursive elements. Those can now be pipelined as depicted in _Fig. 5_.

![https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded-expand-pipeline.png](https://mnemocron.github.io/assets/img/unfolding/simple-iir-unfolded-expand-pipeline.png){: .mx-auto.d-block :}
**Fig 5:** _Final 2x unfolded, rearanged and pipelined IIR filter._

Theoretically, we now have the same critical path as the single IIR filter but now it will process twice the samples per clock cycle.
This should be motivation enough to propose a general approach to unfolding recursive digital logic.

> Sidenote: the true critital path may vary. The single IIR filter could be implemented in a single DSP48 instance. The second multiplier with two subsequent additions will not fit into a DSP48 slice and may require either an additional DSP48 slice or an adder chain in LUT fabric. 

---

### Circuit Unfolding Theory

In practice, algorithms consist of many more internal state signals (memory!). 
It is not practical to perform the above calculations by hand.
There is a systematic way to perform unfolding on circuits.

- $$M$$ Unfolding factor (i.e. oversampling factor)
- $$N_i$$ parallel computational block $$i$$ (where $$i={0,1,2...M-1}$$)

Therefore, there are $$M$$ blocks denoted $$N_i$$. The input samples into block $$N_i$$ are $$x[Mn+i]$$ and the output samples of block $$N_i$$ are $$y[Mn+i]$$.

- $$L$$ previous (non-unfolded) internal delay elements
- $$K$$ unfolded delay elements

An internal state from block $$N_i$$ with $$L$$ delay elements is connected to block $$N_j$$ with $$K$$ delay elements.
With the equations being:

$$ j=(i+L)\,\mathrm{mod}\,M $$

$$ K = \left\lfloor{ \frac{i+L}{M} } \right\rfloor $$

--- 

### Applied Examples
#### 2-tap IIR

Let us consider a 2-tap IIR filter in _Fig. 6_.

![https://mnemocron.github.io/assets/img/unfolding/2tap-iir.png](https://mnemocron.github.io/assets/img/unfolding/2tap-iir.png){: .mx-auto.d-block :}
**Fig 6:** _2-tap IIR filter._

We are only interested in the memory elements in the feedback path.
Therefore the 2-tap IIR filter can be abstracted

![https://mnemocron.github.io/assets/img/unfolding/2tap-iir-abstract.png](https://mnemocron.github.io/assets/img/unfolding/2tap-iir-abstract.png){: .mx-auto.d-block :}
**Fig 7:** _Abstract view of a 2-tap IIR filter._

The IIR filter shall be unfolded by a factor of 2.
It is therefore copied and each IIR instance shall process every other sample.
The instances are denoted `N0` and `N1`.

![https://mnemocron.github.io/assets/img/unfolding/2tap-iir-2phase-abstract.png](https://mnemocron.github.io/assets/img/unfolding/2tap-iir-2phase-abstract.png){: .mx-auto.d-block :}
**Fig 8:** _Abstract view of two 2-tap IIR filters._

Now the systematic equations can be applied and will result in the following table:

| **`i`** | **`L`** | **`j`** | **`K`** |
|:----|:----|:----|:----|
| `0` | `1` | `1` | `0` |
| `0` | `2` | `0` | `1` |
| `1` | `1` | `0` | `1` |
| `1` | `2` | `1` | `1` |

For example, the first row reads as follows:

_"Connect the signal of block `N0` which had `1` delay element to block `N1` using `0` delay elements."_

If done for every signal, the resulting circuit will look like this:

![https://mnemocron.github.io/assets/img/unfolding/2tap-iir-2phase-unfolded.png](https://mnemocron.github.io/assets/img/unfolding/2tap-iir-2phase-unfolded.png){: .mx-auto.d-block :}
**Fig 9:** _Fully unfolded 2-tap IIR filter._

#### 5-tap IIR

Another example for good measure.
Take a 5-tap IIR with more internal delay elements.

![https://mnemocron.github.io/assets/img/unfolding/5tap-iir-abstract.png](https://mnemocron.github.io/assets/img/unfolding/5tap-iir-abstract.png){: .mx-auto.d-block :}
**Fig 7:** _Abstract view of a 5-tap IIR filter._

| **`i`** | **`L`** | **`j`** | **`K`** |
|:----|:----|:----|:----|
| `0` | `5` | `1` | `2` |
| `0` | `4` | `0` | `2` |
| `0` | `3` | `1` | `1` |
| `0` | `2` | `0` | `1` |
| `0` | `1` | `1` | `0` |
| `1` | `5` | `0` | `3` |
| `1` | `4` | `1` | `2` |
| `1` | `3` | `0` | `2` |
| `1` | `2` | `1` | `1` |
| `1` | `1` | `0` | `1` |


![https://mnemocron.github.io/assets/img/unfolding/5tap-iir-2phase-unfolded.png](https://mnemocron.github.io/assets/img/unfolding/5tap-iir-2phase-unfolded.png){: .mx-auto.d-block :}
**Fig 11:** _Fully unfolded 5-tap IIR filter._

