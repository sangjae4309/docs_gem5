---
layout: page
title: Ramulator2
parent: External Simulator
nav_order: 3
permalink: /external_simualtor/ramulator2
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## File Path Setting
```bash
gem5/
├── configs/ 
│   |
│   └── common/
│       |- MemConfig.py
│       |- Options.py
│       └─ ...
|   
├── src/ 
│   └── mem/
│ 	|- ramulator2.cc ramualator.hh
│ 	|- Ramulator2.py
│ 	|- SConscript
│ 	└─ ...
│
├── ext/  # not guided in this docs
│  └── ramulator2/
│ 	|- SConscript
│ 	└─ ramulator2/  
│ 	    |- src/
│ 	    |- libramulator.so
│ 	    └─ ...
```
- Above tree shows overall files which requires modification to integrate gem5 and ramulator2 and its path

{: .warning }
This docs mainly guides only modification on gem5 source files.
To set-up ramulator2 local file, please refer to [ramulator2 documentation.](https://github.com/CMU-SAFARI/ramulator2)

{: .important }
You can get all complete code at [my github.](https://github.com/sangjae4309/gem5_ramulator2)

-----

## Enviornment
- \>= Ubuntu22.04 (due to support on c++23)
- c++-12 (apt-get install c++-12)
- clang++-15 (apt-get install clang clang++-15)
- cmake
- gem5 ver.22.0

----

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
- Connect it to dram_intf


## Creating Wrapper
 - TBD

----

## Add option for Ramulator config file

```py
# gem5/configs/common/Options.py
...
parser.add_argument(
    "--ramulator-config",
    type=str,
    dest="ramulator_config",
    help="inputs ramulator configuration file"
)
...
```
- Now, the option `--ramulator-config` is added


```py
# gem5/configs/common/MemConfig.py create_mem_intf()
...
if issubclass(intf, m5.objects.Ramualator2):
    if not options.ramulator_config:
        print("--mem-type=Ramulator2 requires options --ramulator-config")
        exit(1)
    interface.config_path = options.ramulator_config
...
```
- Connect the option to Ramulator2 simobject
- `interface` is an instance of python-defined Ramulator2 class 
- `options.ramualtor_config` comes from previous subsection

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

    #NOTE newly added to accommodate config file
    config_path = Param.String("", "--ramulator-config")
```
- Define new varialbe in python class to accommodate the new option
- Compiling gem5, `param/Ramulator2.hh` adds new string variable `config_path`

```c++
# gem5/src/mem/ramualtor2.hh
namespace gem5 {
namespace memory {
class Ramulator2 : public AbstractMemory {
...
private:
    std::string config_path;
...
}
}}

# gem5/src/mem/ramualtor2.cc
Ramulator2::Ramulator2(const Params &p){ // Constructor
config_path = p.config_path;
YAML::Node config = parse_config(config_path); // this is pseudo-code
}
```
- In wrapper code, you can call the `config_path` by referencing member variable of Params which comes from `param/Ramulator2.hh`

---





