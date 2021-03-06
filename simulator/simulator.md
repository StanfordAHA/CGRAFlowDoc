# Simulator (run_tbg.csh)

The simulator takes a CGRA bitstream, plus an input image, and
produces an output image.  The output image is supposed to be the same
as actual CGRA hardware would build.

`run_tbg.csh` is the driver for the simulator.  Its main
purpose is to call the [Test Bench Generator](tbg/intro.md) (TBG).
Given a `bsa` bitstream from the
[Assembler](bsbuilder/bsbuilder.md), plus verilog from the [CGRA
Generator](cgra/cgra-generator.md), TBG builds a custom testbench.

Making heavy use of [TBG](tbg/intro.md) scripts, `run_tbg.csh` invokes the
generated testbench using a given input image, and thereby generates
the output image and/or other collateral.

In more detail, `run_tbg.csh` currently does the following:
* verifies and echoes the current github branch;
* verifies and echoes essential details of the target CGRA design (i.e. memheight);
* cleans (removes comments from) and verifies validity of bitstream config file;
* reorders the bitstream to prevent lockup (see
[here](https://github.com/StanfordAHA/CGRAGenerator/wiki/Bitstream-Encoding#important-note-on-bitstream-ordering)
and below.
* optionally calls the [CGRA Generator](cgra/cgra-generator.md) to (re)build a verilator-friendly verilog from scratch;
* calls  [TBG](tbg/intro.md) `process_input` script to shape the input image according to DELAY paramater;
* uses generated verilog, plus the bitstream, to build a verilator testbench;
* runs the testbench on the input image to make an output image.
* calls  [TBG](tbg/intro.md) `process_output` script to shape the output image according to DELAY paramater;

### IO configuration

Along with the bitstream, `run_tbg` needs an IO-configuration
descriptor to set up inputs and outputs, i.e. something that would say
that the sixteen pads on the north side of the chip are being used as
a 16-bit input bus.  This configuration file should be supplied by
whoever created the bitstream, e.g. PNR.  

Here is a sample IO config file.

```
% less io/2in2out.json
{
    "reset_in_pad": {
        "pad_bus" : "pads_W_0",
        "bits": {
            "0": { "pad_bit":"0" }
        },
        "mode": "reset",
        "width": 1
    },
    "io16in_in_arg_1_0_0": {
        "pad_bus" : "pads_N_0",
        "mode": "in",
        "width": 16
    },
    "io16_out_0_0": {
        "pad_bus" : "pads_E_0",
        "mode": "out",
        "width": 16
    },
    "io1_out_0_0": {
        "pad_bus" : "pads_S_0",
        "bits": {
            "0": { "pad_bit":"0" }
        },
        "mode": "out",
        "width": 1
    }
}
```

### Reordering the bitstream

Bitstream configuration order is important!

1. If a LUT is set up as an inverter, it is easy during routing for
the wires to pass through a state such that the inverter output is
connected directly to its input, which makes the simulator hang
forever waiting for the value to stabilize to a 1 or 0 (this has
actually happened).

2. It's bad if e.g. a linebuffer's write-enable signal `WEN` wiggles off
and back on again after the linebuffer has been set up, which can
happen as wires get unconnected and reconnected during the routing
portion of the bitstream setup. It messes up the internal state of the
linebuffer and you wind up with e.g. a 7-deep linebuffer instead of
the 10-deep that you originally configured (this has also happened).


So now the rule for configuration is a) do the switchbox and
connection box wiring FIRST and then b) do the tile setup for LUTs,
memories (e.g. WEN), ALU ops etc. For this reason there is a csh script
`reorder.csh` that takes any bitstream config file `config.bs` and
transforms it for the proper order (ish).

