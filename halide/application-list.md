# Application List
Below are the current set of test cases and applications. Test cases are smaller
in size and scope, and should be used to find and isolate bugs in the full system.
The test cases from `abs` to `ternary` are meant to cover all expected operators
used in Halide. Applications are larger whose output image are attempting to 
accomplish some more meaningful task.

The status of each of the following test cases and applications can be found
here: 
https://docs.google.com/spreadsheets/d/1GLqlCGaTXMDLW3QqCXK0XX1zOzP0UyoJ9eXDo75xpDg

## Test Cases
|Test Name          | Test Case                                                       |
|-------------------|-----------------------------------------------------------------|
|pointwise          | multiply by 2                                                   |
|conv_1_1           | convolution with kernel size 1x1 (same as pointwise)            |
|conv_1_2           | convolution with kernel size 1x2 (one register)                 |
|conv_2_1           | convolution with kernel size 2x1 (one rowbuffer)                |
|conv_3_1           | convolution with kernel size 3x1 (two rowbuffers)               |
|conv_3_3           | convolution with kernel size 3x3 (regs and rbs)                 |
|conv_chain         | parameterizable design of N convs with W kernel size            |
|conv_multi         | 3x3 convolution with 3 multipliers (not fully unrolled)         |
|absolute           | Tests `abs`, `absd `                                            |
|arith              | Tests `*`  `/`  `%`  `+`  `-`                                   |
|bitwise            | Tests `&`  `~`  <code>&#124;</code>  `^`                        |
|boolean_ops        | Tests `&&`  <code>&#124;&#124;</code> `^`  `!`                  |
|counter            | Tests counter                                                   |
|divide             | Tests `/` and `%`                                               |
|equal              | Tests `==`  `!=`                                                |
|multiply           | Tests `*`  `mult_middle` `mult_high`                            |
|inout_onebit       | Tests one bit input and output                                  |
|scomp              | Tests `<`  `>`  `<=`  `>=` using signed numbers                 |
|ucomp              | Tests `<`  `>`  `<=`  `>=` using unsigned numbers               |
|sminmax            | Tests min, max, clamp using signed numbers                      |
|uminmax            | Tests min, max, clamp using unsigned numbers                    |
|sshift             | Tests `<<`  `>>` using signed numbers                           |
|ushift             | Tests `<<`  `>>` using unsigned numbers                         |
|ternary            | Tests MAD, MUX (select)                                         |

## Applications
| Application Name  | Functionality                                                                                            |
|-------------------|----------------------------------------------------------------------------------------------------------|
|gaussian           | 3x3 convolution using normalized values                         |
|cascade            | two back-to-back convolutions                                   |
|harris             | corner detector                                                                                          |
|harris_valid       | corner detector with valid output                                                                        |
|fast_corner        | corner detector using FAST algorithm (sixteen comparisons)                                               |
|unsharp            | Mask to sharpen the image                                                                                |
|demosaic           | Applies demosaic to input to create rgb image                                                            |
|demosaic_harris    | Demosaic followed by harris corner detector                                                              |
|demosaic_flow      | Demosaic followed by optical flow, pixel frame movement                                                  |
|stereo             | Creates depth map from images from two viewpoints                                                        |
|camera_pipeline    | Camera pipeline with hot pixel suppression, deinterleave, demosaic, color correct, gamma correction      |
|canny              | Edge detection                                                                                           |
|bilateral_filter   | Perform edge-preserving blur                                                                             |
|optical_flow       | Pixel movement between two frames                                                                        |
|seedark            | Algin, warp, and blend underexposed images for low light                                                 |

