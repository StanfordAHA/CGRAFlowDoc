# Introduction to BSB format

`bsb` (bitstream-builder format) files are emitted by `cgra_pnr.`
`bsb` files are used by the `bsbuilder` tool to produce CGRA
configuration bitstreams.  In general, like most assembly languages,
one line of bsb compiles into exactly one CGRA machine instruction
(or sometimes just a single field within the instruction).

There are several sections in `bsb` files, including

+ Placement
+ IO
+ Routing

## Placement Section
An example of the placement section is shown below.  Note, any text
following a pound-sign (`#`) is treated as a comment (i.e. it is ignored).
```
Tx0102_add(wire,wire)                       # add_704_707_708$binop
Tx0105_sle(wire,const255_255)               # smin_689_690_691$scomp$compop
Tx0106_uge(const59_59,wire)                 # lb_pcus$valcounter_1$ult$comp$compop
Tx0107_add(wire,wire)                       # add_762_763_764$binop
Tx0109_sub(wire,wire)                       # sub_686_688_689$binop
Tx010A_mux(wire,const255_255,wire)          # smin_689_690_691$min_mux$mux
Tx010B_lut88(wire,wire,const0_0)            # lb_pcus$valid_andr$_join$lut$lut
Tx010D_lut55(wire,const0_0,const0_0)        # lb_plsus$valcounter_1$ult$not$lut$lut
Tx010E_mux(wire,const255_255,wire)          # smin_661_662_663$min_mux$mux
Tx0201_add(const0_0,reg)                    # add_704_705_706$binop
```

The tile number `TILE_NUM` in each `Tx{TILE_NUM}_{OP}` indexes a
specific tile described in the `cgra_info` file that was produced when
the CGRA was generated.
In recent designs, `TILE_NUM` is a 16-bit number where the first and
second eight-bit quantities respectively indicate the tile's row and
column position.  Also see the 
<a href="https://github.com/StanfordAHA/CGRAGenerator/wiki/PE-Spec#tile_number">
PE Spec</a>.

The opcode `{OP}` in each `Tx{TILE_NUM}_{OP}` designation is given in
the input netlist JSON file. Operands `({CONN1}, {CONN2}, {CONN3})`
describe the connectivity of the tile. Please notice that for binary
ops, such as `add`, there are only two connections, whereas for
`mux`, there are three wires connected. In `mux`, the last wire is the
`sel` signal. If there is a constant folded in one of its operands,
it will have `const{value}_{anystring}` in lieu of the wire, where
`{value}` specifies the constant value. If a register is folded to the
tile, `reg` will be used instead.

A list of supported ops can be found in the CGRA 
<a href="https://github.com/StanfordAHA/CGRAGenerator/wiki/PE-Spec#alu_ops">
PE Spec</a>
that is auto-generated each time a design is built.  The basic ops include
`add, sub, abs, gte_max, lte_min, sel, mult_0, mult_1, mult_2, rshft,
lshft, or, and,` and `xor.`
Each op can be optionally prepended by `u` or `s` to indicate signed or unsigned,
and optionally suffixed by a flag-set operation, e.g. `Tx0102_usub.ge`
sets up the tile in row 1, column 2 to do an unsigned subtract and
then set the GE flag.  Supported flags are found in the
<a href="https://github.com/StanfordAHA/CGRAGenerator/wiki/PE-Spec#pe_flags">
PE Spec</a> and are similar to those available in the ARM
architecture,
including `eq, ne, cs, cc, mi, pl, vs, vc, hi, ls, ge, lt, gt,` and `le.`

`Bsbuilder` also supports a number of convenient aliases e.g.

```Python
ALIAS['eq'] = 'sub.eq'

ALIAS['gte'] = 'sub.ge'
ALIAS['ge']  = 'sub.ge'

ALIAS['lte'] = 'sub.le'
ALIAS['le']  = 'sub.le'

ALIAS['gt']  = 'sub.gt'
ALIAS['lt']  = 'sub.lt'

ALIAS['max'] = 'gte_max'
ALIAS['min'] = 'lte_min'

ALIAS['mul'] = 'mult_0'
ALIAS['mux'] = 'sel'
```


