# Hardware Lowering
There are a pair of lowering passes that are unique to hardware. The current
implementation uses two custom passes to achieve this; however, in the 
future some of the more recent analyses might be adopted to perform these
passes in a more robust fashion.

## Linebuffer Extraction
In this pass, the parameters for the linebuffers are lifted from the usage
of the functions. This analysis must:
- identify where the user asked for a linebuffer
- image size (what is the width of the input image)
- output stencil size (i.e., width and height of the convolution kernel)
- input pixel rate (how many pixels are inputted each cycle)

The location of a linebuffer is found by matching the loop variables
annotated by the `accelerate` scheduling primitive when defining the 
hardware accelerator. By matching the `store_at` and `compute_at` variables,
we know that the storage can be optimally created as a linebuffer.

The image size is identified by Halide's bounded-size analysis pass. This
pass traces from the output to the desired function to find what footprint
is needed for the function. This defines the `Func`'s minimum and maximum
`x` and `y` index. The maximum width is then used to define the linebuffer
row's width.

The output stencil size is needed to define how many values are needed on
each iteration. These values are typically a rectangular region of the image,
and in hardware must be available on the cycle. This parameter is used to
control how many registers are created between each rowbuffer. Since a
rectangular region is expected, the box-touched analysis pass can be used
to find which pixel indices are used in each cycle. This box should match
the reduction domain's size for a convolution.

Finally, the input pixel rate defines how many pixels are inserted into
the linebuffer on each cycle. For every cycle iteration, there are a constant
number of pixels that are produced. Based on this constant rate, there is
some overlap of the pixels used from previous iterations, but also some new
pixels. This pass finds how many new pixels are needed by comparing the
difference between iteration `x` and iteration `x+1`. The number of pixels
between these two iterations is expected to be a constant; if not the
compiler throws an error, since a linebuffer cannot satisfy a varying
number of input pixels on different cycles.

## Stream Optimization
With the linebuffers discovered from the previous analysis, the HalideIR 
must be modified to use these new memories. Arrays are replaced, and indexing
is replaced with new indices into the linebuffer output stencil.

In addition, the Halide compiler usually produces sequential code for CPU execution.
However, for hardware, all code segments at the iteration variable execute
simultaneously. A stream of input pixels and stream of output pixels are
expected enter and exit the accelerator. The HalideIR is modified to match
how hardware behaves with streaming operators.