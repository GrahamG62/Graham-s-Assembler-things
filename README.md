# Graham-s-Assembler-things
This is a library that I wrote back in 2001, using the Herculers Emulator whilst between jobs. It contains a savearea stack manager, a descent recursive parser and a disassembler plus associated macros and examples. My goal was to make structured IBM assembler easy to write and use.
Latterly, I've used it to write an enclave SRB to handle zIIP offload. That it has never caused a customer problem, I shall take that as a win. 

Documentation:
The library is composed of some assembler macros to provide an easy way to access the service code and the routines that support them. I'll start with the macros, and then expand on them.

STKR
This is the macro that invoves the stack controller CZLSTK. It allows for a local workarea (bracketed by {{...}} containing local variables to a routine or subroutine) and creates a savearea, plus workarea stack that can grow or contract depending on call depth. When required, it will extend the stack in 4K frames. When finished with, it will free up all stack frames and return to the original caller savearea. The get/free storage routines are configurable - CZLSTG & CZLSTF are examples of how to do this.
Additionally, it can reserve an area to be used as an anchor block and persist this throughout a module. The macro has comments as to how to do this. 

In order to test this, I added a couple of routines that used this mechanism - CZLPAR, CZLMSG & CZLDIS. They all use the STKR/CZLSTK things to create & maintain re-entrancy via the stack with helper macros SUB & SUBR for easier subroutines. CZLENT is an example of how to initialise the environment and call other modules. There's a collection of helper macros starting with MAC* which do things like generate parameter lists and set register values before returning to a calling program, or pick up a register value from a savearea.

CZLTSTO is the original test bootstrap - it calls the disassembler and prints out the disassembled code (current when I wrote it for OS/390) to SYSOUT or whatever. CZLDIS (the disassembler) is pretty compact and uses the OPCD macro in CZLOPC to define an instruction with its format and type. KEYTZ is an example of how the CZLPAR routine is called and works. The KEYW and KEYOP macros define a keyword table - the example here is CZLKEY.

The message formatter CZLMSG uses the ZMSGR and sometimes the ZMSG macros to either issue, or define messages. Messages can be in an assembler routine (as in CZLDIS) or in a message table using ZMSG. By default, it calls a routine to direct the message text to another routine that can be used for national language support, in this case one in UK English. As it stands, messages can either be directed to the system console, or returned to the calling program. A quick example would be:

czlmuk   rsect ,

czlmuk   amode 31

czlmuk   rmode any

zmsg 001,I,'CZL - Hello World!'

zmsg type=end
 
You would call that by: 

         zmsgr mod=TST,                                                +
               num=1,                                                  +
               out=yes,                                                +
               mf=(e,msgpl)

The msgpl parameter list is built using ZMSGR TYPE=PLIST (lower-case works too - I like to not have the shift key under a finger when I'm writing assembler, so bear in mind that constant values *are* case sensitive when interrogated in macro code).

That's the complicated (!) stuff, the other macros should be easy enough to recognise. I like the HLASM technique where you define a bit setting as: 

msflg1   ds    xl1      04+30      * Flag byte #1
msf1ret  equ   msflg1,x'80'        *   Format & return message
msf1con  equ   msflg1,x'40'        *   Issue message on console
msf1sup  equ   msflg1,x'20'        *   Suppress blanks/x'00's
msf1lbr  equ   msflg1,x'10'        *   Use Linkage=BRANCH on WTO

So you can define a bit setting as a length attribute on the flag you are interested. The CZL* code uses this throughout, with the SET, TEST, etc. macros because you just need to refer to the flag without worrying which byte you are referring to. 

For example: 
	*      *--------------------------------------------------------------*
	*      * Return the message to caller's area if requested             *
	*      *--------------------------------------------------------------*
			 test  msf1ret             * Return message ?
			 bno   main010             * No, skip
			 zcall =a(retmsg)          * Return message


ASMCZL assembles & links things and can be adapted as required. 
