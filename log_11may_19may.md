# Log entry from 11 May to 19 May

I reached out to the people developing the Codius project ( Ripple ), they were very helpful and friendly but as Nathan further confirmed <br>
the port was not fit for Atom. V8 was lacking the JIT port, which was instead interpreted. This lead was dropped because of performance concerns 
The [Node.js port to NaCl](https://github.com/codius/codius-nacl-node) proved not to be fit for our situation because of JIT.

See [relevant discussion in this direction](https://groups.google.com/forum/#!topic/native-client-discuss/aribXPUx5B0) for more details.

Not having JIT was pretty much a dealbreaker, so I decided to drop the current path with the Codius V8 and focus on what the NaCl developers
suggested(the official V8 port to NaCl, in the V8 repo).

