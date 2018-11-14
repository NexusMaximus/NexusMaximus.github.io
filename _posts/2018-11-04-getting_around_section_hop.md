---
layout: post
title: "Evading OllyDump's 'Find OEP by Section Hop' feature"
date: 2018-11-03 14:31:00 -0600
categories: notes
---


## OllyDump

One of the best tools in the malware analyst's toolbox is OllyDbg's 'OllyDump' plugin. It sports the ability to dump a debugged program's memory, meaning if you can unpack a packed executable up to the point in which it transfers execution to the malicious payload, OllyDump can write out that program to a file that you can further analyze. This point in execution is called the *original entry point*, or OEP. The jump that the program makes from unpacking code to execution of the malicious payload is called the *tail jump*. 

### Finding the OEP by section hop
Finding the OEP of a program isn't always trivial. That's why OllyDump provides two methods that can be used to help you. Labeled "Find OEP by Section Hop," OllyDump creates a conditional breakpoint that monitors the section of the executable that is currently executing. When execution hops out of that section, the breakpoint triggers. Often times, the section will change when making the tail jump, since the malicious payload likely won't be in the same section as the unpacker code. OllyDump has two versions of this functionality: a trace over feature and a trace into feature. From what I understand, the trace over feature won't monitor section transfers within calls. For example, if the code you're looking at calls a library function, the EIP will definitely be out of range of the current section, but Trace Over won't trigger a break in execution. Trace into will, however. As such, Trace Into goes into more depth, but can also churn out more false positives. Trace Over goes into less depth, but is less likely to churn out false positives. Each is useful in certain situations. Sometimes using a combination of both is the best strategy. Sometimes, neither one will work well. 


### Evading this tool
So, how can malware nullify these tools? I can think of a couple ways:

1. If the unpacking stub is contained within two or more sections and transfers execution between them frequently. Then, the section would change frequently, causing both versions of the section hop feature to trigger. Of course, this could be combatted by adding a multi-section conditional check, but I don't believe this feature is currently available within OllyDump. 

2. Self-modifying code/partially unpacking malware will cause problems. If the code only partially unpacks itself into memory, we may find the section in which it executes using OllyDump's features, but we can't dump the file into a complete executable at any given moment in time. 

In order to further understand how to get around this technique, it will pay to read up on general obfuscation techniques. That will, in turn, lead me to theorize about which can be detected with OllyDbg and which can't.


### Details on how a few common packers work, and how to augment OllyDump's "Find OEP by section hop" functionality to be able to handle these packers

#### FSG: FAst, Small, Good


#### UPX: Ultimate Packer for eXecutables


#### UPack


#### PolyEnE



#### MEW



#### PECompact



#### NSPack



#### nPack


#### ASPack



#### PEtite



#### Yoda's Protector



#### ASProtect












