---
layout: post
title: "PMA Ch. 7 notes: Analyzing Malicious Windows Programs"
date: 2018-09-10 14:31:00 -0600
categories: notes
---


## Chapter summary questions

1. In two sentences or less, provide an overview of what this chapter is about.

  This chapter discusses a plethora of the Windows mechanics that malware authors frequently like to make use of. 

2. What are the three most important takeaways from this chapter?

  The attack surface for Windows-based malware is huge, and it's important to have a grasp on as much of the workings of Windows internals as possible to be best equipped for analyzing malicious samples. 

3. What problems does this chapter address? In other words, why should we care about this chapter?

  Windows is the most popular malware target simply because more people use Windows than other operating systems. As such, it pays to know how Windows works. 

## Windows Mechanics


### Common Windows API Data Types 

* ``WORD`` (w): 16 bits
* ``DWORD`` (dw): 32 bits 
* Handle: a reference to an object. They are like pointers in that they refer to objects or memory addresses, but include more information and cannot be manipulated like pointers. 
  * Examples: ``HModule``, ``HInstance``, ``HKey``. 
* Long Pointer (LP): Pointer to another type. 32 bits, I believe.
  * Examples: ``LPByte`` is a pointer to a byte. 
* Callback: Represents a function that will be called by the Windows API. 


### Special Files
**Special File**: one that can be accessed much like other files, but are not accessed by their drive letter and path. They can be hidden to directory listings. 

#### Shared Files

These start with ``\\serverName\share`` or ``\\?\serverName\share``. The ``\\?\`` prefix tells the OS to disable all string parsing, and allows access to longer filenames. 


#### Namespaces


**Namepsaces** can be thought of as a fixed number of folders, each storing different types of objects. All namespaces exist within the NT namespace, and this can be browsed using the *WinObj Object Manager namespace viewer* from Microsoft. 
  The Win32 device namespace with the prefix ``\\.\`` can be used to access physical devices directly, and read and write to them like a file. Again, this can be used to get around Windows API calls and avoid detection. 

  **It seems like namepsaces are a significant threat to analysts due to the covert abilities it equips malware with.**


#### Alternate Data Streams

This allows additional data to be added to an existing file within NTFS. The extra data does not show up from within a directory listing, nor is it displayed when showing the contents of a file. *It's only visible when viewing the stream.*

