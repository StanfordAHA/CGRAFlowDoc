# Tool Chain Overview

[And/or should this be part of the top-level intro?]

<!-- The steps required to build and run an application are as follows: -->

The procedure for turning a Halide app into a program running on a
CGRA accelerator is shown below.  This example uses the "pointwise
multiply-by-two' Halide example from Jeff Setter's repository.  The
app simply reads an input image, multiplies every pixel in the image by two,
and emits that as the output image.


<b>1. CGRA generator</b> builds a target CGRA
    
    `git clone https://github.com/StanfordAHA/CGRAGenerator`<br/>
    `CGRAGenerator/hardware/generator_z/top/build_cgra.sh`

    CGRA generation should happen *first* as it dictates the design
    information that will be used by all succeeding steps (except the
    Halide front end, which compiles to design-independent intermediate
    form).
    
    
2. <b>Halide compiler</b> reads your Halide source app code and
produces design-independent CoreIR intermediate code
    
    `git clone https://github.com/jeffsetter/Halide_CoreIR`
    `make -C Halide_CoreIR/apps/coreir_examples/pointwise/ pointwise_design_top.json`
    
    See directory
    `https://github.com/jeffsetter/Halide_CoreIR/apps/coreir_examples/pointwise/`
    to see how this Halide app was set up.
    
    
3. <b>CoreIR mapper</b> reads design-independent CoreIR and emits target-specific CoreIR
    
    `git clone ...`
    `CGRAMapper/bin/cgra-mapper pointwise_design_top.json pointwise_mapped.json`
    
    <i>[Note the mapper does not appear to use CGRA-dependent info from
    the cgra_info.xml file...but it should...right?]</i>
    
    
4. <b>Place-and-route (PNR)</b> turns target-specific CoreIR into CGRA assembly language
    
    `...`
    `pnr_flow.sh cgra_info.xml pointwise_mapped.json pointwise_annotated.bsb`
    
    
5. <b>Assembler ("bsbuilder")</b> turns assembly code into a CGRA configuration bitstream
    
    `git clone https://github.com/StanfordAHA/CGRAGenerator`<br/>
    `CGRAGenerator/bitstream/bsbuilder/bsbuilder.py < pointwise_annotated.bsb > pointwise.bsa`
    
6. <b>Simulator ("run.csh")</b> runs the app and emits a result for comparison
    
    `git clone https://github.com/StanfordAHA/CGRAGenerator`<br/>
    `cd CGRAGenerator/verilator/generator_z_tb; ./run_tbg.csh \
       -config    <fullpath>/pointwise_pnr_bitstream \
       -io_config <fullpath>/pointwise_io.json \
       -input     <fullpath>/pointwise_input.png \
       -output    <fullpath>/pointwise_CGRA_out.raw \
       -out1      <fullpath>/pointwise_CGRA_out1.raw \
       -delay     0,0 \
       -nclocks    5M`


\7. test

\8. text

