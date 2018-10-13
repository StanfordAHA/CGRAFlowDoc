# Understanding Halide Schedules for describing Hardware
(keyi: copied from Kayvon's Halide to Hardware Primer)

We'll use convolution with a 3x3 support as the running example.

```C++
Var x, y, xi, yi, xo, yo;
ImageParam input;
Func conv, output;
RDom win(0, 3, 0, 3);   // domain for 3x3 convolution support window

conv(x, y)  = 0.0;
conv(x, y) += input(x + win.x, y + win.y) * weights(win.x, win.y);
output(x, y) = conv(x, y);
```

## Schedule 1: One Output Pixel Per Clock Hardware
First let’s review the semantics of a basic, one output-pixel-per-clock schedule.

```C++
1:  output.tile(x, y, xo, yo, xi, yi, 64, 64);
2:  output.hw_accelerate(xo, xi);
3:  conv.compute_at(output, xi)
4:      .update(0).unroll(win.x).unroll(win.y);
5:  input.in()             // lines 5-8 are equiv to: input.in().linebuffer(output, xo)
6:    .store_in(Linebuffer)
7:    .store_at(output, xo)
8:    .compute_at(output, xi);
9:  weights.in()
10:   .store_in(Register)
11:   .compute_root();
```

The schedule is written in full above, so let’s break down the parts one by one.
### Step 1: Defining the Hardware’s Interface to the Host, and the Granularity of Dispatch
```C++
output.tile(x, y, xo, yo, xi, yi, 64, 64)
      .hw_accelerate(xo, xi);
```
The declaration above creates a tiled loop nest (64x64 tiles), and then
describes how the host offloads work to the hardware in terms of these loops. 

(keyi: I think Kayvon/Jeff should decide whether to convert the primer or
write it from scratch...)
