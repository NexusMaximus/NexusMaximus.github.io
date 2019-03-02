---
layout: post
title: "Notes on Binary Instrumentation and Intel PIN"
date: 2018-12-13 12:31:00 -0600
categories: notes
---

Binary instrumentation is a technique that inserts extra code into a program to collect runtime information, as defined by [this source](http://www.ic.unicamp.br/~rodolfo/mo801/04-PinTutorial.pdf). [Intel PIN](https://software.intel.com/en-us/articles/pin-a-dynamic-binary-instrumentation-tool) provides a rich library used to do just this. 

Pin is the library provided by Intel to do instrumentation. Instsrumentation is conducted by inserting monitoring code into running executables. To do this, Pin makes use of Pintools, which  can be thought of as plugins that can modify the code generation process inside Pin. Pintools comprise two key components:
  * A mechanism taht decides where and waht code is inserted, and
  * The code to execute at insertion points.


Youtube [link](https://www.youtube.com/watch?v=kurzaoWuSHA) of someone setting up and using a tracer pintool to monitor Windows API calls (and more). 

## Terminology



## Steps
Downloaded PIN: https://software.intel.com/en-us/articles/pin-a-binary-instrumentation-tool-downloads

Worked on downloading dependencies. 
  * Cygwin: https://www.techspot.com/downloads/6590-cygwin.html
    * just 'next'-ed through all the defaults






