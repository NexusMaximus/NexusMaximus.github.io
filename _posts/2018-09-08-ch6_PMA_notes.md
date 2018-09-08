---
layout: post
title: "PMA Ch. 6 notes: Recognizing C Code Constructs in Assembly"
date: 2018-09-08 14:31:00 -0600
categories: notes
---


## Chapter summary questions
1. In two sentences or less, provide an overview of what this chapter is about.

	This chapter introduces how some common C code constructs look when implemented in assembly. Being able to understand what these constructs looks like is extremely helpful when analyzing malware as it'll provide insight into the general functionality of the malware, more quickly.

2. What are the three most important takeaways from this chapter?

	Practice is the only thing that will help a new malware analyst recognize general semantics of a program. 


3. What problems does this chapter address? In other words, why should we care about this chapter?

	This chapter provides exercises to quickly speed up analysis of executables by providing a foundation for understanding what common code constructs *generally* look in assembly. It is, however, no substitute for practicing. Labs coming soon. 


## Terminology
**code construct**: a code abstraction level that defines a functional property, but not the details of its implementation. 
	* if statements, for loops, switches, linked lists, etc
	

## Page-to-page notes
* This is the part of malware analysis in which it becomes easy to get bogged down with weeds. Keeping the overall picture in mind and not getting infatuated with unnecesary details is very important. 


### Global vs Local variables
* Global variables are referenced by memory address.
* Local variables are referenced relative to the stack. 

### Different Calling Conventions

#### cdecl
* Parameters pushed ontothe stack from right to left. 
* Caller cleans up stack when function is complete.
* Return value stored in ``eax.``

#### stdcall
* Similar to cdecl, except that stdcall requires the *callee* to clean up the stack when the function is complete.
* Windows API uses stdcall by convention. 
  * Any code calling API functions will therefore not need to clean up the stack, since that's the responsibility of the DLL that implement code for the API functions.

#### fastcall
* The first few arguments (typically two) are passed in via registers (usually ``edx`` and ``ecx``). Additional arguments are loaded from right to left. 
* Calling function is usually responsible for cleaning up the stack, if necessary. 

**In general, you won't know what your malware was compiled with, so you should know the concepts behind why the calling conventions work, so you can recognize the semantics of any calling convention.**

### Notes on specific constructs

#### Switch statements
Switches are typically implemented in one of two ways: the if-style, or jump tables.

* If style: each case statement is compared to the switch conditional variable. Typically there are lots of jz's and cmp's in these.

* Jump table style: Typically seen with large, contiguous switch statements, the compiler optimizes away lots of the comparisons. In this case, there is a lot of ``dd offset loc_abcd`` statements following each other, and the switch variable is used as an index into that table to identify which block of code to hop to (i.e. which case to evaluate). 


#### Arrays
Ararys are accessed using a base address as a starting point. The size of each element is not always obious, but it can be determined by seeing how the array is being indexed. 









