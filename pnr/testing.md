# CGRA PnR Testing
There are two different types of testing in `cgra_pnr` in addition to the whole
flow test in `CGRAFlow`
- self-checking with applications
- micro-testing with Halide operators.

## Self-checking
There are lots of self-checks built-in the entire PnR flow. For every critical
step there is either an assertion or test code that raise exceptions when
conditions fail. It is not meant to cover every cases, but meant to serve a
quick check to ensure the core functionality.

These self checks include:
+ Placement
  + cell overlapping
  + illegal cell location
  + missing cell placement
+ Routing
  + invalid port connection
  + overuse routing resource
+ Bitstream generation
  + valid input files (packing, placement, and routing)
  + PnR files consistent with the netlist (from the mapper)

It uses the following applications to test:
- conv_1_2_mapped
- conv_2_1_mapped
- conv_3_1_mapped
- conv_bw_mapped
- onebit_bool_mapped
- pointwise_mapped

## Micro-testing with Halide Operators
There is a separate repo that runs daily that takes netlists and verifies with
verilator (from CGRAGenerator): https://github.com/Kuree/cgra_test.

Most of the ops are consistent with the ops being tested on Halide level, see
[here](../halide/application-list.md#test-cases).

Following operator passed the tests:
+ abs
+ ucomp
+ arith
+ uminmax
+ bool
+ scomp
+ shift
+ ternary
+ eq

Following operators failed to pass the tests due to the mapper:
+ sminmax
+ bitwise
