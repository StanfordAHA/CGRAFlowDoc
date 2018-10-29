**TODO: Overview of .io.json format**


## Example
An example file ([original
source](https://github.com/StanfordAHA/CGRAGenerator/blob/master/verilator/generator_z_tb/io/2in2out.json))
where the reset in signal comes into `pads_W_0[0]`, a 16 bit input comes into
`pads_N_0[0:16]`, a 16 bit output comes out of `pads_E_0[0:16]` and a 1 bit
output comes out of `pads_S_0[0]`.
```json
{
    "reset_in_pad": {
        "pad_bus" : "pads_W_0",
        "bits": {
            "0": { "pad_bit":"0" }
        },
        "mode": "reset",
        "width": 1
    },
    "io16in_in_arg_1_0_0": {
        "pad_bus" : "pads_N_0",
        "mode": "in",
        "width": 16
    },
    "io16_out_0_0": {
        "pad_bus" : "pads_E_0",
        "mode": "out",
        "width": 16
    },
    "io1_out_0_0": {
        "pad_bus" : "pads_S_0",
        "bits": {
            "0": { "pad_bit":"0" }
        },
        "mode": "out",
        "width": 1
    }
    }
```
