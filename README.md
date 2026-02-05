# single_stage_pipeline_register
# Single-Stage Pipeline Register with Valid/Ready Handshake

A fully synthesizable SystemVerilog implementation of a single-stage pipeline register using standard valid/ready handshake protocol.

## Features

- Standard valid/ready handshake protocol
- Correct backpressure handling (no data loss or duplication)
- Fully synthesizable design
- Asynchronous active-low reset to clean empty state
- Parameterized data width
- Comprehensive testbench included
- Formal verification properties (optional)

## Interface

### Parameters
- `DATA_WIDTH`: Width of data path (default: 32 bits)

### Ports

| Port | Direction | Width | Description |
|------|-----------|-------|-------------|
| `clk` | input | 1 | Clock signal |
| `rst_n` | input | 1 | Active-low asynchronous reset |
| `in_valid` | input | 1 | Input data valid |
| `in_ready` | output | 1 | Ready to accept input data |
| `in_data` | input | DATA_WIDTH | Input data |
| `out_valid` | output | 1 | Output data valid |
| `out_ready` | input | 1 | Downstream ready to accept data |
| `out_data` | output | DATA_WIDTH | Output data |

## Protocol

### Valid/Ready Handshake
- A transaction occurs when both `valid` and `ready` are asserted on the same clock edge
- `valid` may be asserted before or after `ready`
- Once asserted, `valid` must remain high until the transaction completes
- Once asserted, `data` must remain stable until the transaction completes
- `ready` may be deasserted at any time

### Backpressure Handling
- When `out_ready` is low, the pipeline register holds its data
- `in_ready` is deasserted when the register is full and output is not being consumed
- When both input and output transactions occur simultaneously, the register captures new input data

## Implementation Details

### State Machine
The module uses a simple valid bit to track whether the register contains valid data:
- **Empty state** (`valid_reg = 0`): Ready to accept new data
- **Full state** (`valid_reg = 1`): Contains valid data, ready only if output is being consumed

### Ready Signal Logic
```systemverilog
in_ready = !valid_reg || output_fire
```
The input is ready when:
1. The register is empty, OR
2. The output is being drained in this cycle

### Data Path
Data is captured on any input transaction (`in_valid && in_ready`), ensuring no data loss when simultaneous input/output transactions occur.

## Simulation

### Requirements
- Icarus Verilog, Verilator, or any SystemVerilog-compatible simulator
- GTKWave (optional, for waveform viewing)

### Running with Icarus Verilog
```bash
# Compile
iverilog -g2012 -o sim pipeline_register.sv tb_pipeline_register.sv

# Run simulation
./sim

# View waveforms (if VCD dumping is enabled)
gtkwave dump.vcd
```

### Running with Verilator
```bash
verilator --binary -j 0 -Wall --timing \
    --top-module tb_pipeline_register \
    tb_pipeline_register.sv pipeline_register.sv

./obj_dir/Vtb_pipeline_register
```

## Test Coverage

The included testbench verifies:
1. **Basic Transfer**: Simple data pass-through
2. **Backpressure**: Data retention when downstream is not ready
3. **Continuous Flow**: Back-to-back transactions
4. **Random Stimulus**: Randomized valid/ready patterns with queue verification
5. **Reset Behavior**: Proper initialization and reset handling

## Synthesis

The design is fully synthesizable and has been written with the following considerations:
- No combinational loops
- All outputs registered (except combinational `in_ready`)
- Proper reset handling
- No latches or inferred memory

### Synthesis Example (Yosys)
```bash
yosys -p "read_verilog -sv pipeline_register.sv; synth -top pipeline_register; write_verilog synth_output.v"
```

## Formal Verification

Optional SystemVerilog assertions are included (guarded by `FORMAL` define) for formal verification:
- Data stability under backpressure
- Valid signal persistence
- No spurious valid assertions

## Timing

- Maximum frequency depends on target technology
- Critical path: Input data → Register → Output data (single flip-flop delay)
- Ready path is combinational: `valid_reg` and `out_ready` → `in_ready`

## Usage Example

```systemverilog
pipeline_register #(
    .DATA_WIDTH(64)
) my_pipeline_stage (
    .clk(clk),
    .rst_n(rst_n),
    .in_valid(stage1_valid),
    .in_ready(stage1_ready),
    .in_data(stage1_data),
    .out_valid(stage2_valid),
    .out_ready(stage2_ready),
    .out_data(stage2_data)
);
```


