# Unicorn ARM64 ida Trace

[简体中文](./README_zh.md)

This project provides a lightweight Unicorn engine-based ARM64 dynamic tracing emulation tool that can run as an IDA Pro plugin or standalone. The tool integrates tightly with IDA to ensure execution consistency and avoid errors. It provides reliable simulation reproduction capabilities and generates beautiful Tenet logs for debugging and analysis. Key features include dynamic code simulation, memory dumping, register state tracking, and instruction-level logging.

The tool doesn't require full memory dumps, instead dynamically dumping memory on demand during execution, making it highly efficient. Process verification ensures the entire acquisition process is complete and error-free. As a trace tool, it's particularly useful for VM analysis with good applicability and speed. Performance may be affected when dealing with numerous external function calls, which may be optimized in the future.

## Key Features

- **IDA Integration Plugin** (`dyn_trace_ida.py`)
  - Configure simulation parameters via GUI (end address, SO name, TPIDR value, etc.)
  - Automatically dump memory segments and register states
  - Support Tenet-compatible trace logs
  - Error handling (memory access exceptions, range checks, etc.)

- **Standalone Emulator** (`unicorn_trace.py`)
  - Load memory maps and register states from files
  - Custom simulation ranges
  - Generate detailed execution logs (`uc.log` and `tenet.log`)

## File Structure

```
.
├── dyn_trace_ida.py              # IDA plugin version
├── unicorn_trace.py              # Standalone emulator
├── unicorn_trace/                # Emulator core
│   └── unicorn_class.py          # ARM64 emulator base class
├── single_script/                # Utility scripts
│   ├── dynamic_dump.py           # IDA single-script version
│   └── dump_single.py            # Single dump script (not provided)
└── README.md                     # Project documentation
```

## Installation & Usage

### Dependencies
```bash
pip install unicorn capstone
```

### IDA Installation
Place `dyn_trace_ida.py` and the `unicorn_trace` folder in IDA's `plugins` directory

### Single Script Mode (Optional)
Use `single_script/dynamic_dump.py` directly in IDA debugging without library files

### Feature 1: Dynamic Dump / Automated Debugging
Run to specified location in IDA, write the target run address, and it will automatically run and record the trace to the end address

#### IDA Plugin Usage
1. Open configuration window with `Ctrl-Alt-U` in IDA
2. Set parameters:
   - End address (relative offset)
   - SO name (optional, required for Tenet)
   - TPIDR value (optional, required if errors occur)
   - Output path (optional, default local)
   - Enable Tenet logging (optional, not recommended as it affects efficiency; suggest offline updates)
3. Click confirm to start simulation

#### IDA Script Usage
If plugin is installed, use `dyn_trace_ida.py` directly. Otherwise use `dynamic_dump.py` from single_script.

Fill required parameters in main function (same as above)

### Feature 2: Standalone Emulator Usage
```python
from unicorn_trace import SelfRunArm64Emulator

# Initialize emulator
emulator = SelfRunArm64Emulator()
emulator.setup_from_files("libtarget.so", "./dumps")

# Run simulation
emulator.custom_main_trace(
    so_name="libtarget.so",
    end_addr=0x123456,
    tenet_log_path="./trace.log",
    user_log_path="./sim.log",
    load_dumps_path="./dumps"
)
```

## Example Workflow
Also see [Kanxue Article](https://bbs.kanxue.com/thread-289135.htm)

1. **Dynamic Execution, Memory Dump, Save State**:
Use plugin or script to run to end position

2. **Analyze Trace, Generate Tenet Log**:
Use `unicorn_trace.py` to generate tenet.log and combine all logs

3. **Log Analysis, Offline Simulation**:

## Notes

1. Special register values (like TPIDR) require manual configuration
2. Memory region dumping affects efficiency - larger `DUMP_SINGLE_SEG_SIZE` is slower, smaller may cause errors
3. Exception handling support:
   - Memory access errors (UC_ERR_READ_UNMAPPED)
   - Range violations (Code Run out of range)
   - AUTIASP instruction exceptions
   - B4 register handling
   - UNICORN runtime comparison with IDA
4. Each run is independent - if an error occurs mid-run, existing dump folders are unaffected. Just delete the latest error dump file and rerun after fixing the issue
5. Single round external call limit is 50 (shows `restart` when exceeded). Modify `ROUND_MAX` in code to change

## Error Handling

Common error codes:
- Missing `TPIDR`: Need PC value near error (or check log assembly), manually debug to `mrs xxx` location to get `TPIDR` register value. This occurs because IDA can't retrieve it, though it remains constant throughout execution
- IDA breakpoints causing errors: Disable all IDA breakpoints before running to avoid errors during comparison and call skipping
- IDA errors: IDA errors pause execution - just restart

## Contribution

Welcome to submit Issues or Pull Requests. Please ensure:
- Follow existing code style
- Add necessary unit tests
- Update relevant documentation

## TODO

- Improve efficiency: Continue execution after mid-process memory dumps instead of restarting
- Multi-architecture support

## Reference

Tenet IDA 9.0: https://github.com/jiqiu2022/Tenet-IDA9.0

Tenet: https://github.com/gaasedelen/tenet
