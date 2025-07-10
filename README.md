While studying electronics, particularly computer microarchitecture, I came to understand how closely linked 
the functioning of hardware and software is.

This document explains the function of an integrated circuit that I designed myself after studying the IEEE-754 standard as well as 
floating point units (FPUs) and SIMD units.

The circuit diagram does not follow any specific order of magnitude.
I am not a professional, so it is possible that the circuit is not optimal in terms of its architecture or due to certain 
physical limitations that may not have been taken in count.

The diagram is based on NMOS and PMOS transistors (rather than high-level logic gate diagrams) so that the circuit is closer to reality in its 
architecture, mainly to highlight the compromises I had to make.
Furthermore, most of the circuits are CMOS technology, but for reasons of compromise, some circuits 
are transmission circuits.

My primary intention with this project is to make the complexity of the algorithms behind our computers, as well as our favorite software and video games, more concrete (more visual).

// to update.
The file is divided into several sections, as follows:
- i. reminder of the IEEE-754 standard
- ii. the problem solved by the circuit
