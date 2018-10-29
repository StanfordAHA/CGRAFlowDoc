The Test Bench Generator provides two scripts for processing the input and
output files:
* [process_input.py](https://github.com/StanfordAHA/TestBenchGenerator/blob/master/process_input.py)
* [process_output.py](https://github.com/StanfordAHA/TestBenchGenerator/blob/master/process_output.py)

These scripts consume the PnR collateral (`.io.json`) and performs the renaming
of input and output files to files that match the mapped pad name.  This is
because the generated test bench harness expects the input and output files to
match the pad names.  These scripts also handle a `DELAY_IN` and `DELAY_OUT`
parameter which handles padding inputs and trimming outputs to handle delay in
the design.
