# Overview
Stages of harness generation
* define tracing macros (if enabled)
* chip reset (based on pnr collateral)
* configuration (based on use_jtag)
* (optional) verify config (use_jtag only)
* main loop 
  * stream input file data to input pads
  * step the global clock
  * stream output pad values to output files

## Details
Generating the harness is divided into the metaprogramming of 11 specific
program components:
* [Includes](#includes)
* [trace setup](#trace-setup) (`--trace` only)
* [file setup](#file-setup)
* [jtag setup](#jtag-setup)
* [chip init](#chip-init)
* [chip reset](#chip-reset)
* [chip stalling (`--use-jtag` only)](#chip-stalling---use-jtag-only)
* [running configuration](#running-configuration)
* [verifying configuration (`--use-jtag` and `--verify-config` only)](#verifying-configuration---use-jtag-and---verify-config-only)
* [chip unstalling/reset](#chip-unstallingreset)
* [clock switch (`--use-jtag` only)](#clock-switch---use-jtag-only)
* [running test](#running-test)

### Includes
This sets up include files used by the harness. All generated test benches will
include verilator's header file (`verilated.h`) and various standard library
headers (`<iostream>`, `stdint.h`, `<fstream>`, `printf.h`).  The one
dynamically generated include is for the verilator module `V{wrapper_name}.h`
where `wrapper_name` corresponds to the name of the top module in the generated
Verilog.

### Trace Setup
If `--trace` is enabled, the harness will append `verilated_vcd_c.h` to the
include files, as well as some code to setup the tracing logic.  In particular,
it calls the verilator function `Verilated::traceEverOn(true)` to configure
verilator to perform tracing. Then, it sets up a VCD file for verilator to dump
the trace too, configured by the `--trace-file-name` parameter, and adds the
code to close the trace file during cleanup. It also sets up a `time_step`
variable used to track the current time step during test execution.  Finally,
the `next` and `step` macros are redefined to include the required method calls
to dump the current state of the simulation to the trace file for each clock
step.

### File Setup
This code handles the opening of `fstream` objects for the input and output
files specified by the PnR collateral.  The `--input-chunk-size` and
`--output-chunk-size` parameters specify the size in bits of the data in the
input and output files (**TODO: It would be nice to generalize this over
multiple input and output files, or have it specified by some input
collateral**).

### JTAG Setup
If `--use-jtag` is enabled, the harness includes `jtagdriver.h` which defines
the interface to the C++ JTAG driver.  It also adds an instantiation of the
`JTAGDriver` object which will be used by the configuration code.

### Chip Init
This phase initializes the `clk_pad` and `reset_pad` inputs to be zero. If
`--use-jtag` is enabled, it also calls `jtag.init()` which initializes the jtag
ports to zero. Then, it calls `eval` once.

### Chip Reset
This phase resets the chip by setting `reset_pad` to 1, calling `eval`, setting
`reset_pad` to 0, and calling `eval`.  If `--use-jtag` is enabled, it will
reset the chip using the `trst_n` port, and then bring up the JTAG clock by
setting `tck` to 1 and stepping twice.

### Chip Stalling (`--use-jtag` only)
If `--use-jtag` is enabled, this phase will stall the chip by writing `0xF` to
the `OP_WRITE_STALL` register of the JTAG TAP.

### Running Configuration
This phase loops over the bitstream and sends the commands to configure the
chip.  If `--use-jtag` is enabled, it will use the `JTAGDriver` to configure
the chip with the `write_config` method. Otherwise, it will directly poke the
`config_data_in` and `config_addr_in` ports and step the clock twice.

### Verifying configuration (`--use-jtag` and `--verify-config` only)
If `--use-jtag` and `--verify-config` are enabled, this phase will read the
configuration state back (inverse of the configuration phase) to verify that
the chip has been properly configured through the global controller.

### Chip Unstalling/Reset
If `--use-jtag` is enabled, this phase will unstall the chip by writing `0x0`
to the `OP_WRITE_STALL` register of the JTAG TAP.

Otherwise, it sets the `reset_in_pad` provided by the PnR collateral to 1,
which resets the chip state.

### Clock Switch (`--use-jtag` only)
If `--use-jtag` is enabled, this phase switches the clock from the test clock
(used for configuration) to the fast clock (used for normal operation).  The
`JTAGDriver` provides a method `switch_to_fast`.

### Running Test
To actually run the test, the harness generates a loop that streams data from
the input files, pokes the values onto the input pads specified by the PnR
collateral, steps the clock, streams the output pad values to the output files,
and steps the clock.  This loop runs until any of the input file streams close
(run out of data).
