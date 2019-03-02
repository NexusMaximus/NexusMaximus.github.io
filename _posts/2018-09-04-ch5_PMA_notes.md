---
layout: post
title: "PMA Ch.5 notes: IDA Pro"
date: 2018-09-04 18:32:00 -0600
categories: notes
---

There is a HUGE amount of content to know about IDA, and these notes hardly puts a dent in covering the helpful content. 

## Chapter summary questions
1. In two sentences or less, provide an overview of what this chapter is about.
	
	This chapter introduces Ida Pro, which is one of the most important tools in the market. 

2. What are the three most important takeaways from this chapter?

	Ida is an extremely powerful tool, and the information in the chapter is just barely enough to *get started*. More will be provided as the book continues. 

3. What problems does this chapter address? In other words, why should we care about this chapter?

	This chapter is more of a flyover tutorial than a conceptually dense section of the book. As such, we should care because it provides tips and tricks to get a novice reverser up and running with Ida. 

## Terminology
**Rebasing**: PE assets/images/posts are compiled to load at a preferred base address. If the Windows loader cannot use this address (say, because it's taken up by another program), then *rebasing* may occur. It's not explicitly defined in this chapter, but they do mention that if DLLs appear to be loading into processes different than what Ida says, the DLL may be being rebased, and this can be fixed by clicking the *Manual Load* checkbox on the load file window. 



## Ida Cheatsheet
* switch between graph mode and text mode: **space**

* Cross references are things that help you find where certain names are used. Think of it as the opposite of clicking a link-- it goes the other direction. Find cross references with **x**. 

* Jumping to a virtual address can be done with **g**. GOTO, of course! 
  * Jumping to a raw offset is possible too with **jump -> jump to file offset**. 

* Has Ida failed to identify some code as a function? Create it by pressing **p**. 
  * Has Ida also failed to identify a EBP-based stack frame, and thus not abstracted away some simple changes that add for readability (like putting variable names into assembly instruction operand spots)? Try **Alt+p**, select **BP based frame**, and specify **4 bytes for saved registers.** 

* Add your own comments to places by putting the cursor on a line of a disassembly and hitting **:**. OR BETTER YET: put a comment on an address that has cross-references to it, and have the comment be repeated: hit **;**.

* Sometimes Ida mislabels numbers to be addresses, or vice versa, since everything's a number under the hood anyway. Change numbers to cross-referencable addresses and back via **O**. 

* To represent highlighted bytes as code: **C**

* To represent defined code as not a function, code, or data: **U**



## Ida Tips and Tricks

* When Idaing, you may find a few keystrokes suddenly causes the interface to become impossible to navigate (imagine the first couple times you used Vim). Reset the interface with **Windows -> reset desktop.**
  * Similarly, if you love your current desktop configuration, do a **Windows -> save desktop**. 

* Graph mode excludes certain helpful info, such as line numbers and op codes. Change this:
  * **Options -> general -> select ``line prefixes``** and set number of opcode bytes to 6. Most instructions are 6 or less bytes, so setting this to 6 will allow one to see the memory locations and opcode values for each instruction in the code listing. 

* **Auto Comments** can help aid analysis via adding assembly comments throughout the disassembly window. Go to **Options -> General** and click the **Auto Comments** checkbox. 


* For information on plugins for Ida, see page 103. I'm not good enough to appreciate that yet, so I'm putting this subsection on the backburner for now. 

### Useful Analysis Windows
* *Functions Window*: lists all functions in the executable and each function's length. 
* *Names window*: Lists every address with a name, including functions, named code, named data, and strings.
* *Strings window*: Shows all strings. Default is ASCII only; longer than 5 characters. 
  * Can change this by right-clicking in the strings window and selecting setup. 
* *Imports/Exports windows*: self-explanatory. The exports window is helpful when analyzing DLLs (since DLLs provide exports). 
* *Structures window*: Lists the layout of all active data structures. 


* In general, double clicking stuff in windows will link back to the disassembly window. 

### Searching
The Search functionality in the disassembly window is helpful. See p.94 for more details about common searches (if they aren't self explanatory in Ida). (Search -> next code, search -> test, search-> sequence of bytes for starters) 


### Graphing Options  
Graphing options help provide insight into the control flow of the program. However, due to how they are used, it  wouldn't be very helpful for me to annotate this section as it's so diagram dependent. Look at page 98 for graphing options information. 

### Enhancing the disassembly 

#### Renaming Locations
Should be doable by right clicking names of things like sub\_400699. Changes propogate throughout the program. 

#### Comments
Add your own comments to places by putting the cursor on a line of a disassembly and hitting **:**. OR BETTER YET: put a comment on an address that has cross-references to it, and have the comment be repeated: hit **;**.

#### Formatting operands
You can right click constants and represent the data there in a plethora of ways (binary, octal, hex, decimal). 

Imagine ``stdin.`` What number comes straight to mind? 0, of course! ``stdin`` is a named constant of sorts. Ida can help reverse where a compiler plugged in a number for a named constant in the initial source code. The option to do so is given in the same menu as the above options. 

If the constants don't show up, then the proper libraries that correspond to those function calls may not be loaded in Ida. Do a **View -> open subviews -> type libraries** to get to the submenu to change that. 

#### Loading symbolic constants
If you have random looking numbers being passed to Windows API functions, you can have Ida identify what constants those numbers refer to. Go to **View -> Open Subviews -> Type Libraries** and then right click and load a new library. Look for ``ntapi`` and load it. (I presume this will work assuming ``ntapi`` shows up, which, in my case, it does *not*.)

## Page-to-page notes

* When loading a program, Ida maps the file into memory in a similar fashion to how the OS does it. 

* When loading a file into Ida, you can choose to interpret the file as raw data. This can be helpful in the case that malware has additional stuff appended to the end of the executable. This extra stuff would normally not be loaded into memory. 

* Jump arrow color key:
  * *red*: Jump not taken
  * *green*: Jump taken
  * *blue*: unconditional jump

* Ida provides a *Navigation band.* 
  * Light blue is library code recognized by Ida's super cool 'FLIRT' technology. 
  * Red is compiler generated code.
  * Dark blue is user-written code. 






