---
layout: page
title: Integrate with gem5
nav_order: 2
has_children: true
permalink: /about/
---
## Overview
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
- Above tree shows overall files which requires modification to integrate gem5 and ramulator2
and its path

{: .warning }
This docs mainly guides only modification on gem5 source files.
To set-up ramulator2 local file, please reference to [ramulator2 documentation](https://github.com/CMU-SAFARI/ramulator2)

{: .important }
You can get all complete code at [sangjae4309/gem5_ramulator2](https://github.com/sangjae4309/gem5_ramulator2)