## IO Section
Because the CGRA chip has two different width of tracks, i.e. 16-bit and 1bit,
we need to specify different IO pad information. Below is an example of how
IO pads are configured to have 16-bit and 1bit input/output.
```
Tx1101_pad(out,16)
Tx1102_pad(out,16)
Tx1103_pad(out,16)
Tx1104_pad(out,16)
Tx1105_pad(out,16)
Tx1106_pad(out,16)
Tx1107_pad(out,16)
Tx1108_pad(out,16)
Tx1109_pad(out,16)
Tx110A_pad(out,16)
Tx110B_pad(out,16)
Tx110C_pad(out,16)
Tx110D_pad(out,16)
Tx110E_pad(out,16)
Tx110F_pad(out,16)
Tx1110_pad(out,16)
Tx0111_pad(in,1)
```

## Routing Section
```
# net id: e16
# m273: lb_p3_lyy_stencil_update_stream$lbmem_2_0$cgramem::rdata
# p269: add_704_709_710$binop::data1
# r15: lb_p3_lyy_stencil_update_stream$lb1d_2$reg_1::reg
Tx0C0C_rdata -> Tx0C0C_out_s3t1
Tx0B0C_in_s1t1 -> Tx0B0C_out_s3t1
Tx0A0C_in_s1t1 -> Tx0A0C_out_s3t1
Tx090C_in_s1t1 -> Tx090C_out_s3t1
Tx080C_in_s1t1 -> Tx080C_out_s3t1
Tx070C_in_s1t1 -> Tx070C_out_s3t1
Tx060C_in_s1t1 -> Tx060C_out_s3t1
Tx050C_in_s1t1 -> Tx050C_out_s3t1
Tx040C_in_s1t1 -> Tx040C_out_s3t1
Tx030C_in_s1t1 -> Tx030C_out_s3t1
Tx020C_in_s1t1 -> Tx020C_out_s2t1
Tx020B_in_s0t1 -> Tx020B_out_s2t1
Tx020A_in_s0t1 -> Tx020A_out_s2t1
Tx0209_in_s0t1 -> Tx0209_out_s2t1
Tx0208_in_s0t1 -> Tx0208_out_s2t1
Tx0207_in_s0t1 -> Tx0207_out_s2t1
Tx0206_in_s0t1 -> Tx0206_out_s2t1
Tx0205_in_s0t1 -> Tx0205_out_s3t1
Tx0105_in_s1t1 -> Tx0105_out_s2t1 (r)
Tx0205_in_s0t1 -> Tx0205_out_s2t1
Tx0204_in_s0t1 -> Tx0204_out_s2t1
Tx0203_in_s0t1 -> Tx0203_out_s2t1
Tx0202_in_s0t1 -> Tx0202_out_s1t1
Tx0202_out_s1t1 -> Tx0202_data1
```
Here, `in` and `out` indicate it is either the input direction or the output
direction. ``s3t0`` means ``side 3, track 0`` where sides `(0, 1, 2, 3)`
map to `(E, S, W, N)` i.e. `(right, bottom, left, top)`. Please notice that
`Tx0105_out_s2t1 (r)` has `(r)` suffixed. It is because there is a register
configured in the SB. Also pay an attention to
```
Tx0205_in_s0t1 -> Tx0205_out_s3t1
2Tx0105_in_s1t1 -> Tx0105_out_s2t1 (r)
2Tx0205_in_s0t1 -> Tx0205_out_s2t1
```
This is where the MST branch splits.

## Usage
You can find `bsbuilder.py` in `CGRAGenerator/bitstream/bsbuilder/`. To build
an annotated bitstream, simply do
```
$ python bsbuilder.py cgra_info.txt < input.bsb > output.bsa
```
where `cgra_info.txt` is design-specific info created by the
CGRA generator, and `input.bsb` is the `bsb` file built by `cgra_pnr.`
