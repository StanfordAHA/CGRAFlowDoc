CGRA generation should happen *first* as it dictates the design
information that will be used by all succeeding steps (except the
Halide front end, which compiles to design-independent intermediate
form).

## Usage

  git clone https://github.com/StanfordAHA/CGRAGenerator
  CGRAGenerator/hardware/generator_z/top/build_cgra.sh

## What it does

The `build_cgra.sh` invokes Genesis2 to build a complete CGRA design
including SystemVerilog description along with machine-readable
collateral `cgra_info.xml` containing design-specific parameters for
use by tools targeting the design.

Specifically `build_cgra` does the following:
* locate Genesis2 on the host machine, else install Genesis2 as necessary
* implement verilator hacks if necessary
* execute Genesis2 to build the design as elaborated SystemVerilog files in a subdirectory `genesis_verif`
* where available, uses `xmllint` to perform a sanity check on the generated `cgra_info.xml`

#### Verilator hacks

Verilator does not understand tristate buffers; therefore, by default,
the `build_cgra` script sets an environment variable such that the
design uses separate `in` and `out` ports instead of `inout` tristate
pads for I/O.  

Also: certain process-specific modules, such as the SRAM and the
proprietary JTAG tap, are unavailable to Verilog, so the script will
substitute generic equivalents in their place.

The user can suppress or enable this behavior by using command
line switches `--use_verilator_hacks` or `--no_verilator_hacks.`

If `build_cgra` detects that it is running on travis or kiwi, it
assumes Verilator usage and implements the hacks by default.  Again,
this behavior can be overriden by the appropriate command-line switch.


#### Genesis2

Given a list of Genesis2 verilog-mixed-with-perl source modules, plus
optional user-specified parameters, Genesis2 builds a complete
elaborated Verilog design along with an exhaustive list of the
parameters it used.  As of this writing, the `build_cgra` script
invokes the following Genesis2 default command

  Genesis2.pl -parse -generate -top top -hierarchy top.xml \
    -xml ./bin/shortmem.xml \
    -input \
    top.vp \
    cgra_core.vp \
    \
    ../pad_ring/pad_ring.svp \
    ../pad_ring/io_group.svp \
    ../pad_ring/fixed_io.vp \
    \
    ../sb/sb.vp \
    ../cb/cb.vp \
    \
    ../pe_new/pe/rtl/test_pe_red.svp \
    ../pe_new/pe/rtl/test_pe_dual.vpf \
    ../pe_new/pe/rtl/test_pe_comp.svp \
    ../pe_new/pe/rtl/test_pe_comp_dual.svp \
    ../pe_new/pe/rtl/test_cmpr.svp \
    ../pe_new/pe/rtl/test_pe.svp \
    ../pe_new/pe/rtl/test_mult_add.svp \
    ../pe_new/pe/rtl/test_full_add.svp \
    ../pe_new/pe/rtl/test_lut.svp      \
    ../pe_new/pe/rtl/test_opt_reg.svp  \
    ../pe_new/pe/rtl/test_simple_shift.svp \
    ../pe_new/pe/rtl/test_shifter.svp  \
    ../pe_new/pe/rtl/test_debug_reg.svp  \
    ../pe_new/pe/rtl/test_opt_reg_file.svp  \
    \
    ../pe_tile_new/pe_tile_new.svp \
    \
    ../empty/empty.vp \
    ../io1bit/io1bit.vp \
    ../io16bit/io16bit.vp \
    ../global_signal_tile/global_signal_tile.vp \
    \
    ../memory_tile/memory_tile.vp \
    ../memory_tile/mem_shift_reg.svp \
    ../memory_core/input_sr.vp \
    ../memory_core/output_sr.vp \
    ../memory_core/linebuffer_control.vp \
    ../memory_core/fifo_control.vp \
    ../memory_core/mem.vp \
    ../memory_core/memory_core.vp \
    \
    ../global_controller/global_controller.svp \
    \
    ../jtag/jtag.svp \
    ../jtag/Template/src/digital/template_ifc.svp \
    ../jtag/Template/src/digital/cfg_ifc.svp \
    ../jtag/Template/src/digital/flop.svp \
    ../jtag/Template/src/digital/tap.svp \
    ../jtag/Template/src/digital/reg_file.svp \
    ../jtag/Template/src/digital/cfg_and_dbg.svp \
    || exit 13

Pre-built configuration files `shortmem.xml` and `tallmem.xml` contain
user parameters that can direct Genesis2 to build a shortmem design
(where memory tiles are the same height as ALU tiles) or a tallmem
design (where memory tiles are twice as high as PE tiles).

By default `build_cgra` produces a shortmem design but the user can
control this using command line switches `--tallmem` or `--shortmem.`

### cgra_info

[FIXME/TBD should probably describe cgra_info.xml file here i guess?]

