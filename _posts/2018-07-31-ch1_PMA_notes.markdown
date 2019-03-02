---
layout: post
title: "PMA Ch.1: Basic Static Analysis Techniques"
date: 2018-07-31 14:11:00 -0600
categories: notes
---

As I walk through the Practical Malware Analysis book, I'll be doing two posts per studied chapter: one that acts as a brief overview of what I think is the most helpful content from the chapter, and one that walks through my analysis of the lab assignments.

In addition to including technical details inscribed within the chapter, I'll be answering the following questions in an attempt to keep in mind the big picture of the chapter. 

1. In two sentences or less, provide an overview of what this chapter is about.
2. What are the three most important takeaways from this chapter?
3. What problems does this chapter address? In other words, why should we care about this chapter?


Let's get started. 


## Chapter summary questions
1. In two sentences or less, provide an overview of what this chapter is about.
	
	This chapter discusses basic static analysis of executable assets/images/posts and provides a slew of tools used to do so. Static analysis of a file is analysis that does NOT involve running the file, such as by examining the header of the file and looking for strings contained within. 


2. What are the three most important takeaways from this chapter?

	* Static analysis is one tool in the toolbox for identifying malware, but there are ways to make static analysis virtually useless. 
	* Section names are merely convention. They can be renamed to be misleading.
	* There are three kinds of linking: *static*, *runtime*, and *dynamic*. Malware more frequently makes use of runtime linking due to how it's easier to conceal runtime behavior using this version. 


3. What problems does this chapter address? In other words, why should we care about this chapter?

	Static analysis is the first step in analyzing any executable, and you should do it. It can provide clues that will pave the way for productive dynamic analysis.





## Page-to-page notes
* Ascii strings use 1 byte per char; Unicode strings use two per char. 
* Packed and otherwise obfuscated code will usually include library functions "GetProcAddress" and "LoadLibrary" as those two are typically used to decrypt/deobfuscate other functions. **These two functions can be used to access any function in any library on the system.**

* Linking is used to connect code in libraries to an executable. There are three kinds of linking:
  * _Static:_ the library code is copied ver batim into the executable. Common in malware due to how it can be obfuscated; rare in goodware as there's little purpose to do it other than to hide meaning of a program.
  * _Runtime:_ Library code is searched for only when needed within a program, as opposed to at the beginning of execution of a program (see dynamic linking). It's popular with malware writers as the imports can be concealed at the beginning of the program's execution and revealed at some point in the runtie of the program, making static analysis harder. 
  * _Dynamic:_ The host OS searches for necessary libraries only when the program is loaded. When executing library code, the code executed is within the library's space (i.e. it's not copied somewhere and executed there). 

* Appendix A contains lots of DLLs of high interest to malware analysts. 

#### A little about the PE file format

Consists of a header followed by a series of sections:
* **.text**: Contains instructions that the CPU executes
* **.rdata**: typically contains import and export information, which is the same info available from Dependency Walker (a tool) and PEViEW (a slightly out of date tool). 
* **.data**: Contains program's global data, accessable from anywhere in the program. 
* **.rsrc**: includes resources used by the executable that are not ocnsidered part of the executable (such as icons, images, menus, and strings). Strings are usually (but not necessarily) stored here for multilanguage support.

These section names are convention; Windows doesn't require them to be labeled as above. It determines what section is what based on info in the PE header. 



## Tools discussed in this section
[Dependency Walker](http://dependencywalker.com/) is a free utility that scans Windows executables and builds a hierarchical tree diagram of all dependent modules. 
P16 discusses a bit about how to use it.

PeID: support was discontinued in 2011, but it is still helpful for detecting packed assets/images/posts.

[PEview](http://wjradburn.com/software/): used to view executable file metadata (i.e. information in the headers of an executable).

[Resource Hacker](http://angusj.com/): browses the .rsrc section of an executable. P25 discusses usage of this tool. It also enables one to extract embedded assets/images/posts from within executable assets/images/posts. 

[PEBrowse Professional](http://www.smidgeonsoft.prohosting.com/pebrowse-pro-file-viewer.html) is similar to PEview and allows you to look at the bytes from each section. It does a good job of presenting info from the resource (.rsrc) section.

[PE Explorer](http://heaventools.com/) has a rich GUI that allows one to navigate through the various parts of a PE file. It permits editing assets/images/posts, too. RIP if you don't have money to pay for it, though. 



## Questions encountered (and answers, if found)

_Ascii  representation of "BAD":_
0x42 0x41 0x44 0x00

_Unicode representation of "BAD":_
0x42 0x00 0x41 0x00 0x44 0x00 0x00 0x00

What are the extra null terminators after each character typically used for?


###### LA-UR-18-27643