Also see [here](https://github.com/StanfordAHA/CGRAGenerator/wiki/Bitstream-Encoding#important-note-on-bitstream-ordering).


### Verilator hacks

The tapeout version of the chip contains proprietary modules for
e.g. SRAM and JTAG drivers, plus it has tri-state buffers for the
programmable IO pads.  Unfortunately, we have not been able to make
Verilator work with tri-state buffers, and proprietary modules are
verboten to use where there is no NDA protection (e.g. github
repositories and/or our development server "kiwi."

Therefore!  When "run_tbg" detects that it is running in a
non-proprietary environment it makes certain changes to the design:

* each tri-state IO pad is replaced by separate 'input' and 'output' pads
* proprietary modules (SRAM, JTAG driver) are replaced by handwritten stubs with equivalent functionality


### run.csh vs. run_tbg.csh

`run.csh` is the deprecated older version of `run_tbg.csh.` It uses a
handwritten test bench that is compiled per CGRA design, and which can
then be used with any bitstream, as opposed to an automatically
generated testbench customized per bitstream as with TBG.


## Usage and defaults (--help)

```
./run_tbg.csh --help
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
       -io_config io/2in2out.json \
       -input     io/conv_bw_in.png  \
       -output    /tmp/output.raw \
       -out1      /tmp/onebit.raw \
       -delay         0,0 \
       -input-size    8 \
       -output-size   8 \
       -nclocks      1M
```


## Sample run_tbg output

```
run_tbg.csh: I think we are in branch 'genspec'
run_tbg.csh: Looks like memtile_height is 1

Running with the following switches:
./run_tbg.csh -v \
   -gen \
   -config    ../../bitstream/examples/pointwise.bsa \
   -io_config io/2in2out.json \
   -input     io/conv_bw_in.png \
   -output    /tmp/run.csh.fYN/output.raw \
   -out1      /tmp/run.csh.fYN/onebit.raw \
   -delay     0,0 \
   -nclocks    1M

bin/reorder.csh /tmp/pointwise.bs > /tmp/pointwise_reordered.bs

run_tbg.csh: Building CGRA because it's the default...
run_tbg.csh: ../../hardware/generator_z/top/build_cgra.sh
./build_cgra.sh WARNING I think we are running from kiwi;
                setting USE_VERILATOR_HACKS

NOTICE Building shortmem design

--------------------------------------------------------------------
Here is what I built (it's supposed to look like an array of tiles).
// mem_tile_height (_GENESIS2_DECLARATION_PRIORITY_) = 1

//CGRA      00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 10 11
//CGRA   00 .. .. .. .. .. .. .. .. .. .. .. .. .. .. .. .. .. ..
//CGRA   01 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   02 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   03 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   04 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   05 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   06 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   07 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   08 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   09 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   0A .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   0B .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   0C .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   0D .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   0E .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   0F .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   10 .. p  p  p  m  p  p  p  m  p  p  p  m  p  p  p  m  ..
//CGRA   11 .. .. .. .. .. .. .. .. .. .. .. .. .. .. .. .. .. ..
--------------------------------------------------------------------

run_tbg.csh: Building the verilator simulator executable...
build_simulator_tbg.csh -v \
  pointwise_reordered.bs 2in2out.json \
  conv_bw_in.png output.raw 8 8

build_simulator_tbg.csh: Generating the harness
python3 $dir/generate_harness.py \
  --pnr-io-collateral io/2in2out.json \
  --bitstream /tmp/pointwise_reordered.bs \
  --max-clock-cycles 5000000 \
  --output-file-name build/harness.cpp \
  --input-chunk-size 8 --output-chunk-size 8

Building simulator source files...
verilate.py \
  --harness harness.cpp          \
  --verilog-directory ../../hardware/generator_z/top/genesis_verif/   \
  --output-directory build       \
  --top-module-name top          \

Found an existing verilator binary, skipping

make -C build -j -f Vtop.mk Vtop

  First prepare input and output files...
BITSTREAM '/tmp/pointwise_reordered.bs':
00080101 00200003
00020101 00000005
...


python3 process_input.py io/2in2out.json /tmp/pw_in.raw 0,0
    Done resetting
    Beginning configuration
    Done configuring
    Running test
    Cycle: 1000
    Cycle: 2000
    Cycle: 3000
    Cycle: 4000
    Reached end of file io16in_in_arg_1_0_0.raw
    Done testing

python3 $TBG/process_output.py io/2in2out.json /tmp/output.raw pw 0,0

INPUT  od -t u1 /tmp/pw_in.raw
0000000  95  95  98  89  98 103  95  97  93  92  90  89  84  83  81  82
0000020  94  91  87  81  96  88  91  86  83  80  91  91  81  83  87  84
...

OUTPUT od -t u1 /tmp/1output.raw
0000000 190 190 196 178 196 206 190 194 186 184 180 178 168 166 162 164
0000020 188 182 174 162 192 176 182 172 166 160 182 182 162 166 174 168
...

```



