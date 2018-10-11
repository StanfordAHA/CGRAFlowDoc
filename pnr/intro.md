# Introduction to PnR
Place-and-Route (PnR) is a critical step in the CGRA flow, where hardware
resources are configured based on application logic.

The input to PnR tolchain is usually a netlist, the same type of hypergraph
used in ASCI and FPGA. The details of hypergrpah can be found
[here](https://en.wikipedia.org/wiki/Hypergraph). Essentially the input is a
list of nets and in each net there is one output pin that drives one or more
input pins. In out CGRA, the pins are either 16-bit or 1-bit ports, such as
`data0` and `wen`.

The output from the PnR toolchain is usually a bitstream, which encodes the
control logic and will be load to the chip to perform computation. In our
CGRA, PnR tools output an assembly-level human-readable code that can be
directly compiled into bitstream through `bsbuilder`. The reason to use an
assembly-level is that we can easily see which part goes wrong. Also `bsbuider`
has many built-in checks so if PnR tools produce an illegal result, the mistake
will be caught by `bsbuilder`. However, as `garnet` becomes more and more
mature, PnR tools may choose `garnet` to generate bitstream directly.

Overall, there are three stages of PnR:
- [Packing](packing.md)
- Placement
  - Global placement
  - Detailed placement
- routing
  - Global routing (to be implemented)
  - Detailed routing
