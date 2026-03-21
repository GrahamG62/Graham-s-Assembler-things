# Graham-s-Assembler-things
This is a library that I wrote back in 2001, using the Herculers Emulator whilst between jobs. It contains a savearea stack manager, a descent recursive parser and a disassembler plus associated macros and examples. My goal was to make structured IBM assembler easy to write and use.
Latterly, I've used it to write an enclave SRB to handle zIIP offload. That it has never caused a customer problem, I shall take that as a win. 

Documentation:
The library is composed of some assembler macros to provide an easy way to access the service code and the routines that support them. I'll start with the macros, and then expand on them.

STKR
This is the macro that invoves the stack controller CZLSTK. It allows for a local workarea (bracketed by {{...}} containing local variables to a routine or subroutine) and creates a savearea, plus workarea stack that can grow or contract depending on call depth. When required, it will extend the stack in 4K frames. When finished with, it will free up all stack frames and return to the orioginal caller savearea. The get/free storage routines are configurable - CZLSTG & CZLSTF are examples of how to do this.
Additionally, it can reserve an area to be used as an anchor block and persist this throughout a module. The macro is commentes as to purpose. 

In order to test this, I added a couple of routines that used this mechanism - CZLPAR, CZLMSG & CZLDIS. They all use the STKR/CZLSTK things to create & maintain re-entrancy via the stack with helper macros SUB & SUBR for easier subroutines. CZLENT is an example of how to initialise the environment and call other modules. There's a collection of helper macros starting with MAC* which do things like generate parameter lists and set register values before returning to a calling program, or pick up a register value from a savearea.

