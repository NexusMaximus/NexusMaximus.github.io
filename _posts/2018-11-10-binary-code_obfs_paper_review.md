---
layout: post
title: "Paper review: Binary-Code Obfuscations in Prevalent Packer Tools"
date: 2018-11-10 11:31:00 -0600
categories: notes
---


## Abstract

Security analysts' understanding of malware samples depends on the ability to build high-level analysis products from the raw bytes of program binaries. As such, the first step to analyzing defensive malware samples is understanding what obfuscations are present in these binaries, and how the obfuscations hinder analysis. This paper provides details on those techniques.

Section 2 is all about the methodology and tools used to perform the study. Section 3 is a taxonomy of the obfuscation techniques, along with current approaches to dealing with the techniques. Section 4 provides a summary table of the obfuscation techniques that shows their relative prevalence in real-world malware. 


## Section 2: Methodology

-- to be written, maybe --

## Section 3: Obfuscation techniques

### Binary code extraction
This is the ultimate goal of analysis-- to extract the code from a sample so it can be analyzed. On unpacked/nondefensive binaries, this is trivial, as we simply need to read from the executable portions of the binary file. It gets more challenging with programs that create code or self-modify at runtime. 

In practice, most malware authors create an unpacked executable and then pack it with a packer tool to create a packed version. Most packers can be thought of as elaborations on UPX's packing scheme.

#### How UPX works

UPX (or almost any other packing tool) sets the executable's entry point to the entry point of the bootstrap code that unpacks the payload and then transfers control to the payload's OEP. When the bootstrap code unrolls packed code and data, it places them at the same memory addresses that they occupied in the original binary, so that position-dependant instructions do not move and data accesses find the data in their expected locations. 

UPX also packs the PE's Import Table and Import Address Table data structures. These tables list functions to import from other shared libraries. They are packed because just knowing these functions can reveal significant information about the payload code (and also because the tables are highly amenable to compression). 

#### Building on this technique

The most common elaboration of this basic recipe is one in which portions of the packer's bootstrap code itself are also packed. Most packers use a small unpacking loop to decompress a more sophisticated decompression or decryption algorithm that unpacks the actual payload. For example, ASProtect unpacks 99% of its unpacking code, in addition to the program payload itself. 

#### The X-Ray technique [Perriot and Ferrie 2004]

This technique statically analyzes the program binary with the aim of seeing through the compression and encryption transformation with which the payload code is packed. It leverages statistical properties of packed code to recognize compression algorithms and uses *known cipher-text attacks* to crack weak encryption schemes. Weakness: ineffective against strong encryption and multiple layers of compression or encryption. Further work has been done to improve the technique, but it hasn't yet been shown to work on a broad collection of samples (as of when this paper was written, in 2013). 

#### Identifying written-then-executed code

This technique is used by a host of dynamic analysis tools. They detect and capture unpacked code bytes by tracing the program's execution at a fine granularity and logging memory writes to identify written-then-executed code. Some leverage the Qemu whole-system emulator and the Xen virtual machine monitor respectively, which allows them to observe execution of monitored malware without being easily detected. [2007ish time frame-- Renovo and EtherUnpack] 

There exist unpacker tools that run packed programs either for a certain timeout period, or until they exhibit behavior that could indicate that they are done unpacking. The primary limitations of fine-grained monitoring techniques are that they only identify code bytes that are actually executed and they incur orders of magnitude slowdowns in execution time (those conditional breakpoints tho). This technique is good for antivirus products, but the coarse memory-page granularity means they cannot identify the actual code bytes on unpacked code pages and they cannot identify or capture overwritten code bytes. 


#### Code Overwriting

Self-modifying programs move beyond unpacking by overwriting existing code with new code at runtime. Code overwriting often occurs in small amounts, affecting just a single instruction or even just a single instruction operand. For example, ASPack modifies the push operand of a ``push 0, ret`` instruction sequence at runtime to plug in the OEP address onto the call stack. UPack, however, has a second unpacking loop that overwrites the first one, removing several basic blocks at once from the function currently executing. 

Packers surveyed in this paper only overwrite their own metacode, but it's possible to have more complex overwriting scenarios. Packers generally probably only overwrite their own code in an effort to maintain correctness and to be able to be used in a general case (i.e. for any malicious binary passed to it). An example of more complex unpackers: the MoleBox packer tool and the DarkParanoid virus repeatedly unpack sensitive code into a buffer, so only one buffer-full of the protected code is exposed to the analyst at any given time. This approach is sufficiently hard to implement, however, according to a paper in 2002.

Self-modifying code presents a problem to all approaches of anlaysis because there is no point in time in which all of the program's code is present in the binary. The only way to account for this is to take snapshots of basic blocks (i.e those without jumps in our out) as soon as the basic blocks execute. 


### Disassembly

Once code bytes have been captured, static analysis techniques can accurately disassemble most of the code in compiler-generated program binaries, even when those binaries have been stripped of all symbol information. The general technique employed by disassembly tools is to disassemble the binary code starting from known entry points into the program. 

*Linear-sweep parsing* disassembles sequentially from the beginning of the code section and assumes that the section contains nothing but code. The problem with this is the code section frequently contains non-code bytes such as string data and padding. 