ADS data is named according to the convention *normalFile.txt:Stream:$DATA*, which is what allows a program to read and write to a stream. (I don't really understand the concept of streams in this context yet, but if it's not further answered in the chapter, I'll look it up elsewhere.)


### The Windows Registry
The registry is used to store OS and program configuration information, such as settings and options. It is often used by malware to establish persistence in addition to its usual use of configuration. Important terminology related to the registry:

* *Root Key*: registry is divided into five top-level sections called root keys. ``HKEY`` and ``hive`` are pseudonyms of this. 
* *Key*: It's like a folder in the registry.
* *Subkey*: it's like a subfolder within a folder. A subkey is a key. 
* *Value Entry*: An ordered pair with a name and a value.
* *Value or Data*: data stored in a registry entry. 

#### Root Key 101

* ``HKEY_LOCAL_MACHINE (HKLM)``: Stores settings global to the local machine
* ``HKEY_CURRENT_USER (HKCU)``: Stores settings specific to the current user
* ``HKEY_CLASSES_ROOT``: Stores info defining types
* ``HKEY_CURRENT_CONFIG``: Stores settings about the current hardware configuration (especially differences between current and standard configuration)
* ``HKEY_USERS``: Defines settings for default/new/current users

#### Popular ways to establish persistence
``HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`` contains values that are executables that are started as soon as a user logs in. 


### Networking APIs
Note that ``WSAStartup()`` must be called before other networking functions to allocate resources for the networking libraries. 

#### Berkeley Compatible Sockets
These socket functions provide functionality that is almost identical on Unix and Windows systems, and these functions are implemented in the Winsock libraries-- *ws2_32.dll*. Most common:
* ``socket``: Creates a socket.
* ``bind``: Attaches a socket to a particular port, prior to the accept call
* ``listen``: Indicates that a socket will be listening for incoming connections
* ``accept``: Opens a connection to a remote socket and accepts that connection
* ``connect``: Opens a connection to a remote socket that is waiting for the connection
* ``recv``: Receives data from the remote socket
* ``send``: Sends data to the remote socket

#### WinINet API
These functions are stored in *Wininet.dll* and provide higher-level networking APIs. 
* ``InternetOpen``: used to initialize a connection to the Internet
* ``InternetOpenUrl``: used to connect to a HTTP or FTP resource
* ``InternetReadFile``: allows the program to read data from a file downloaded from the Internet. 


### Processes
The OS will allocate memory for processes on a per-process basis. One process's addresses may be the same as another process's addresses, but due to virtual memory and abstraction by the OS (I believe), these processes will not overlap.


### Threads
A process's instructions are executed by one or more threads. A thread is an independent sequence of instructions that are executed by the CPU without waiting for other threads. Threads within a process share the same memory space, but each has its own processer registers and stack. 

Threads have complete control of the CPU when they're executing. Their context, the *thread context*, is saved in a structure when a context switch is performed. 


### Mutexes (aka Mutants)
These are global objects used to control access to shared resources. They often use hard-coded names, which acts as a host-based indicator to the malware analyst. 

### Services
These are ran by SYSTEM and can only be installed with administrator privilege. There are different types of services, each of which execute in different ways. The most commonly maliciously used type is the ``WIN32_SHARE_PROCESS``, type, which stores the code for the service in a DLL and combines several different services into a single, shared process. 

In Task Manager, there exist several *svchost.exe* instances, and these run ``WIN32_SHARE_PROCESS`` type services. ``WIN32_OWN_PROCESS`` is also used because it stores the code in a .exe file and runs it as an independent process. 

Services information is stored in ``HKLM\SYSTEM\CurrentControlset\Services``. 

### COM executables
The *Component Object Model* is an interface standard that makes it possible for different software components to call each other's code without knowledge of specifics of each other. It was designed to support reusable software componets that could be usable by all programs. 

COM is implemented as a client-server framework. The clients are what make use of the COM objects, and the servers are the reusable software components. Every thread that uses COM must call ``OleInitialize`` or ``CoInitializeEx`` at least once before calling other COM library functions. 

COM objects are accessed via their *globally unique identifiers* (GUIDs) known as *class identifiers (CLSIDs)* and *interface identifiers (IIDs)*. 

One common function used by malware is ``Navigate`` which allows a program to launch Internet Explorer and access a web address. ``Navigate`` is part of the ``IWebBrowser2`` interface, which specifies a list of functions that must be implemented, but does not specify which program will provide that functionality. A *COM Class* is the program that provides said functionality. Interfaces are identified with an *IID* and the classes are identifed with a *CLSID*. 

COM stuff is tricky and dense. I am going to bookmark this page (by saying here that it's on and near p155) and come back to it if I encounter such things in malware.


### Exceptions
Windows uses *structured exception handling (SEH)* to deal with handling exceptions. In 32 bit systems, SEH info is stored on the stack (which means it's potentially vulnerable to things such as buffer overflows). 

### Kernel vs. User Mode
Nearly all code runs in user mode, save the operating system itself and hardware drivers. Usually, user mode programs do not have direct access to hardware devices and *must* go through the Windows API. ``SYSENTER``, ``SYSCALL``, and ``Int 0x2E`` assembly instructions are all used to do this. 

All processes running in the kernel share resources and memory addresses, and kernel code has fewer security checks, not to mention it has ultimate power over a system. It's basically the priviliged land for malware authors.

### The Native API
The Native API is a lower-level interface for interacting with Windows that is rarely used by nonmalicious programs. Calling native API functions bypasses the normal Windows API. See p159 for a diagram describing how context switches from user mode to kernel mode when calling native functions. Native functions are usually predicated by Nt. 

``ntoskrnl.exe`` contains functions that run in kernel space. 




## Quick Windows Function Cheat Sheet
I include some unintuitive conceptual 'gotchas' here, but I don't discuss details behind how to use the functions. Check out MSDN documentation for that.

### File System Functions
* CreateFile
* ReadFile and WriteFile
* CreateFileMapping and MapViewOfFile
  - *File Mappings* are commonly used by malware writers because they allow a file to be loaded into memory and manipulated easily. ``CreateFileMapping`` loads a file from disk into memory; ``MapViewOfFile`` returns a pointer to the base address of that mapping. 
  - File Mappings can be used to replicate the functionality of the Windows loader-- after the malware reads in an executable file to memory, it can parse the PE and do whatever it wants with it, (possibly) without the use of Windows API functions, which can help evade monitoring programs. 


### Common Registry Functions
* RegOpenKeyEx
* RegSetValueEx: Adds a new value to the registry and sets its data (or modifies an existing value)
* RegGetValue: Get the data for a value entry in the registry

#### .reg scripts
You can write scripts that interact with the registry. Files with the .reg extension, when double clicked, modifies the registry by merging the info the file contains *into* the registry. 


### Common Process Functions
* CreateProcess
  * One of the parameters to this function is the ``startupinfo`` structure. You can configure ``stdin``, ``stdio``, and ``stderr`` streams in this, and potentially set them to sockets that you can send across the wire. See p148 for an example of this.

  
  Executables can store other executables in their resource section. An executable can extract an executable from within itself in this manner, write it to disk, and then call ``createProcess`` to run it. 


### Common Thread Functions
* CreateThread. The caller of this function specifies the start point of the thread. 
  * The caller of CreateThread can specify the function where the thread starts, and a **single** parameter that can be passed to this function. 
    * Malware can use CreateThread to load a new malicious library into a process when LoadLibrary is what the thread is configured to run, and the parameter is the name of the library that should be passed to LoadLibrary. LoadLibrary could then load a malicious DLL, thereby invoking DllMain, which could do more malicious things. 
    * Malware could use CreateThread to create two separate threads-- one for input, and one for output. 

### Common Mutex Functions
* WaitForSingleObject is used by a thread to wait on a mutex. 
* ReleaseMutex is called by a thread when it's done with a mutex. Obv. 
* CreateMutex: it destroys a mutex. (sarcasm)
* OpenMutex: One process can get a handle to another process's mutex by using this. Malware will commonly create a mutex and attempt to open an existing mutex with this technique to ensure only one version of the malware is running on the machine at once. 


### Common Services-related Functions
* OpenSCManager: returns a handle to the service control manager (SCM), which is used for all subsequent service-related function calls. 
* CreateService: adds a new service to the SCM. Caller specifies if the service is to be ran at startup or not. 
* StartService: You guessed it! It starts a service. It's used only if the service is not configured to start automatically (or, say, if you just freshly installed the malware on a system and it hasn't rebooted the machine yet). 

## Transferring execution among files

### DLL files
DLLs export functions and cannot be ran on their own. They must run their code via an executable of some form. 
* They only needed to be loaded into memory once. Then they can be used by multiple programs (unlike static libraries, which would need to be loaded on a per-exectuable-instance basis). 
* DLLs look extremely similar to exectuable files-- they use the same format as an execuable (the PE file format).A single flag differentiates the two. 
* DLLs contain DllMain()-- a function that has no label and isn't an export in the DLL, but is specified in the PE header as the file's entry point. It's called to notify the DLL when a process loads or unloads the library, creates a new thread, or finishes an existing thread. This allows the DLL to manage any per-process or per-thread resources. 







## Other page-to-page notes

*Autoruns* is a program downloadable from Microsoft that lists code that'll run automatically when the OS starts. It pulls from about 25-30 registry locations (albeit not all of the possible ones). 






