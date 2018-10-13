# Routing
Qoute from the [Wikipedia entry](https://en.wikipedia.org/wiki/Routing_(electronic_design_automation):
> In electronic design, wire routing, commonly called simply routing, is a step in the design of printed circuit boards (PCBs) and integrated circuits (ICs). It builds on a preceding step, called placement, which determines the location of each active element of an IC or component on a PCB. After placement, the routing step adds wires needed to properly connect the placed components while obeying all design rules for the IC. 

In the current CGRA, routing is done via configuring switch boxes (SB) and
connection boxes (CB). Please review the CGRA architecture information if you
are not family with the terminology. Generally speaking SB controls tile-to-
tile communication whereas CB controls tile to logic unit connections.

## Usage
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
## Routing file format

For most of the connections it `(tile_from) -> (tile_to) (track_out) ->
(track_in)`. Put them together it becomes `tile_from:track_out ->
tile_to:track_in`. For the source connection, because the parser can infer
the next time based on the `track_out`, `(tile_to)` is left out on purpose.
As a result, it becomes `(tile_form):port -> (track_out) -> (track_in)`.
The source connection will also have `SOURCE` as an indication. For sink
port connection, because we can also infer the coming tile, we can simple
have `(tile_in):port <- (track_in)`.
```
Node 0: SOURCE (9, 9)::out -> (16, 1, 0, 0) -> (16, 0, 2, 0)
Node 1: (10, 9) -> (11, 9)	(16, 1, 0, 0) -> (16, 0, 2, 0)
Node 2: (11, 9) -> (12, 9)	(16, 1, 0, 0) -> (16, 0, 2, 0)
Node 3: (12, 9) -> (13, 9)	(16, 1, 0, 0) -> (16, 0, 2, 0)
Node 4: (13, 9) -> (14, 9)	(16, 1, 0, 0) -> (16, 0, 2, 0)
Node 5: (14, 9) -> (15, 9)	(16, 1, 0, 0) -> (16, 0, 2, 0)
Node 6: (15, 9) -> (16, 9)	(16, 1, 0, 0) -> (16, 0, 2, 0)
Node 7: (16, 9) -> (16, 8)	(16, 1, 3, 0) -> (16, 0, 1, 0)
Node 8: (16, 8) -> (16, 7)	(16, 1, 3, 0) -> (16, 0, 1, 0)
Node 9: (16, 7) -> (16, 6)	(16, 1, 3, 0) -> (16, 0, 1, 0)
Node 10: (16, 6) -> (16, 5)	(16, 1, 3, 0) -> (16, 0, 1, 0)
Node 11: (16, 5) -> (16, 4)	(16, 1, 3, 0) -> (16, 0, 1, 0)
Node 12: (16, 4) -> (16, 3)	(16, 1, 3, 0) -> (16, 0, 1, 0)
Node 13: (16, 3) -> (16, 2)	(16, 1, 3, 0) -> (16, 0, 1, 0)
Node 14: (16, 2) -> (16, 1)	(16, 1, 3, 0) -> (16, 0, 1, 0)
Node 15: (16, 1) -> (17, 1)	(16, 1, 0, 0) -> (16, 0, 2, 0)
Node 16: SINK (17, 1)::in <- (16, 0, 2, 0)
```

For registers that are put on the switch box, the port will be `reg`.
The bitstream generator needs to recognize this.
