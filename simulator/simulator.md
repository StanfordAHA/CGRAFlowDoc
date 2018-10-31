# Simulator (run_tbg.csh)

The simulator takes a CGRA bitstream, plus an input image, and
produces an output image.  The output image is supposed to be the same
as actual CGRA hardware would build.

`run_tbg.csh` is the driver for the simulator.  Its main
purpose is to call the [Test Bench Generator](tbg/intro.md) (TBG).
Given a ```bsa`` bitstream from the
[Assembler](bsbuilder/bsbuilder.md), plus verilog from the [CGRA
Generator](cgra/cgra-generator.md), TBG builds a custom testbench.

Again making heavy use of TBG scripts, `run_tbg.csh` invokes the
generated testbench using a given input image, and compares the
resulting output image against a given gold standard.

In more detail, `run_tbg.csh` currently does the following:
* verifies and echoes the current github branch;
* verifies and echoes essential details of the target CGRA design (i.e. memheight);
* cleans (removes comments from) and verifies validity of bitstream config file;
* optionally calls the [CGRA Generator](cgra/cgra-generator.md) to (re)build verilog from scratch;
* calls TBG `process_input` script to shape the input image according to DELAY paramater;
* uses verilator to run the testbench on the input image;
* calls TBG `process_output` script to shape the output image according to DELAY paramater;
* compares output file(s) to gold standard file(s)
* emits PASS (exits normally) or FAIL (exits with error code)




#### run.csh vs. run_tbg.csh

`run.csh` is the deprecated older version of run_tbg.  It uses a
handwritten test bench that is compiled per CGRA design, and can then
be used with any bitstream, as opposed to an automatically generated
testbench customeized per bitstream as with TBG.



#### Usage and defaults (--help)

```
./run_tbg.csh --help
run_tbg.csh: I think we are in branch 'genspec'

top.v:
run.csh: Looks like memtile_height is 1

Usage:
    ./run_tbg.csh <textbench.cpp> -q [-gen | -nogen] [-nobuild]
        -usemem -allreg
        -config    <config_filename.bs>
        -io_config <io_config_filename.json>
        -input     <input_filename.png>
        -output    <output_filename.raw>
        -out1    <1bitout_filename>,
        -delay   <ncy_delay_in>,<ncy_delay_out>
       [-input-size <8>]
       [-output-size <8>]
       [-trace   <trace_filename.vcd>]
        -nclocks <max_ncycles e.g. '100K' or '5M' or '3576602'>
        -build   # no longer supported, use -rebuild_from_scratch instead
        -nobuild # no genesis, no verilator build
        -nogen   # no genesis
        -gen     # genesis

Defaults:
    ./run_tbg.csh top_tb.cpp \
       -gen         \
       -config    ../../bitstream/examples/pw_padring_shortmem.bsa \
       -io_config /nobackup/steveri/github/CGRAGenerator/verilator/generator_z_tb/io/2in2out.json \
       -input     io/conv_bw_in.png  \
       -output    /tmp/run.csh.IeU/output.raw \
       -out1          /tmp/run.csh.IeU/onebit.raw \
       -delay         0,0 \
       -input-size    8 \
       -output-size   8 \
       -nclocks   1M
```


#### Example (pointwise)

## Notes
This is what is currentlyin toolchain overview:

<b>6. Simulator ("run_tbg.csh")</b> runs the app and emits a result for comparison
```
  git clone https://github.com/StanfordAHA/CGRAGenerator
  cd CGRAGenerator/verilator/generator_z_tb; ./run_tbg.csh \
       -config    <fullpath>/pointwise_pnr_bitstream \
       -io_config <fullpath>/pointwise_io.json \
       -input     <fullpath>/pointwise_input.png \
       -output    <fullpath>/pointwise_CGRA_out.raw \
       -out1      <fullpath>/pointwise_CGRA_out1.raw \
       -delay     0,0 \
       -nclocks    5M`
```

#### Sample run_tbg output




