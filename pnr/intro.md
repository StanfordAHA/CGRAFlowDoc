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

Overall, there are four stages of PnR:
- [Packing](packing.md)
- [Placement](placement.md)
  - [Global placement](global-placement.md)
  - [Detailed placement](detailed-placement.md)
- [Routing](routing.md)
  - [Global routing](global-routing.md) (to be implemented)
  - [Detailed routing](detailed-routing.md)
- [Bitstream Generation](bitstream-gen.md)

### Usage
`cgra_info` repo has a one-button script to run the entire PnR flow:
```
$ ./scripts/pnr_flow.sh
Usage: ./scripts/pnr_flow.sh [--no-reg-fold] <arch_file> <netlist.json> [<output.bsb>]
    if <output.bsb> not specified, it will output <netlist.bsb>
    to the same directory as <netlist.json>
```

urrently everything is implemented in Python and it is compatible with both
Python 2 and Python 3. In the near future the core-part of the toolchain will
be implemented in C++ with Python binding.
