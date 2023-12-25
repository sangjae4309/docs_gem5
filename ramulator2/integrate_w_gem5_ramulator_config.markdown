---
layout: page
title: Adding ramulator config
parent: Integrate with gem5
nav_order: 4
permalink: /ramulator_config/
---
## Introduction
- Ramulator2 selects the memory types and spec with its configuation file in form of `.yaml`
- Simulating Ramulator2-alone requires option `-f`(e.g, `-f ./example_config.yaml`) to load configuration.
- However, in gem5, the flag to read ramulator2 configuration file not exists.
- This section guides how to manually add new option and connects it to simobject

---
## Add option to parser

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

---

## Connect the option to Ramulator2 simobject
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

