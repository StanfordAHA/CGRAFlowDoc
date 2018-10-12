# Detailed Routing
Because the current `cgra_pnr` uses a simple single-stage router, it only has a
detailed router without global routing information.

## Routing Rules and Strategies
There many rules in routing due to the architecture constraint. Please review
CGRA architecture if you are not familiar with the routing resource on the chip.
Some of the rules are imposed due to hardware limitation whereas some are
implementation details aiming to reduce the total wirelength.
1. If a net has register in it, we will route the net which is drived by that
particular register. Hence we will have the case like `net -> net -> net ...`
I call these grouped net mega-net. It is because registers have to be
configured on the switch box. As a result, the placer does not make the
decision on which switch box to put on. Hence it is the router's job to place
registers. Currently the router will route from the tile where the register
is place to the next port. Whichever SB the wire goes through will be treated
as a register placement.
2. It tries to construct a minimum spanning tree (MST) while routing the net.
The MST may not be perfect since the router does not have the global routing
information. As a result, it will create routing congestion and increase
the total wirelength.
3. It routes every 5 channels at once for each net, and pick whichever has the
fewest number of wires as the wiring for the net. Hence it makes a local greedy
decision for every net.
4. If there is a *self-connection*, that is, a net goes through both input
ports for a single PE tiles, the router will try to avoid using SB. However,
sometimes SB has to be used because the way CB is designed.
5. The majority of the nets use 16-bit tracks and only a few use 1-bit track.
These two types nets are routed together.

There are more nitty-gritty details on the routing rules and strategies used
in the current implementation. Please refer to the comments in the [source
code](https://github.com/Kuree/cgra_pnr/blob/master/router.py)
