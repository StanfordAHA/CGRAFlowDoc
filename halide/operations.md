# Supported Operations

|Halide representation                   |CoreIR instances                            |
|----------------------------------------|--------------------------------------------|
|`InputParam`                            |`def.input`                                 |
|`Param`                                 |`const` (set during configuration)          |
|`_const_`                               |`const`                                     |
|`*` `/` `+` `-`                         |`mul`, `{a,l}shr`, `add`, `sub`             |
|`/` `%`                                 |`{u,s}div`, `{u,s}rem`        `             |
|`!=` `==`                               |`neq`, `eq`                                 |
|`<`  `<=`  `>`  `>=`                    |`{u,s}lt`, `{u,s}le`, `{u,s}gt`, `{u,s}ge`  |
|`&&` <code>&#124;&#124;</code> `!`      |`and`, `or`, `not`                          |
|`&`  <code>&#124;</code>  `~` `^`       |`and`, `or`, `not`, `xor`                   |
|`>>` `<<`                               |`{a,l}shr`, `shl`                           |
|`select`                                |`mux`                                       |
|`max`  `min`                            |`{u,s}max`, `{u,s}min`                      |
|`absd`, `*` `+`                         |`absd`, `mad`                               |
|`(x*y)>>8`, `(x*y)>>16`                 |`mult_middle`, `mult_high`                  |
|`for`                                   |`counter`                                   |
|                                        |                                            |
|`hw_accelerate`                         |Create circuit between input                |
|`store_at`                              |Create memory                               |
|`RDom`                                  |Defines input stencil size for linebuffer   |
|`update`                                |Gets reduction update handle for scheduling |
|`unroll`                                |Duplicates hardware                         |
|`tile`                                  |Defines linebuffer width                    |
|`reorder `                              |Defines how data is read from image         |
