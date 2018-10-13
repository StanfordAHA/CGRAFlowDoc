# Introduction to PnR
Place-and-Route (PnR) is a critical step in the CGRA flow, where hardware
resources are configured based on application logic.

The input to PnR tolchain is usually a netlist, the same type of hypergraph
used in ASCI and FPGA. The details of hypergrpah can be found
[here](https://en.wikipedia.org/wiki/Hypergraph). Essentially the input is a
list of nets and in each net there is one output pin/port that drives one or
more input pins/ports. We will use pin and ports interchangably throughout this
documentation. In out CGRA, the ports are either 16-bit or 1-bit, such as
`data0` and `wen`.

The output from the PnR toolchain is usually a bitstream, which encodes the
control logic and will be load to the chip to perform computation. In our
CGRAFlow, `cgra_pnr` outputs an assembly-level human-readable code that can be
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

Timing and area analysis:
- [Analysis](analysis.md)

Call for contribution:
- [Retiming](retiming.md)

### Install
```
$ git clone https://github.com/Kuree/cgra_pnr
$ make
$ pip install -r requirements.txt
```

### Usage
`cgra_info` repo has a one-button script to run the entire PnR flow:
```
$ ./scripts/pnr_flow.sh
Usage: ./scripts/pnr_flow.sh [--no-reg-fold] <arch_file> <netlist.json> [<output.bsb>]
    if <output.bsb> not specified, it will output <netlist.bsb>
    to the same directory as <netlist.json>
```

Currently everything is implemented in Python and it is compatible with both
Python 2 and Python 3. In the near future the core-part of the toolchain will
be implemented in C++ with Python binding.

### Place for FPGA
The placer should be able to place various FPGA placement benchmark such as
VPR. Titan 23, and ISPD. However, because it's not designed to place generic
netlists, it may not obtain an optimal solution, or may be very slow to
converge.
