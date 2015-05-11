# Google Summer of Code Project – Atom Security

This is a research project with the aim of discovering techniques we can apply to provide better security in Atom without sacrificing Atom's current power and flexibility. Summer of Code fellow @pandrei will be heading the project, and @nathansobo will be the contact on the core team. Progress reports and discussions for the project as a whole will take place on this repository. It will also include links to relevant repositories where work is taking place here.

## Project Summary

One of Atom's defining features is that code in extensions runs in the same process as code in Atom's core. This makes it easy to define rich, expressive APIs and exposes the full power of Atom's web-based platform, but it also presents challenges for security. Like every other popular extensible text editor, Atom's ecosystem currently relies on trust and reputation for security. When you install an Atom package, you're granting third party code broad access to your system. This means packages can do amazing things, but this power is a double edged sword. This project will explore techniques for limiting the privileges granted to third party code running within Atom, while at the same time preserving the convenience and power that makes it compelling to extend Atom in the first place.

The goal is a security model that is nearly transparent to the package author. They will continue to write packages as they do today, but will also include a manifest of sensitive privileges requested by their package. Users can view the privileges requested by a package upon installation, and make a conscious decision whether to grant them to a given package. For example, we may want to require an explicit permission grants for the following facilities:

* Read/write access to the file system
* Ability to run subprocesses
* Access to credentials stored in the system keychain

The project consists of two major components:

### Securing Native Extensions

This is the higher risk, more speculative component of the project. Because of Atom's Node.js integration, third party packages can run native V8 extensions to perform computationally intensive tasks or access system APIs. Because these are direct extensions to the V8 virtual machine, they run directly in the Atom render process, making process-based isolation an inadequate solution.

The big question we want to answer is whether it's possible to provide any security guarantees when running untrusted code in this configuration. We plan to focus our research around Google's Native Client project (NaCl). NaCl is used in Chrome to run untrusted native code with limited permissions by pre-validating machine code to ensure it doesn't perform unauthorized instructions.

Our situation is much different from Chrome's however. Chrome uses the NaCl validator to sandbox code at the instruction level, but this code is *also* run in its own sandboxed operating system process, a "double sandbox" design. The open question is whether NaCl's inner instruction level sandbox could be applied to code running in a very different context, namely directly in a Chrome render process as an extension of the v8 virtual machine.

The [Codius project](https://github.com/codius/codius-nacl-node) is an example of a limited version of Node.js running in an NaCl sandbox, which gives us hope that this may be possible in Atom.

The ideal solution meets the following criteria:

* Native extensions can be developed and compiled in the same manner they are today in a non-secure context.
* Modifications to Chrome and v8 are minimized. Preferably, no modifications to the source code of either project should be required.
* Overhead for non-sandboxed native code should be zero.
* Overhead for sandboxed native code should be low.

Ideally, our system can limit what system calls native code is allowed to perform, but otherwise require no modifications to today's development workflow.

### Securing JavaScript

This component is less speculative, and will be valuable independent of the results of the native security guarantees. Basically the goal is to explicitly grant or deny package access to various Atom APIs for JavaScript provided by third party packages using the object capabilities facilities built into ECMAScript 6.

This would involve loading each package in its own context and selectively exposing objects. These techniques have been extensively discussed by Mark S. Miller. [Here's a video](https://github.com/codius/codius-nacl-node).
