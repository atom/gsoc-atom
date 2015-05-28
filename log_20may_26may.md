# Log entry from 20 May to 26 May

Worked on building the V8 from it's [official repo](https://github.com/v8/v8-git-mirror). Didn't work initially due to a option unknown to GCC
('-Wno-format-pedantic') . Worked smoothly for some architectures(x64, ia32, arm) but failed for NaCl, both for nacl_ia32 and nacl_x64


Tried searching for the error, found some similar issues in other tools, but nothing V8 specific. Opened [this thread](https://groups.google.com/forum/#!topic/v8-users/OnvT2A5rxQE) a bit too late, unfortunately.


Since I was stuck with this, I tried to make use of my time and read [Language-Independent Sandboxing of Just-In-Time Compilation and Self-Modifying Code](http://static.googleusercontent.com/media/research.google.com/ro//pubs/archive/37204.pdf) which, imho, proves that what we are looking for is not only possible, but already done.

(This V8 port)[https://github.com/p-march/v8-taint] is <i><not/i> related to NaCl but interesting nonetheless, could prove a significant optimization even on how things are right now.

Moreover, (here)[https://github.com/p-march/nacl-v8] we have a V8 port to NaCl that does have JIT via NaCl calls. Looking forward to what this unfolds to!