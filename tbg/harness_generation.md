Stages of harness generation
* define tracing macros (if enabled)
* chip reset (based on pnr collateral)
* configuration (based on use_jtag)
* (optional) verify config (use_jtag only)
* main loop 
  * stream input file data to input pads
  * step the global clock
  * stream output pad values to output files
