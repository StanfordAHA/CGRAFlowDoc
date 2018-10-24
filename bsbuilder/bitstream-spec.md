# Bitstream Encoding for CGRA Configuration

The actual encoding for the configuration bitstream changes according
to the specific CGRA design that was generated.  See the
<a href="https://github.com/StanfordAHA/CGRAGenerator/wiki/PE-Spec">PE
Spec</a> for details.

As an example, here is how pointwise multiply-by-two gets encoded for
a CGRA "shortmem" design:

```
% cat $bsb/testdir/examples/pointwise.bsa
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

00080101 00200003
# data[(1, 0)] : @ tile (1, 1) connect wire 3 (pe_out_res) to out_BUS16_S0_T0
# data[(21, 20)] : @ tile (1, 1) connect wire 2 (in_BUS16_S3_T0) to out_BUS16_S2_T0

00080102 00000001
# data[(1, 0)] : @ tile (1, 2) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

00080103 00000001
# data[(1, 0)] : @ tile (1, 3) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

00010104 00000001
# data[(1, 0)] : @ tile (1, 4) connect wire 1 (in_0_BUS16_S2_T0) to out_0_BUS16_S0_T0

00080105 00000001
# data[(1, 0)] : @ tile (1, 5) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

00080106 00000001
# data[(1, 0)] : @ tile (1, 6) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

00080107 00000001
# data[(1, 0)] : @ tile (1, 7) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

00010108 00000001
# data[(1, 0)] : @ tile (1, 8) connect wire 1 (in_0_BUS16_S2_T0) to out_0_BUS16_S0_T0

00080109 00000001
# data[(1, 0)] : @ tile (1, 9) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

0008010A 00000001
# data[(1, 0)] : @ tile (1, 10) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

0008010B 00000001
# data[(1, 0)] : @ tile (1, 11) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

0001010C 00000001
# data[(1, 0)] : @ tile (1, 12) connect wire 1 (in_0_BUS16_S2_T0) to out_0_BUS16_S0_T0

0008010D 00000001
# data[(1, 0)] : @ tile (1, 13) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

0008010E 00000001
# data[(1, 0)] : @ tile (1, 14) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

0008010F 00000001
# data[(1, 0)] : @ tile (1, 15) connect wire 1 (in_BUS16_S2_T0) to out_BUS16_S0_T0

00010110 00000001
# data[(1, 0)] : @ tile (1, 16) connect wire 1 (in_0_BUS16_S2_T0) to out_0_BUS16_S0_T0

# INPUT  tile 257 ( 1, 1) / in_BUS16_S3_T0 / wire_0_1_BUS16_S1_T0
# OUTPUT tile 272 ( 1,16) / out_0_BUS16_S0_T0 / wire_1_16_BUS16_S0_T0

# WARNING You did not designate a 16-bit output bus, so I will build one: 
# Configuring side 0 (right side) io1bit tiles as 16bit output bus
00000111 00000001
00000211 00000001
00000311 00000001
00000411 00000001
00000511 00000001
00000611 00000001
00000711 00000001
00000811 00000001
00000911 00000001
00000A11 00000001
00000B11 00000001
00000C11 00000001
00000D11 00000001
00000E11 00000001
00000F11 00000001
00001011 00000001
```