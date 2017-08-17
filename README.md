# CAPI SNAP Development for Programmers

## Introduction

IBM [CAPI SNAP](https://github.com/open-power/snap) is an open source framework targeted at making FPGA-acceleration on IBM POWER servers as easy as possible. This book wants to help developers getting started by providing an A-to-Z explanation on how to create an acceleration example in SNAP. It includes setting up the environment, building and simulating the the given examples, creating one by yourself and evaluating it against software. If you have access to an IBM POWER System with a Nallatech 250S \(or another compatible\) FPGA-card you will also be able to execute examples on the real hardware.

<img src="/assets/nallatech250s.jpg" alt="Nallatech 250S FPGA" style="max-height: 300px;">
<p class="figure-caption">Nallatech 250S FPGA</p>

### Why was this written?

When making first steps with [IBM CAPI](https://developer.ibm.com/linuxonpower/capi/), which is the basis for SNAP, but also requires in-depth understanding of hardware development, the blog seriers ["Tinkering with CAPI"](http://suchprogramming.com/tinkering-with-capi/) by Kenneth Wilke helped us immensely. As there was no equivalent for SNAP, we thought writing up our acquired knowledge could help other people get started. It also serves as a report for a corresponding university project and therefore contains information in different levels of detail. Feel free to just pick out what you need.

### Who wrote it?

The authors of this book are

* Balthasar Martin &lt;balthasar.martin@student.hpi.de&gt;
* Robert Schmid &lt;robert.schmid@student.hpi.de&gt;
* Lukas Wenzel &lt;lukas.wenzel@student.hpi.de&gt;

who are students at the Hasso-Plattner-Institute in Potsdam and constituted the masters project _"Heterogeneous Computing: acceleration and FPGAs in context of POWER and CAPI"_ . The project was offered at the chair for [Operating Systems and Middleware](https://hpi.de/en/research/research-groups/operating-systems-and-middleware.html) in cooperation with IBM and was supervised by

* Prof. Dr. Andreas Polze &lt;andreas.polze@hpi.de&gt;
* Felix Eberhardt &lt;felix.eberhardt@hpi.de&gt;
* Max Plauth &lt;max.plauth@hpi.de&gt;.

We also want to thank all the people at IBM who held close contact and helped us during the project, including but not limited to

* Joerg-Stephan Vogt &lt;jsvogt@de.ibm.com&gt;
* Sven Boekholt &lt;boekholt@de.ibm.com&gt;
* Frank Haverkamp &lt;haver@linux.vnet.ibm.com&gt;
* Thomas Fuchs
* Bruno Mesnet
* Bruce Wile



