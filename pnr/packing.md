# Packing
The mapped netlist has many instances that can be packed into one single PE
tile: this process is called packing. It aims to reduce the number of nets and
instances that go through placement and routing, thus reducing the runtime.

Currently the packer performs the following packing rules:
### Constant folding
If a constant only drives a net with one input port, we will pack the
constant to the operand register. For instance, if we see something like
```
const0_0.out -> add123.data0
```
we will the `data0` register to be constant 0. Hence we remove one net from
the netlist.
### Register folding
If a register only drives a net with on input port, we will put that register
to the operand register and merge the nets. For instance, if we see something
like
```
mem1.rdata -> reg1.in -> mult1.data1 -> mult2.data1
reg1.out -> add123.data0
```
we will remove the second net and merge it with the first net. At the same
time, we will wire the `data0` register to the new net, e.g.,
```
mem1.rdata -> add123.data0 (r) -> mult1.data1 -> mult2.data1
```
where `(r)` denotes the operand register.

If for some reason we don't want to fold registers, such as for debugging
purpose, we can change a register with an adder where the other operand's
register will be a constant 0.
