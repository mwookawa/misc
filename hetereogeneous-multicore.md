Notes on Heterogeneous Multicore Architectures and The Barrelfish Multikernel OS 3/31/17
========================

In the synthesis group we talked briefly about dynamically added and removing processing cores without halting the system. I was trying to remember whether I'd see this as a research prompt on the ETH site or..

in fact, it turns out that the OSDI14 barrelfish paper was on adding this functionality to barrelfish and more: cores can be fused and split, kernels on a specific core can be patched without halting dependent processes, and more.

https://people.inf.ethz.ch/troscoe/pubs/osdi14_zellweger.pdf

-------------------------------------

two more notes brought up in the paper and above:

**live patching in native runtime systems** has been approached previously at userlevel, most notably by Iulian Neamtiu's Ginseng. Reductively speaking, Ginseng's technique finds safe execution points in which to flip function pointers from one implementation to another within the same virtual address space:

http://www.cs.umd.edu/~mwh/papers/ginsengMT.pdf
http://www.cs.umd.edu/~mwh/papers/mutatis-journal.pdf

Ginseng's safety requirements and the layers of abstraction required to do the safety reasoning prevent implementation at the kernel layer.

**dark silicon, code-dependent heterogeneous MP:** one mode of heterogeneous computing that has been looked at in the past few years has been the notion that processing dies are shrinking much faster than switching impedances are dropping; as a result, very little processor die can be active at any given time. hence, as silicon wafer fabrication nodes become smaller, an drastically increasing percentage of the fabricated processor die must be 'dark', or inactive per unit time.

Consequently, there has a wide debate on what to lithograph onto silicon that will largely be dark. Generally, people agree that more cores should be placed on a die. However, these cores can (and maybe must) be much more specialized than the primary or "application" core. 

One approach to this is the wildly successful big.LITTLE approach, due to ARM, which places a very small power efficient core on the same die as a very large high performance (eg, SIMD or MP) core. In this model, the LITTLE core does serialized background tasks without interaction with the big core. The large core then only activates for parallel, compute-heavy tasks. (note that modern "turbo boost" mechanisms are a single-core form of this, in which functional on-core units such as SSE2/AVX, ALU and FPUs are clocked-down or gated until absolutely needed)

Another approach, which may prove relevant to PRINCESS some day, is to synthesize "code-specific" cores that are akin to ASICs for hotspots in the code expected to execute on a core. This approach was reified by a project called GreenDroid:

http://cseweb.ucsd.edu/~swanson/papers/Micro2011QASICS.pdf

One major, if not primary limitation of the greendroid approach is that the cores are currently tightly tied to the code running on them. If the code running changes or the "Qcores" themselves change to optimize a new common-case codebase, the Qcores will likely never be activated, making them truly dark. 

In some sense, this is an exaggeration of the current problem of changing ISAs: if your ISA is a set of overfit instructions that each execute a module of your application in one clock, your ISA becomes useless when you modify your application (and vice versa!).

If instead we could synthesize kernels that ran on these hyper-specialized cores and "understood" the computations they could perform efficiently, and those they could not, processor dies designed in this manner could be re-purposed to new software, and software written in this manner could be repurposed to new processors.
