---
layout: post
title: "PMA Ch.9 Notes: OllyDbg"
date: 2018-10-12 14:31:00 -0600
categories: notes
---


## Chapter summary questions
1. In two sentences or less, provide an overview of what this chapter is about.

	This chapter covers how to use many features that OllyDbg provides.

2. What are the three most important takeaways from this chapter?
	
	For this chapter, it's hard to choose three *most* important takeaways, simply because all the information inside the chapter is helpful in some form of malware analysis scenario. 

3. What problems does this chapter address? In other words, why should we care about this chapter?

	We should care about this chapter as it gives us the introducton to one of the most powerful and popular tools in the malware analyst's toolbox. 


## Using OllyDbg

### Opening a file
EZ. **File -> open.** The only time in which you can pass in command line arguments is at this time. 

You can also attach OllyDbg to a running process using **File -> attach**. When you do this, OllyDbg will foist itself upon the running process and break all threads. It'll display the current executing thread's code. This may be windows API code, so you should have the program break upon access to the entire code section. 


### Understanding Olly's GUI

![n1.PNG](/assets/images/posts/ch9sc/n1.PNG)

Upon opening a program with OllyDbg (in my case, Lab09-01.exe), we have four different windows. 

#### Top left: the disassembler window
This is the disassember window. It shows the debugged program's code-- the current instruction pointer with several instructions before and after it. To modify instructions or data (or add new instructions), press spacebar within this window. 


#### Top right: registers window
You guessed it-- it shows registers. They will change color as the program executes. Red is to highlight a recently changed value. You can right click any register value and select **modify** to change any register value. 

#### Bottom right: stack window
This shows the stack of the program. It'll always show the top of the stack for a given thread. You can manipulate it in the same way as you can manipulate registers-- right clicking a spot on the stack and selecting **modify**. 


#### Bottom left: memory dump window
This window shows a dump of live memory for the debugged process. Pressing **ctrl+G** in this window permits you to go to a specific spot in memory. Clicking a memory address and selecting **Follow in Dump** permits you to dump tha tmemory address. 

You can edit memory in this window, too. Right click some memory and choose **Binary -> edit** to do so. This can be used to modify global variables and other data that malware stores in RAM. 


### Executing code

See page 186 for a nice table on the hotkeys used to execute code. There are a few notable features worth mentioning:
* When paused within library code, do a **Debug -> Execute until User Code** to run until you're no longer in the library. 
* Step-into: F7. 
* Step-over: F8


#### Breakpoints

You can add or remove a breakpoin tby selecting the instruction in the disassembler window and pressing F2. You can view active breakpoints by selecting ** View -> Breakpoints** or clicking the B icon in the toolbar. 

You can generally select what breakpoints you want to use by right-clicking and going from **Breakpoint -> [breakpoint type here]**. There are hotkeys for some of these too-- see p188.


### Buttons and other windows

#### Memory map window (M)
Click the M box or click **View -> Memory** to display all memory blocks allocated by the debugged program. This is a great way to see how a program is laid out in memory. All DLLs and their code sections are also visible. You cna double-click any row in the memory map to show a memory dump of that section, or you can send the data in a memory dump to the disassembler window by right-clicking and selecting **View in Disassembler.** **This seems very helpful for obfuscated malware.**

### DLLs
OllyDbg can debug DLL assets/images/posts. Since DLLs cannot be executed directly, OllyDbg uses a dummy program called *loaddll.exe* to load them. By default, OllyDbg breaks at the DLL entry point (dllmain) when the DLL is loaded.

To call exported functions with arguments inside the debugged DLL:
1. Load the DLL
2. Click the play button to run ``DllMain`` and any other intialization code the DLL requires
3. Olly will pause, and you can call specific exports with arguments and debug them by selecting **Debug -> Call DLL Export** from the main menu. (See an example of this on p191/p192.)



### Tracing
**Tracing** is a powerful debugging technique that records detailed execution information for you to examine. 

#### Standard Back Trace
Whenever you're moving through the disassembler window with the Step Into and Step Over functions, OllyDbg is recording the movement. You can use the minus key on the keyboard to move back in time and see the instructions previously executed. The plus key takes you forward. 

#### Call Stack
**View -> Call Stack** will allow you to view the call stack. Imagine that. 

To walk the call stack, click the Address or Called From sections of the call stack window. The registers and stack will not show you what was going on when in that location, unless you are performing a run trace.


#### Run Trace
A **run trace** allows you to execute code and have OllyDbg save every executed instruction and all changes made to the registers and flags. 

* Highlight code you wish to trace. Select **Run Trace -> Add Selection**. After, choose **View -> Run Trace** to see the instructions. Use the + and - keys as discussed above to run through the trace. 
* You can also use the **Trace Into** and **Trace Over** options. These will simply trace until a breakpoint is hit. 

### Patching
You can modify instructions or memory by highlighting a region, right-clicking the region, and selecting **Binary -> Edit**. This just changes the live memory; to take patching a step further and saving the on-disk executable, right click the disassembler window wher eyou patched the code and select **Copy to Executable->All Modifications**, and then select **Save file**. 

### Analyzing shellcode (or other random instructions)

See the step by step process on p196. All I would do in these notes is copy that ordered list of instructions, more or less otherwise. 




## Rebasing
This occurs when a module in Windows is not loaded at its preferred *base address.* All PE assets/images/posts in Windows have a preferred base address, known as the *image base* defined in the PE header. Most executables are designed to be loaded at ``0x400000``, which is probably why so many variables and functions in Ida are named using 0x400000 or some close number in some fashion. 

Of course, this is a preferred address, but it doesn't guarantee a program will be loaded there. For example, if a program has two DLLs and they each have the same base address, they can't both be loaded at the same address. 

Some instructions rely on the data being loaded at a specifc address, such as this instruction:

``mov eax, dword__40CF60``

That won't work if the DLL with that instruction isn't loaded in the right spot due to the use of an absolute address. Most DLLs come packaged with a list of fix-up locations in the ``.reloc`` section of the PE header. 

DLLs are loaded after the .exe and are loaded in any order, meaning you can't generally predict where DLLs will be located in memory if they're rebased. If a DLL lacking a relocation section can't be loaded at its preferred base address, then it cannot be loaded. 

## Wrapping up

Assistance features-- p197
* Logging
* Watches window (i.e. watching when certain things change)
* Help 
* Labelling
  * right click an address and select **Label** which will prop up a window, promting for a label name. Works like in Ida. 


How to use plugins-- p197

Scriptable debugging -- using Python to do plugins. 



