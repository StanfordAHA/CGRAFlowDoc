# Routing
Qoute from the [Wikipedia entry](https://en.wikipedia.org/wiki/Routing_(electronic_design_automation):
> In electronic design, wire routing, commonly called simply routing, is a step in the design of printed circuit boards (PCBs) and integrated circuits (ICs). It builds on a preceding step, called placement, which determines the location of each active element of an IC or component on a PCB. After placement, the routing step adds wires needed to properly connect the placed components while obeying all design rules for the IC. 

In the current CGRA, routing is done via configuring switch boxes (SB) and
connection boxes (CB). Please review the CGRA architecture information if you
are not family with the terminology. Generally speaking SB controls tile-to-
tile communication whereas CB controls tile to logic unit connections.

### Usage
```
$ python router.py -h
usage: CGRA Router [-h] -i PACKED_FILENAME -o ROUTE_FILE -c ARCH_FILENAME -p
                   PLACEMENT_FILENAME [--no-reg-fold] [--no-vis]

optional arguments:
  -h, --help            show this help message and exit
  -i PACKED_FILENAME, --input PACKED_FILENAME
                        Packed netlist file, e.g. harris.packed
  -o ROUTE_FILE, --output ROUTE_FILE
                        Routing result, e.g. harris.route
  -c ARCH_FILENAME, --cgra ARCH_FILENAME
                        CGRA architecture file
  -p PLACEMENT_FILENAME, --placement PLACEMENT_FILENAME
                        Placement file
  --no-reg-fold         If set, the placer will treat registers as PE tiles
  --no-vis              If set, the router won't show visualization result for
                        routing
```
