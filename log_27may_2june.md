# Log entry from 27 May to 2 June


The [V8 port](https://github.com/p-march/nacl-v8) I found last week is more of an experimental build, a proof of concept rather than a production build. Overall, no-go.

Shifted focus soley on official V8. After (way too) many attempts of [building V8](https://github.com/v8/v8-git-mirror) and looking through build system/code, I concluded that even if I could get it to work on - some - setup, it's definitely not production ready.

With a bit more research I found this [NaCl thread](https://groups.google.com/forum/#!topic/native-client-discuss/Xw5yCe3Ubwc) in which is described that V8 has been ported to NaCl through ARM instructions generators.

Another thing I researched was sandboxing node modules in pure JS context,
this [node mudule](https://github.com/felixge/node-sandboxed-module) is exactly what we were looking for but it ended up on a shelf(for now).

Now, the focus has shifted from the Security, Nacl-based investigation towards a keyboard issue, specifically fixing alt-codes for Atom.
