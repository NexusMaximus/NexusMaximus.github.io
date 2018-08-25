---
layout: post
title: "PMA Ch.3 notes: Basic Dynamic Analysis"
date: 2018-08-22 14:31:00 -0600
categories: notes
---

[/]:## Contents
[/]:* Running DLLs


## Chapter summary questions
1. In two sentences or less, provide an overview of what this chapter is about.
	
	This chapter introduces dynamic analysis as well as the main tools used to perform it and how to use those tools. **Dynamic analysis** is the act of observing a program run, and gathering information about what it's doing by watching it do its thing in real time while monitoring the state of the machine with several tools. 

2. What are the three most important takeaways from this chapter?

	Dynamic Analysis is powerful for analytic reasons, but dangerous, since it involves actually running malware. It's important to have properly set up a malware analysis environment before doing this form of analysis. 

3. What problems does this chapter address? In other words, why should we care about this chapter?

	The workflow introduced in this chapter is *typically* the same workflow used to analyze most malware samples, and provides overview information to the analyst about what the malware does. It is effectively the second step of the analysis process and it will generally help direct the analyst's efforts by showing them what they need to look more in depth at. 

## Terminology and Keys

__Dynamic Analysis__ is any examination performed during or after executing malware. It permits viewing of the malware's true functionality. 
  * Do static analysis first. Dynamic analysis is (obviously) much more dangerous. 

__Sandboxes__ are (\$\$\$\$) security mechanisms for running untrusted programs in a safe environment without fear of harming "real" systems. 
  * They are automated, and often make use of the CVD to check for common vulnerabilities being exploited.
  * They do, however, tend to just run the executable without command line options, which may not mimic the environment in which the malware was designed to execute in. 
  * Like virtual machines, sandboxes are detectable and may report an exectuable as not malicious if it senses its in a sandbox and reacts differently than in normal circumstances. 
  * Sandboxes can tell you specifics behind what a malware sample does, but it cannot perform overall semantic analysis. 


Service Entries key: ``HKLM\SYSTEM\CurrentControlSet\Services``

Services ran at startup: ``HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run``


## Page-to-page notes


### Running DLLs
*rundll32.exe* is what we use to run functions or ordinals from DLL files. Syntax:

```
C:\rundll32.exe DLLname, export arguments
```

So, imagine you open up a file *rip.dll* and examine its exports with PEView or with PE Explorer, and you see it has two exports: Install, Uninstall. You can run of these as such: 

```
C:\rundll32.exe rip.dll, Install
```
Or if there are functions exported by ordinal (i.e. as an exported function with only an ordinal number), you do this sort of thing:
```
C:\rundll32.exe someDll.exe, #5
```
The pound (#) is important. 

DLLMain() is executed whenever a DLL is loaded, so you can often get information dynamically by forcing the DLL to load using runndll32.exe. Alternatively, a DLL can be turned into an executable by modifying the PE header and changing its extension to force Windows to load the DLL as it would an executable. 
* Modifying the PE header: wipe the ``IMAGE_FILE_DLL`` (0x2000) flag from teh characteristics field in the ``IMAGE_FILE_HEADER``. This won't run imported functions, but it will run the DLLMain() method.  

Malware may install itself as a service too, potentially using a legit utility such as the convenient export ``InstallService`` from ipr32x.dll:
```
C:\> rundll32 ipr32x.dll,InstallService serviceNameHere
C:\> net start serviceNameHere
```
You can use ``sc`` on cmd to play with services as well. 

(I presume there is a 64 bit version of rundll for Windows but I've not looked into what it is at time of writing.)


### Faking a Network
If the malware reaches out to an external domain, it's helpful to fake being that external domain to further watch how the malware behaves. The book discusses using ApateDNS and Netcat on p51, but I'm going to use INetSim on a Remnux box and Wireshark to observe network behavior (see further down for more details on INetSim). 


## Tools discussed in this section
**procmon** is an advanced, extremely helpful monitoring tool for Windows that provides a way to monitor certain registry, file system, network, process, and thread activity. 
  * It can miss device driver activity of a user-mode component talking to a rootkit via device I/O controls, and can miss certain GUI calls such as ``SetWindowsHookEx``. 
  * Generally, it shouldn't be used for logging network activity, as it doesn't work consistently across Microsoft Windows versions. 
  * Using procmon: at and around pg44.
  * Filtering is super important with Procmon. 

**Process Explorer** is a free utility provided by Microsoft, probably as part of [sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite). 
  * Use it for listing active processes, DLLs loaded by a process, various process properties, and overall system information. It can also be used to kill processes, log out users, and launch/validate processes. 
  * It dynamically shows when new processes are created or modified, as well as what created them. 
  * You can *verify* binaries with it. There's a verify button on the image tab, and the image tab is found when viewing a process's properties. The verify button checks to see if the image on disk is the microsoft signed binary. 
    * This can be used to identify process replacement/injection, since that involves modifying the executable, which'll screw up the signature of the file. 
  * There is a strings window in the process properties collection of tabs as well. It can check for strings in the in-memory process as well as the on-disk image. If the strings vary greatly, you know that something fishy is going on. 
  * You can use process explorer to launch depends.exe -- Dependency Walker -- on a running process. Right click a process name and select "Launch Depends."
  * It can be used to analyze malicious documents by observing processes opened when clicking a malicious document. The image tab will tell you exactly where the malicious payload executed by the document is on disk. 

**Regshot** is used to take snapshots of the registry. Take the first shot before running the malware, and take the second shot after. It then tells you of all differences between the two snapshots. 

**Wireshark** is briefly discussed on page 53. 

**INetSim** is free, Linux-based software for simulating common Internet servies. It's widely accepted as the best tool for providing fake services, allowing you to analyze the network behavior by emulating servies such as HTTP(S), FTP, IRC, DNS, SMTP, and others. See page 55 of the book for a full list. 


## Malware Analysis Work Flow
Typically, you will use many of the tools simultaneously when executing and observing malware. My generic work flow sessions tend to go like this:
0. If you're observing network traffic, set up your virtual network (instructions to do this will be provided later).
1. Run procmon but don't capture events yet.
2. Start process explorer.
3. Start regshot. 
4. If you're observing network business, start Wireshark and INetSim on the remnux box. 
5. Take a VMWare Snapshot so you can return to this point multiple times for multiple detonations of the malware. 
6. Take the first registry snapshot.
7. Begin capturing procmon events. (regshot is super noisy and doing it outside of the procmon analysis window makes it such that that noise is ignored).
8. Detonate the malware!
9. Stop the capture of events after malware finishes running (or you're done running it or whatever)
10. Take the second registry snapshot. 
11. Analyze.
12. Revert back to the previous VMWare snapshot and do it again making whatever changes you want to make. 

The book also provides a solid sample analysis sesh starting on page 57. 






