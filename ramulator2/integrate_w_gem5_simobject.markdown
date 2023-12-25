---
layout: page
title: Add to Simobject
parent: Integrate with gem5
nav_order: 2
permalink: /simobject/
---
## Introduction
- gem5 manages memory system in form of simobject
- This section describes how to add ramulator2 into simobject system
- Add the following codes in your own gem5

---

## Declaring Simobject
```py
# gem5/src/mem/SConscript
if env['HAVE_RAMULATOR2']:
    SimObject("Ramulator2.py", sim_objects=['Ramulator2'])
    Source("ramulator2.cc")
    DebugFlag("Ramulator2")
```
- SimObject("Ramulator2.py"...): Add class "Ramulator2" as a simobject.
- Source("ramulator2.cc"): compile the predefined c++ wrapper code.
- DebugFlag("Ramulator2"): (optional), add "Ramulator2" as one of gem5 debug flag


```py
# gem5/src/mem/Ramulator2.py
from m5.SimObject import *
from m5.params import *
from m5.objects.AbstractMemory import *

class Ramulator2(AbstractMemory):
    type = "Ramulator2"
    cxx_class = "gem5::memory::Ramulator2"
    cxx_header = "mem/ramulator2.hh"
    port = ResponsePort("The port for receiving memory requests and sending responses")
```
- Defines the python class for Ramulator2
- While compiling, gem5 follows cxx_headers path and reach cxx_class instance

---
## Connecting to dram_intf
```py
# gem5/common/MemConfig.py config_mem() new code
...
if issubclass(intf, m5.objects.Ramulator2):
    mem_ctrl = dram_intf
else:
    mem_ctrl = dram_intf.controller()
...
```
- While the conventional dram class returns dram interface instance by calling function called controller(), Ramulator2 is not
