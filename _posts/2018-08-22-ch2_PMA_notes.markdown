---
layout: post
title: "PMA Ch.2 notes: Malware Analysis in Virtual Machines"
date: 2018-08-22 14:31:00 -0600
categories: notes
---


## Chapter summary questions
1. In two sentences or less, provide an overview of what this chapter is about.

	This short chapter discusses how to create a virtual environment to analyze malware in. It explains details about the author's preferred tool, VMWare Workstation, and details how to create/configure a virtual machine, including details about virtual networking, snapshotting, using peripheral devices, VMWare's record/replay feature, and provides discussion on the risks that must be considered when analyzing malware in ANY case. 


2. What are the three most important takeaways from this chapter?
	
	There is always some risk inherent with the field of malware analysis, but using the right tools can help attenuate the risk to an acceptable level. 

3. What problems does this chapter address? In other words, why should we care about this chapter?

	This chapter provides the key information behind how to set up the analysis environment used in the rest of the book. 

# Page-to-page notes
* Structure of a VM: pg30
* _Snapshotting_ is the act of saving the state of a VM and being able to revert back to it within seconds. THIS IS WHY VMWARE WORKSTATION IS NECESSARY. Player does not support this feature. Now's the time when you agree and go buy workstation. :)
* Configuring virtual networks using services machines such as Remnux: pg33
* **connecting malware to the internet is genreally a bad plan. it can:**
  * spread around
  * Notify the author that it's on a machine
  * drop more malware
  * etc etc

	so NEVER connect malware to the internet without first doing some analysis to determine what it might do when it connects. Then, use your best judgement to determine whether or not it's worth it to connect the malware to the internet. 
* Malware can detect if it's running in a virtual machine, and it can behave differently if it is. Of course, at the end of the day, it's just running instructions, and those can be checked for and modified by the analyst, but it all takes time. 
* **There is ALWAYS an inherent risk from running malware even with a virtual environment. Good practices can help attenuate the risk, but will never make the risk 0.** 



## Tools discussed in this section
[VMWare Workstation](https://www.vmware.com/products/workstation-pro.html): potentially the most important piece of software for malware analysis. 
* Record and replay seemed especially neat and potentially useful. It records _every_ instruction executed in the process of a recording session. I'm sure there are uses for this. 







