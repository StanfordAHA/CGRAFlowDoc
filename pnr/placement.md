# Introduction to Placement
Quote from [Wikipedia entry](https://en.wikipedia.org/wiki/Placement_(electronic_design_automation)):
> Placement is an essential step in electronic design automation - the portion of the physical design flow that assigns exact locations for various circuit components within the chip's core area. An inferior placement assignment will not only affect the chip's performance but might also make it non-manufacturable by producing excessive wirelength, which is beyond available routing resources. Consequently, a placer must perform the assignment while optimizing a number of objectives to ensure that a circuit meets its performance demands.

In PnR toolchain, placement happens right after the packing stage. The main
goal of placement is to reduce the wirelength for each net and avoid routing
congestion if possible.

Usually the more heterogeneous the chip becomes, the more complicated the
legalization rules will be. By legalization or legal, we mean the cell location
is allowed on the chip. For instance, suppose we have a memory instance that
needs to be placed. If we place that memory instance on a PE tile, it is
illegal and the placement has to be legalized in order to proceed.

In modern ASIC/FPGA/CGRA, placement has two stages: global placement and
detailed placement. Typically during global placement instances are assigned
to a rough location and then legalized iteratively. In detailed placement, the
instance locations are further refined to reduce the wirelength. Because of
different chip designs, placement algorithms can get very complicated and
legalization and re-packing can happen at any iteration.
