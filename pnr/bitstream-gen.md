# Bitstream Generation
Given packing, placement, and routing result, we can finally write the
bitstream! In `cgra_pnr` I adopt Steve's simple assembly language, which his
`bsbuilder` tool can compile into an
actual bitstream. We will call the language `bsb,` short for `bitstream
builder format`. You can see more details about `bsb` in the
[BSB-Format section](../hardware/bsb-format.md).

There are several reasons to use `bsb` files:
1. It's human-readble and editable. So it's easy to spot any error and correct
the bitstream by hand.
2. `bsbuider` has extensive checks for routings. So it's very helpful to debug
the router and the bitstream generator, especially in the early stage.
Although the PnR toolchain also has some self-check, it's also a good idea to
have a third-party checking your result.
3. The PnR toolchain does not need to worry about the underlying PE/SB/CB
encoding change as it is now `bsbuider's` job to keep that updated.

## Implementation Notes
Because of the way `bsb` configures switchboxes, the routing format can not be translated
to `bsb` directly. For instance, the routing format is `out -> in`, whereas
the `bsb` uses `in -> out` on the same tile. In addition, because the router
uses MST to minimize the wirelength, there will be some *jumps* in the
routing result. The bitstream generator is required to recognize the jump
and figure out where tile makes the jump and reconstruct `in -> out` format.

## Example usage

<pre>
% CGRAGenerator/bitstream/bsbuilder/bsbuilder.py \
    < pointwise_annotated.bsb \
    > pointwise_pnr_bitstream
</pre>

where

* `pointwise_annotated.bsb` is the bsb file input and
<pre>
    # INPUT::mul_347_348_349_PE.data.in.0
    Tx0101_in_s3t0 -> Tx0101_out_s2t0
    Tx0101_out_s2t0 -> Tx0101_op1

    # mul_347_348_349_PE::OUTPUT
    Tx0101_pe_out -> Tx0101_out_s0t0
    Tx0102_in_s2t0 -> Tx0102_out_s0t0
    ...
</pre>

* `pointwise_pnr_bitstream` is the bitstream output
<pre>
    F1000101 00000002
    # data[(15, 0)] : init `data1` reg with const `2`

    FF000101 0002F00B
    # data[ 5,  0] = 11 :: alu_op 'mult_0'
    # data[ 6,  6] =  0 :: sign   'u'
    # data[15, 12] = 15 :: flag   'pe'
    # data[17, 16] =  2 :: data0  'wire_a' (REG_BYPASS)
    # data[19, 18] =  0 :: data1  'const_b' (REG_CONST)

    00020101 00000005
    # data[(3, 0)] : @ tile (1, 1) connect wire 5 (out_BUS16_S2_T0) to data0
</pre>

