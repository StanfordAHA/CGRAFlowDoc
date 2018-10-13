# PnR Analysis
`cgra_pnr` comes with a very simple analyzer on how the PnR performs. Currently
it computes:
1. Area usage, such as PE and MEM tiles.
2. Total wirelength.
3. Routing resource usage
4. Critical path delay (breakdown) and maximum clock frequency.

Here is an example for `Harris`:
```
Area Usage:
I    ████                                                                6.25%
P    ██████████████████████████████████████████████████████              80.21%
M    ██████████                                                          15.62%
--------------------------------------------------------------------------------
Total wire: 1202
--------------------------------------------------------------------------------
Critical Path:
Delay: 9.20 ns Max Clock Speed: 108.70 MHz
MUL  █████████████████████████████                                       43.48%
ALU  █████████████████████████                                           36.96%
SB   █████████████                                                       19.57%
CB                                                                       0.00%
MEM                                                                      0.00%
REG                                                                      0.00%
--------------------------------------------------------------------------------
BUS: 16
TRACK 0 █████████████████████████████████                                49.67%
TRACK 1 ███████████████████                                              28.91%
TRACK 2 █████████████                                                    19.40%
TRACK 3 ██████████████                                                   21.32%
TRACK 4 █████                                                            8.46%
BUS: 1
TRACK 0 ████████████████████                                             30.08%
TRACK 1 █████████████                                                    20.21%
TRACK 2 █████                                                            8.11%
TRACK 3 █                                                                1.89%
TRACK 4                                                                  0.00%
```
