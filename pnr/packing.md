# Packing
The mapped netlist has many instances that can be packed into one single PE
tile: this process is called packing. It aims to reduce the number of nets and
instances that go through placement and routing, thus decreasing the runtime.

## Packing Rules:
Although our CGRA is different from a typical FPGA, we can still pack the
netlist to further reduce the size of netlists and thus improve the PnR result.
Currently packing is done in `packer` (part of `cgra_pnr`), as opposed to the
mapper.

Packing is done through multiple passes that modify the underlying netlist.
Here are the two passes it currently uses:

### Constant folding
If a constant only drives **one** net with **one** input port, we will fold the
constant to the operand register. For instance, if we see something like
```
const0_0.out <-> add123.data0
```
we will set the `data0` register to be constant 0. Hence we remove one net from
the netlist.

### Register folding
If a register only drives **one** net with **one** input port, we will put that
register to the operand register and merge the nets. For instance, if we see
something like
```
mem1.rdata <-> reg1.in <-> mult1.data1 <-> mult2.data1
reg1.out <-> add123.data0
```
we will remove the second net and merge it with the first net. At the same
time, we will wire the `data0` register to the new net, e.g.,
```
mem1.rdata <-> add123.data0 (r) <-> mult1.data1 <-> mult2.data1
```
where `(r)` denotes the operand register.

If for some reason we don't want to fold registers, such as for debugging
purpose, we can change a register to an adder where the other operand's
register will be a constant 0. You can use `--no-reg-folg` in the PnR toolchain
to specify this need.

### Extra wire removing
Because all the current applications do not use certain signals, we can safely
remove some nets. Those signals will be wired to a constant `1` by `bsbuilder`.

Below is a list of wires removed by this particular pass:
+ `cg_en`
+ `ren`

### Instance renaming
Although each instance has their unique name in the mapped netlist, their names
are either too long or doesn't reflect their instance type. A pass can be used
to rename all instances to internal IDs.

## Usage:
```
$ python packer.py -h
usage: CGRA Packing tool [-h] -n INPUT -o OUTPUT [--no-reg-fold]

optional arguments:
  -h, --help            show this help message and exit
  -n INPUT, --netlist INPUT
                        Mapped netlist file, e.g. harris.json
  -o OUTPUT, --output OUTPUT
                        Packed netlist file, e.g. harris.packed
  --no-reg-fold         If set, the packer will turn registers into PE tiles
```
## Packing file format
There are several sections in the file:
- Netlists: the netlist after packing stage.
- Folded Blocks: instances removed through packing stage. It also specifies
  which port the const/reg folded to.
- ID to Names: conversion from the internal block IDs to the instance name
- Changed to PE: normally empty unless `--no-reg-fold` is used.
- Netlist Bus: tell you which track the net uses, either 16-bit or 1-bit.

```
Netlists:
e1: (p2, out)   (i0, in)
e2: (i1, out)   (p2, data0)

Folded Blocks:
(c3, out) -> (p2, const2__348, data1)

ID to Names:
i0: io16_out
i1: io16in_in_0
p2: mul_347_348_349_PE
c3: const2__348

Changed to PE:

Netlist Bus:
e1: 16
e2: 16
```
