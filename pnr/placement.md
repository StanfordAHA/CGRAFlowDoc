# Introduction to Placement
Quote from [Wikipedia entry](https://en.wikipedia.org/wiki/Placement_(electronic_design_automation):
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

### Domain Specific Placement Algorithm
The placement algorithm uses the fact that all the inputs are generated
from Halide. Hence the inputs carries many patterns unique to image
processing applications, such as the usage of line buffer. The placement
algorithm first identifies the computational graph via static graph analysis.
Then it partitions the netlists into much smaller clusters. 

(keyi: more details to come.)

### Usage:
```
$ python place.py -h
usage: CGRA Placer [-h] -i PACKED_FILENAME -e NETLIST_EMBEDDING -o
                   PLACEMENT_FILENAME [-c CGRA_ARCH] [--no-reg-fold]
                   [--no-vis] [-s SEED] [-u LAMBDA_URL] [-f FPGA_ARCH]
                   [--mock MOCK_SIZE]

optional arguments:
  -h, --help            show this help message and exit
  -i PACKED_FILENAME, --input PACKED_FILENAME
                        Packed netlist file, e.g. harris.packed
  -e NETLIST_EMBEDDING, --embedding NETLIST_EMBEDDING
                        Netlist embedding file, e.g. harris.emb
  -o PLACEMENT_FILENAME, --output PLACEMENT_FILENAME
                        Placement result, e.g. harris.place
  -c CGRA_ARCH, --cgra CGRA_ARCH
                        CGRA architecture file
  --no-reg-fold         If set, the placer will treat registers as PE tiles
  --no-vis              If set, the placer won't show visualization result for
                        placement
  -s SEED, --seed SEED  Seed for placement. default is 0
  -u LAMBDA_URL, --url LAMBDA_URL
                        Lambda function entry for detailed placement. If set,
                        will try to connect to that URL
  -f FPGA_ARCH, --fpga FPGA_ARCH
                        ISPD FPGA architecture file
  --mock MOCK_SIZE      Mock CGRA board with provided size
```

## Placement file format:
The first line is a header telling you the column name and the second line is the separator.
```
Block Name          X   Y       #Block ID
----------------------------
io16_out        17  1       #i0
io16in_in_0     0   16      #i1
mul_347_348_349_PE      9   9       #p2
```
