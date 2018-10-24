# Introduction
Here are some fabric-independent CoreIR transformations that should be run (Although I do not think these are being run in Flow master at the moment)
- `packbitconstants` -Transformation to turn individual bit constants into array constants
- `packconnections` - Packs single bit connections into array connections
- `cullzexts` - Remove unnecessary zero-extends
- `cullgraph` - Remove unused modules
- `deletedeadinstances` -Remove unconnected instances
- `removeconstduplicates` - Remove constant duplicates.
- `fold-constants` - Does common subexpression elimination like `x+0` or `x*1`

## Mapper
The “CoreIR methodology” for mapping is to implement your mapper as a set of generators. CoreIR defines a basic set of primitives (like add, sub, mul, and, etc…) which are all generators without implementations. All the mapper really has to do is define that generator in terms of the technology-specific primitives (PEs, Mem tiles, IO, etc).

For this mapper, I also do some op substitution (LT as a GTE and a not) which is also implemented in terms of generators. For more complicated “primitives” like linebuffers, the mapper provides a custom generator implementation for the linebuffer also in terms of PEs and Mem tiles, as if it is any other op. This can be thought of as op specialization.

In addition to doing the above, I run the following passes:
- `rungenerators` - Simply runs any unrun generators (recursively)
- `verifyconnectivity` - Verifies there are no unconnected inputs
- `removebulkconnections` - forces all connections to be either single bit or single array connections
- `verifycanmap` - Verifies that all the coreIR modules are in terms of mappable PEs/Mems
- `deletedeadinstances` – Removes unconnected instances
- `removewires`  - Removes any unnecessary extra wires
- `flatten` - Flattens module hierarchy completely
- `constdupilcation` - Duplicates constants so they can be baked into the PE registers
- `memconst` - Hack to get around the fact that you cannot bake constants into Memory tiles.

Finally, I serialize the final graph into JSON and hand off to mapper.
