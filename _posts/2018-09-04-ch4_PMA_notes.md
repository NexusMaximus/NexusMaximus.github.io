---
layout: post
title: "PMA Ch. 4 notes: A Crash Course in x86 Disassembly"
date: 2018-09-03 14:31:00 -0600
categories: notes
---


## Chapter summary questions
1. In two sentences or less, provide an overview of what this chapter is about.
    
    This chapter lays the foundation of assembly and computer architecture required for effective malware analysis. 

2. What are the three most important takeaways from this chapter?

	* How the stack works and how functions utilize it
	* What information is contained in which parts of RAM (see the Computer Architecture subsection below)
	* Which registers do what. I presume that's known upon reading this section, but just in case it isn't, this chapter cover sit. 

3. What problems does this chapter address? In other words, why should we care about this chapter?

	Without this information, one won't even have a foundation for being able to understand what 90% of malware analysis is about-- assembly instructions and how they're used. 


## Page-to-page notes

### Levels of Abstraction

There are six levels of abstraction in a computer system. From lowest to highest:

1. *Hardware.* This is the actual circuitry of the system.

2. *Microcode.* This is the firmware of the system. It operates *only* on the exact circuitry for which it was designed.

3. *Machine code.* The machine code level consists of *opcodes*, which are hex digits that tell the processor what to do. Machine code is the result of the compilation process.

4. *Low-level languages.* Assembly and the like is what is found here, and is generally accepted as the lowest-level human-readable abstraction level. There are several dialects of assembly, among which include x86, which is what is taught in this book. **Assembly is the highest level language that can be reliably and consistently recovered from machine code when the high level source code is unavailable (as it usually is.)**

5. *High-level languages.* Things like C, C++, etc. 

6. *Interpreted languages.* Languages like Java, C#, Perl, .NET. Interpreted languages are not compiled but are instead translated into *bytecode*, which is an intermediate representation specific to the programming language. It executes within an *interpreter*, which translates bytecode into executable machine code on the fly at runtime. 

### Computer Architecture

* RAM contains the Stack, Heap, Code, and Data of a program.

	* *Data*. Contains the data section, which contain values put in place when a program is initially loaded. These are static values (opposite of dynamic values which are contained within the heap), or they are global values since they're available to any part of the program. 
	* *Code.* Instructions for the CPU are here.
	* *Heap.* Used for dynamic memory during program execution. 
	* *Stack.* Used for local variables and parameters for functions, and to help control program flow. More on this later. 

* *Operands* take one of three forms. 
  * *Immediates*: fixed values. Ex: ``0x42``
  * *Registers*: refer to registers. Self-explanatory. Ex: ``eax, esp, ah``
  * *Memory addresses*: refer to memory addresses which in turn could point to locations in memory or registers. Ex: ``[eax]``. 

* *Registers* are small amounts of data storage available to the CPU. We have four different kinds of registers.
  * *General* registers are used by the CPU during execution.
  * *Segment* registers are used to track sections of memory.
  * *Status flags* are used to make decisions and are modified by certain instructions.
  * *Instruction pointers* point to the next instruction to execute. 


### Notes on specific assembly instructions
The book uses Intel syntax, which follows a ``dest, src`` notation. 

* LEA *dest, src* is used as a quick way to calculate an effective address. For example, ``lea eax, [ebx + 8]`` puts the value specified within ebx, +8, into eax. 

* SUB will subtract into the destination register. 

* MUL and DIV always operate on EAX. 
  * ``MUL *value*`` will calculate EDX:EAX = EAX * value
  * ``DIV *value*`` will calculate EDX:EAX (treating the two as a compound register) by value, and store the quotient (division) in eax, and the remainder (modulus) in edx. 

* XOR is a common instruction used to zero things out. ``XOR eax, eax`` => eax is 0 now. 

* TEST and CMP are used to set flags. They do not affect operands. See p80 for a little table that helps make sense of what cmp sets under what conditions. 

* The REP family of instructions are used for manipulating data buffers. I won't bother with understanding these in too much depth at the moment, as it's something that can be easily looked up when I'm face-to-face with a rep instruction later on. See page 81. 



### Stack Details
It's very important to know how the stack is used to conduct function calls, as it is frequently abused. 

1. Arguments are ``push``ed onto the stack.
2. A ``call [functionName]`` occurs. FunctionName is simply a pointer to some code. 
  * ``call`` is a compound instruction-- that is, it is an instruction that combines two others. In this case, those two instructions are ``push eip`` followed by ``mov eip, functionName``. In this manner, the return address is saved onto the stack. 
3. Function prologue stuff (i.e. boiler plate stuff) happens now. Space is allocated on the stack for local variables, and ``ebp`` is pushed onto the stack. Pushing ``ebp`` 'saves the context' of the stack as it was before the function was called, which is important as we need that context back after the function completes. 
4. The function does its work.
5. Now, the stack is restored to the state in which it was before the function was called, via a function epilogue. ``ebp`` is restored to what it was. ``leave`` is often used to restore the stack as ``leave`` does a ``mov esp, ebp`` followed by a ``pop ebp``, which is all that needs to happen to set the stack back to what it was-- as long as things weren't modified by the function. 
6. The function returns by calling ``ret``. ``ret`` does a ``pop eip``, which transfers control back to the calling function. 
7. The stack is adjusted to remove arguments that were sent, unless they'll be sent again later. 

Diagrams are great for visualing the stack. Due to fear of copyright stuff, I'll just tell you that a great diagram can be found on p79. 

Note that the stack grows upward (i.e. as things are pushed onto the stack, memory addresses of 'higher' items are lower in value than memory addresses of 'lower' items. 

Finally, note that function *arguments* are going to be at a positive offset relative to EBP, while *local variables* are going to be at a negative offset relative to EBP. 

