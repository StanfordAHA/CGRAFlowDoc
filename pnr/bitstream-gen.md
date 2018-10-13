# Bitstream Generation
Given packing, placement, and routing result, we can finally write the
bitstream! In `cgra_pnr` I adopt a simple assembly language made from Steve
that has a tool called `bsbuilder` to compile the assembly language to the
actual bitstream. We will call the language `bsb`, short for `bitstream
builder`.

There are several reasons to use `bsb` files:
1. It's human-readble and editable. So it's easy to spot any error and correct
the bitstream by hand.
2. `bsbuider` has extensive checks for routings. So it's very helpful to debug
the router and the bitstream generator, especially in the early stage.
Although the PnR toolchain also has some self-check, it's also a good idea to
have a third-party checking your result.
3. The PnR toolchain does not need to worry about the underlying PE/SB/CB
encoding change as it's the `bsbuider`'s job to keep updated.

## Implementation Notes
Because the way `bsb` configures SB, the routing format can not be translated
to `bsb` directly. For isntance, the routing format is `out -> in`, whereas
the `bsb` uses `in -> out` on the same tile. In addition, because the router
uses MST to minimize the wirelength, there will be some *jumps* in the
routing result. The bitstream generator is required to recognize the jump
and figure out where tile makes the jump and reconstruct `in -> out` format.