The *recursive-traversal approach* finds instructions by following all statically traversable paths through the program's control flow starting from known function addresses. While more accurate, this technique misses control-transfer targets of instrucions that determine their targets dynamically based on register values and memory contents. 
Binaries stripped of symbol information can still have machine learning techniques applied to them to identify enough function entry points (by recognizing compiler-generated boiler plate code) to help recursive-traversal parsing. 
Of course, the weakness behind this approach is that the malware author would only need to modify the boiler-plate code of the compiler to trick the machine learning algorithms expecting a specific instruction pattern. 
Furthermore, to limit the amount of code that can be found by following control-transfer edges from the entry point (which must be left visible in the binary, or the OS can't begin executing it), one can handwrite extremely irregular assembly code that a parser will have significant trouble interpreting. 

Here are some techniques that hide code, followed by techniques that corrupt analysis with non-code bytes or uncover errors in the disassembler. 

#### Non-returning Calls

A ``call`` instruction is basically a glorified ``jmp`` instruction. Its job is to push a return location onto the stack, and then jump to another section of code. It is intended to be used with a corresponding ``ret`` instruction, which pops the previously-pushed value into a register or area in memory and unconditionally jumps to it. This can be abused by, well, using the instructions in unconventional ways. This causes problems for analysis tools in three ways.

1. Recursive-traversal parsers assume that bytes following a non-returning call represent a valid instruction, because they presume the call will return execution to the area immediately after the call, but it doesn't need to. 
2. The attack breaks an important assumption made by code parsers for identifying function boundaries; namely, that the target of a call belongs to a different function than that of the call itself. There's no reason why you can't use a call to transfer execution somewhere else in the current function. 
3. A binary instrumenter cannot move the call instruction of a call-pop sequence without changing the PC-relative address that the pop stores into a general-purpose register. Moving the call instruction without compensating for this change usually results in incorrect program execution, as packer metacode frequently uses the PC-relative address as a base pointer from which to access data. 

Researchers have proposed removing the assumption that there is valid code at the fall-through address of each call instruction, but doing so drastically reduces the percentage of code that the parser can find, presumably since it can no longer be assumed that any code after a call is actually code. Kruegel et al. compensate by speculatively parsing after call instructions and use a statistical model of real code sequences to determine whether the speculatively parsed instruction sequences are valid. However, this technique was done by researchers in an attempt to just handle one kind of obfuscator, and made assumptions that were valid only for that obfuscator. 

#### Call-stack Tampering

One can semantically generate a ``jmp`` instruction with a ``push [value], ret`` combination, since ``ret`` pops and jumps. ASProtect does this frequently. Furthermore, you could have this series of instructions:
```
pop ebp
inc ebp
push ebp
ret
```

This would jump to an area immediately after a call, offset by +1 bytes, which would totally break the assumption that code **immediately** following a ``call`` is valid, since the valid code is now offset by an amount. 

The authors of this paper implement a technique to deal with this. They apply static analysis techniques to a ``ret`` target prediction, which allows them to improve coverage and accuracy of pre-execution analyses. They use backwards slices at return instructions to determine whether a  ``ret`` jumps to a function's return address as expected or to another address, and falls back on dynamic analysis if static analysis fails. 


#### Obfuscated control-transfer targets

Using indirect versions of the ``call`` and ``jmp`` instructions obfuscates control-transfer targets by using register or memory values to determine their targets at runtime. Furthermore, it provides some space savings, which is significant motivation for small packers. Obfuscation is also a considerable motivating factor as static analysis tools often let these jumps go unresolved. Single indirect jumps can have multiple targets as well over the course of an unpacker's runtime lifetime. 

Static analysis of indirect control-transfer targets is particularly difficult in packed binaries, as they frequently use instructions whose targets depend on register values. Value-set analyses [Balakrishnan and Reps 2004] can theoretically reveal all possible targets for such indirect control-transfer instructions, but this requires knowing all the code in the binary, which is not a satisfiable condition in the realm of packed binaries. Dynamic analysis trivially discovers targets of indirect control transfers that execute, but may leave a significant fraction of code unexecuted. There has been research done on forcing program execution down multiple execution paths [Babic et. al. 2011; Moser et al. 2007] but these are extremely resource-intensive techniques and do not acheive perfect code coverage. 

#### Exception-based control transfers
Signal- and exception-handling mechanisms allow for the creation of obfuscated control transfers whose source instruction and target address are well-hidden from static analysis techniques. Statically identifying sources of control transfers such as these requires predicting which instructions will raise exceptions, which is difficult in theory and practice [Muth and Debray 2000]. This means current disassembly algorithms wil usually parse through fault-raising instructions into what may be non-code bytes that'll never execute. Furthermore, at least on Windows, it is hard to find exception handlers, since they can be registered on the call stack at runtime with no need to perform any Windows API or system calls. 

Using a debugger is the easiest way to detect exception-based control transfers, as it informs the debugger process of any fault-raising instructions and identifies all registered exception handlers whenever a fault occurs. However, use of a debugger interface can be detectd unless extensive precautions are taken. 

#### Ambiguous code and data

Yet another form of control flow obfuscation involves using conditional branches to introduce a fork in the program's control flow with only one path that ever executes, while junk code (fake code bytes or non-code) populates the other path. 

Using current techniques, the only way to identify code with perfect exclusion of data is to disassemble only those instruction sthat appear in an execution trace of a program. 


#### Disassembler Fuzz Testing
Fuzz testing refers to the practice of stress testing a software component by feeding it large quantities of unusual inputs int eh hope of detecting a case that the component hadles incorrectly. By fuzz testing binary-code disassemblers, packer tools can cause the diassembler to misparse instructions or mistake valid instructions for invalid ones. 













