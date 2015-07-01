# Porting a Node.js based application over NaCl

<p align="justify"> This article describes the attempts of porting the <a href=https://atom.io>Atom editor</a> over <a href=https://developer.chrome.com/native-client>NaCl</a>, a detailed description of the research project, along with  a description of the major obstacles encountered. The need to port node.js over NaCl rose from the necessity of offering Atom a better security model, starting from the lowest level(native code) towards javascript context-level security. Node.js relies on <a href=https://code.google.com/p/v8>Google Chrome's V8 JavaScript Engine</a> to load native code modules, making NaCl an evident choice for security.
</p>

## Native Client

<p align="justify">
NaCl is an open-source technology that allows web applications to run native code while maintaing the level of security that a pure web application can offer, witout breaking portability. Essentially, it is a sandbox that can run C or C++ code securely. <a href=http://static.googleusercontent.com/media/research.google.com/ro//pubs/archive/34913.pdf>As we can read in the first paragraph of section 2</a> [1], on a conceptual level NaCl splits code into two parts, trusted and untrusted. The access between trusted(inner sandbox) and untrusted(exterior) code is mediated through a mechanism called <b>trampoline</b>.

A trampoline is a small piece of code, residing at the bottom of the untrusted process' address space. Each syscall has it's own trampoline,  although all trampolines are identical.  A trampoline does two things:
</p>

- Exits the hardware sandbox (on non-SFI implementations) by restoring the original system value of %ds.
- Calls the untrusted-to-trusted context switch function (NaClSyscallSeg)

<p align="justify"> In the figure below, we can see a transition from untrusted to trusted code, and back to untrusted again.</p>
<img src=https://goo.gl/EGrqPh height=40% width=40%>
</p>
<p align="justify">The figure, along with more information on how trampoline works can be found on <a href=https://www.chromium.org/nativeclient/reference/anatomy-of-a-sys>Anatomy of a Syscall</a>[2].
</p>

<p align="justify"> Having the basic NaCl concepts explained, we can look on how it would fit in an actual application.</p>
<img src=https://leafsr.files.wordpress.com/2012/09/4d5bf-nacl-architecture.png?w=400&h=341 height=40% width=40%>
<p align="justify">  As we can see in section 2.1 [1] IMC/ SRPC is the abstraction of inter process communication, it is used by trusted and untrusted modules. Another aspect we can notice is that there is no direct, unmediated access to the renderer process. Thus, the chance of illicit access gain decrease, as well as the chance that a malfunctioning module can take down the whole process.</p>

## Approach

<p align="justify">The native modules are loaded through V8 API, as such the main goal was quickly set as porting V8 over NaCl. Since our starting point was pretty much ground zero, we had no information on wether V8 could be ported over NaCl, if it has been done, if it is possible. Alternatively, if the whole V8 could not be ported, having a clear separation, the ability to draw a line between trusted and untrusted sections and prompting the user to explicitly approve was also an acceptable situation. </p>

## Research process

### First step - a custom V8 NaCl port
<p align="justify"> At first, I found this <a href=https://github.com/codius/codius-nacl-node> Node.js port over NaCl</a> done by the folks at Ripple. I asked them a few questions on how  they implemented it and if they made any major changes  from the official V8. Unfortunately, it did. That port lacked the <b>JIT compiler</b> or dynamic code generation. The performance hit was too significant in too many Atom routines to afford to use this version. After a quick deliberation, we decided that this version is not usable and decided to put effort into another approach. </p>

### Second step - a custom V8 NaCl port with dynamic code generation

<p align="justify">It was clear that the dynamic code generation was a dealbreaker, and thus I began search for a V8 ported over NaCl that did have JIT. I found <a href=https://github.com/p-march/nacl-v8>this nacl experimental build.</a> That V8   port was a research project in itself, accompanied by a highly detailed <a href=https://github.com/p-march/nacl-v8/blob/master/nacljit.pdf>paper</a> [3].<br><br>
While the research project was nonetheless useful, it was too difficult to use in production. The V8 port was more than one year old and not actively maintained. In order to adapt the port to a newer version of V8, significant changes were required. Moreover, it would have required to have someone continously integrate important V8 updates into the fork. Based on all the aforementioned facts, we decided to give up this approach, as well.</p> <br>


### Last step - official V8 port over NaCl

<p align="justify">While searching for V8 ports, I also encountered an official port of V8 over NaCl which was now our last and best shot at getting this to work. It also had a promising start, at least after <a href=https://groups.google.com/forum/#!topic/native-client-discuss/aribXPUx5B0>this discussion</a> with NaCl Google developers who were helpful and pointed me in the right direction. When trying to build, in order to test the V8, I ran into an unexpected issue. The V8 wouldn't build.  Now, this is often common due to missing dependencies, wrong setups or many more other circumstances, but this wouldn't change. I also opened a <a href=https://groups.google.com/forum/#!topic/v8-users/OnvT2A5rxQE> thread </a>  on V8 users, that has no answers to this day. While searching for a solution, I tried <strike>a couple</strike> way too many that I should have V8 versions, following this
<a href=https://gist.github.com/domenic/aca7774a5d94156bfcc1> V8 versions for embedders</a> guide, but none worked. <br><br>
The error I was getting was related to ARM, which, at first, didn't make any sense, as I was building over NaCl, but on x64. Tracing the cause I found <a href=https://groups.google.com/forum/#!topic/native-client-discuss/Xw5yCe3Ubwc>how the V8 was actually ported</a>,having V8 generate ARM instructions and then use a portable ARM interpreter to avoid the need to port V8 code generators to NaCl. This lead us to the obvious decision to give up.  </p>

## Conclusion

<p align="justify">This were all the alternatives we did find at the time of writing. Fully exploring these,along with the complex nature of V8 alone paired with the constrains imposed by NaCl led us to believe that having a V8 port of NaCl that could offer a better security model is something we cannot obtain - at least at the moment - unless fully we commit to the effort of porting and maintaining it ourselves.
</p>

## Details
Operating system  : Ubuntu 14.04 <br>
Platform : x64 <br>
Node version : v0.10.x

## References


[1] [Native Client: A Sandbox for Portable, Untrusted x86 Native Code](http://static.googleusercontent.com/media/research.google.com/ro//pubs/archive/34913.pdf)

[2][Anatomy of a Syscall](https://www.chromium.org/nativeclient/reference/anatomy-of-a-sys)

[3][Language-Independent Sandboxing of Just-In-Time Compilation and Self-Modifying Code](https://github.com/p-march/nacl-v8/blob/master/nacljit.pdf)
