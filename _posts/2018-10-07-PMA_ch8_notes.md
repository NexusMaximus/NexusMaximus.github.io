---
layout: post
title: "PMA Ch8 Notes: Debuggers"
date: 2018-10-07 12:02:00 -0600
categories: notes
---


## Chapter summary questions
1. In two sentences or less, provide an overview of what this chapter is about.

	This chapter provides an overview of what debuggers are, what breakpoints are and how they work, and what exceptions are. They cover just the conceptual topics of debuggers, meaning no technical information is provided. 

2. What are the three most important takeaways from this chapter?
  
	* There are several types of breakpoints and it pays to know how they work so you can be aware of potential reprecussions of using them. 
	* Debuggers can be used to modify program execution by modifying the control registers, which can be helpful for identifying what code *would* have done if it detected it was in a different environment.
	* Debuggers can be used to just run single functions as well, which may cause the program to crash afterwards but can be very useful for understanding exactly how, say, a decoding routine works. 

3. What problems does this chapter address? In other words, why should we care about this chapter?

 	This chapter provides the foundation for understanding what debuggers do and how they work, which enables more technical discussion of how to use debuggers, which is information covered in the next two chapters. 




## Terminology
*Debugger*: a piece of software or hardware used to test or examine the *execution* of another program. 
  * Malware analysis typically makes use of *assembly-level* debuggers, which run an instruction at a time, as opposed to source-level debuggers, which run a line of source code at a time. (We don't have the source code, so...)

*Breakpoints* are used to pause execution of a program. 

*Step-into vs. step-over*: stepping over means it'll execute all the code represented by a single instruction. In other words, if you step over a function, it'll call that function and break immediately after the function returns. Stepping into means you'll break at the first instruction within a function. 
  * For instructions that don't transfer execution, stepping into and stepping over are functionally equivalent. 
  * Some debuggers have a step-out function as well-- they will run until the function you're currently in returns. 

## Types of breakpoints (and how they work!)

### Software breakpionts
When a software breakpoint is set, the debugger overwrites the first byte of the broken instruction with ``0xCC``, which is the instruction for INT 3, which is the breakpoint interrupt designed for use with debuggers. When this is executed, the OS generates an exception and transfers control to the debugger. 
  * One problem with this is the code is actually being changed in memory. Functions that perform integrity checks will notice the existence of a breakpoint. Self-modifying code can also remove the breakpoint. Code that reads in the memory of a function will read in a 0xCC byte instead of whatever byte is typically there. 

### Hardware breakpoints
Hardware execution breakpoints are supported via dedicated hardware registers on the x86 architecture. Every time the processor executes an instruction, there is hardware to detect if the instruction pointer is equal to the breakpoint address. As such, the problems discussed with software breakpoints are nonexistent. 

Furthermore, these breakpoints can be set to break on access rather than execution. If code reads memory from a specific address, a hardware breakpoint can be used to break the program on the instruction that reads the memory. 
  * The drawback with hardware breakpoints is there are only four hardware registers that store breakpoint addresses. Additionally, they are easy to modify by the running program. 
    * This modification can be partially remediated, however, by setting the General Detect flag in the DR7 register. This causes a break to occur whenever a ``mov`` instruction accesses a debug register. 
    * See more info about specific debug registers on page 175. 

### Conditional breakpoints
These are software breakpoints that will break only if a certain condition is true. This would be helpful for breaking when functions are called with certain arguments, for example. They are evaluated as software breakpoints, and the debugger evaluates the condition. 

  * One problem with these is they slow down the execution speed of a program dramatically, because the program state is examined and evaluated in addition to the actual execution of the program. 

## Exceptions
Debuggers are usually given two opportunities to handle the same exception: a *first-chance exception* and a *second-chance* exception. 
  * When *First-chance exceptions* occur, the program stops executing and the debugger is given a first chance at control. The debugger can handle the exception or pass it to the program. If the program has a registered exception handler, that is given a chance to handle the exception after the debugger's first chance. 
  * If the program does not resolve the exception, then the debugger is given a second chance to handle it-- *second-chance exception*. When the debugger receives these, it means the program would have otherwise crashed, and the debugger must resolve the error if the program is to continue. 
      * As analysts, we don't typically care about first-chance exceptions since it's common that they are used to direct control flow. However, we do care about second-chance exceptions. There may be bugs in the malware, or it's possible that the malware does not like the environment in which it's running. 


