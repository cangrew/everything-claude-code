---
name: gem5-patterns
description: gem5 simulator development patterns — SimObject authoring, SCons builds, Python configuration, memory system, CPU models, ISA work, debug/trace, and statistics. Use when writing, reviewing, or debugging gem5 C++ components or Python config scripts.
origin: ECC
---

# gem5 Development Patterns

Patterns and conventions for developing with the gem5 architecture simulator. Covers SimObject authoring, build system, configuration, memory hierarchy, CPU models, ISA work, and debugging.

## When to Activate

- Creating or modifying SimObjects (C++ components)
- Writing or debugging Python configuration scripts
- Working with the SCons build system
- Designing cache hierarchies or memory systems
- Adding or modifying ISA instructions
- Debugging simulation behavior with trace flags / stats

### When NOT to Use

- General C++ not related to gem5 (use `cpp-coding-standards`)
- General Python not related to gem5 config (use `python-patterns`)
- Non-gem5 simulators (Verilator, SystemC, Spike)

## SimObject Development

SimObjects are the fundamental building blocks. Every hardware component is a SimObject subclass.

### Lifecycle

Override these methods (called in this order after construction):

| Method | Purpose |
|--------|---------|
| Constructor | Parse params, initialize members, create ports |
| `init()` | Cross-SimObject setup after all objects are created and ports connected |
| `regStats()` | Register statistics — call parent's `regStats()` first |
| `initState()` | Cold-start initialization (skipped when restoring checkpoint) |
| `startup()` | Schedule initial events — clock is valid, all state is initialized |
| `serialize()` / `unserialize()` | Checkpoint support |

### Params System

Every SimObject needs a Python parameter class and the `PARAMS()` macro.

**Python params** (`YourObject.py`):

```python
from m5.params import *
from m5.SimObject import SimObject

class MyBuffer(SimObject):
    type = 'MyBuffer'
    cxx_header = "sim/my_buffer.hh"
    cxx_class = "gem5::MyBuffer"

    size = Param.Unsigned(64, "Buffer capacity in entries")
    latency = Param.Cycles(1, "Access latency")
    port = ResponsePort("Port for incoming requests")
```

**C++ header** (`my_buffer.hh`):

```cpp
#ifndef __SIM_MY_BUFFER_HH__
#define __SIM_MY_BUFFER_HH__

#include "params/MyBuffer.hh"
#include "sim/sim_object.hh"

namespace gem5 {

class MyBuffer : public SimObject
{
  public:
    PARAMS(MyBuffer);
    MyBuffer(const Params& p);

    void init() override;
    void startup() override;
    void regStats() override;

  private:
    const unsigned size;
    const Cycles latency;
};

} // namespace gem5

#endif // __SIM_MY_BUFFER_HH__
```

**C++ source** (`my_buffer.cc`):

```cpp
#include "sim/my_buffer.hh"
#include "debug/MyBuffer.hh"

namespace gem5 {

MyBuffer::MyBuffer(const Params& p)
    : SimObject(p),
      size(p.size),
      latency(p.latency)
{
}

void MyBuffer::init()
{
    SimObject::init();
}

void MyBuffer::startup()
{
    // Schedule initial events here — clock is now valid
}

void MyBuffer::regStats()
{
    SimObject::regStats();
    // Register stats here (see Statistics section)
}

} // namespace gem5
```

### DON'T

```cpp
// Scheduling events in the constructor — clock is not valid yet
MyBuffer::MyBuffer(const Params& p) : SimObject(p) {
    schedule(someEvent, curTick()); // BUG: clock not initialized
}
```

### Port System

Ports connect SimObjects through the memory system. `RequestPort` initiates transactions; `ResponsePort` receives them.

```cpp
class BufferResponsePort : public ResponsePort
{
  protected:
    bool recvTimingReq(PacketPtr pkt) override;   // must implement
    void recvRespRetry() override;
    Tick recvAtomic(PacketPtr pkt) override;       // must implement
    void recvFunctional(PacketPtr pkt) override;   // must implement
    AddrRangeList getAddrRanges() const override;
};
```

Key rules:
- Implement all pure virtuals on `ResponsePort` (`recvTimingReq`, `recvAtomic`, `recvFunctional`)
- Use `sendTimingResp()` from `ResponsePort`, `sendTimingReq()` from `RequestPort`
- Handle back-pressure: return `false` from `recvTimingReq` when busy, call `sendRetryReq()` when ready
- Override `getPort()` on the owning SimObject to return the port by name

### Event System

```cpp
// In header:
EventFunctionWrapper processEvent;

// In constructor:
MyBuffer::MyBuffer(const Params& p)
    : SimObject(p),
      processEvent([this]{ processQueue(); }, name() + ".processEvent")
{
}

// Scheduling:
schedule(processEvent, curTick() + latency);

// Rescheduling (must deschedule first if already scheduled):
if (processEvent.scheduled())
    deschedule(processEvent);
schedule(processEvent, curTick() + newTime);

// Or use reschedule (equivalent):
reschedule(processEvent, curTick() + newTime, true);
```

## Build System (SCons)

### SConscript Patterns

Register source files and SimObjects in the `SConscript` in your source directory:

```python
Import('*')

SimObject('MyBuffer.py', sim_objects=['MyBuffer'])
Source('my_buffer.cc')
DebugFlag('MyBuffer', "Traces for MyBuffer component")
```

For ISA-specific code:

```python
Source('my_isa_thing.cc', tags='x86 isa')
# Or for multiple ISAs:
Source('my_thing.cc', tags='riscv isa')
```

### Build Commands

```bash
# Build for a specific ISA and variant
scons build/X86/gem5.opt -j$(nproc)
scons build/RISCV/gem5.debug -j$(nproc)
scons build/ARM/gem5.fast -j$(nproc)

# Variants:
#   gem5.debug  — assertions + debug symbols, no optimization (for GDB)
#   gem5.opt    — optimized + debug symbols + assertions (default for development)
#   gem5.fast   — max optimization, no assertions/debug (for experiments)
#   gem5.prof   — profiling build (gprof)
```

### Common Build Errors

| Error | Fix |
|-------|-----|
| `ImportError: No module named 'MyBuffer'` | Add `SimObject('MyBuffer.py')` to SConscript |
| `undefined reference to gem5::MyBuffer` | Add `Source('my_buffer.cc')` to SConscript |
| `fatal error: params/MyBuffer.hh: No such file` | Rebuild after adding SimObject; check `cxx_header` path matches |
| `multiple definition of` | Guard headers; check you're not including `.cc` files |
| `error: 'THE_ISA' was not declared` | Add `#include "arch/generic/isa.hh"` or use ISA tags in SConscript |

## Python Configuration

### stdlib Config (v21+, recommended in v24+)

```python
from gem5.components.boards.simple_board import SimpleBoard
from gem5.components.cachehierarchies.classic.no_cache import NoCache
from gem5.components.memory import SingleChannelDDR3_1600
from gem5.components.processors.simple_processor import SimpleProcessor
from gem5.components.processors.cpu_types import CPUTypes
from gem5.isas import ISA
from gem5.resources.resource import obtain_resource
from gem5.simulate.simulator import Simulator

board = SimpleBoard(
    clk_freq="3GHz",
    processor=SimpleProcessor(cpu_type=CPUTypes.TIMING, isa=ISA.X86, num_cores=1),
    memory=SingleChannelDDR3_1600("1GiB"),
    cache_hierarchy=NoCache(),
)
board.set_se_binary_workload(obtain_resource("x86-hello64-static"))
Simulator(board=board).run()
```

### Classic Config Style

```python
import m5
from m5.objects import *

system = System()
system.clk_domain = SrcClockDomain(clock='1GHz', voltage_domain=VoltageDomain())
system.mem_mode = 'timing'
system.mem_ranges = [AddrRange('512MB')]
system.cpu = TimingSimpleCPU()
system.membus = SystemXBar()

system.cpu.icache_port = system.membus.cpu_side_ports
system.cpu.dcache_port = system.membus.cpu_side_ports
system.cpu.createInterruptController()
system.system_port = system.membus.cpu_side_ports

system.mem_ctrl = MemCtrl()
system.mem_ctrl.dram = DDR3_1600_8x8()
system.mem_ctrl.dram.range = system.mem_ranges[0]
system.mem_ctrl.port = system.membus.mem_side_ports

process = Process(cmd=['path/to/binary'])
system.cpu.workload = process
system.cpu.createThreads()
system.workload = SEWorkload.init_compatible(process.executable)

root = Root(full_system=False, system=system)
m5.instantiate()  # MUST call before simulate
exit_event = m5.simulate()
```

### DON'T

```python
root = Root(full_system=False, system=system)
m5.simulate()  # BUG: must call m5.instantiate() first
```

## Memory System and Cache Hierarchy

### Cache Configuration

```python
from m5.objects import Cache

class L1ICache(Cache):
    size = '32kB'; assoc = 8
    tag_latency = 1; data_latency = 1; response_latency = 1
    mshrs = 4; tgts_per_mshr = 8

class L1DCache(Cache):
    size = '64kB'; assoc = 8
    tag_latency = 2; data_latency = 2; response_latency = 1
    mshrs = 16; tgts_per_mshr = 16

class L2Cache(Cache):
    size = '256kB'; assoc = 16
    tag_latency = 10; data_latency = 10; response_latency = 1
    mshrs = 32; tgts_per_mshr = 16
```

### Wiring a Two-Level Hierarchy

```python
system.cpu.icache = L1ICache()
system.cpu.dcache = L1DCache()
system.l2cache = L2Cache()
system.l2bus = L2XBar()

system.cpu.icache.cpu_side = system.cpu.icache_port
system.cpu.dcache.cpu_side = system.cpu.dcache_port

system.cpu.icache.mem_side = system.l2bus.cpu_side_ports
system.cpu.dcache.mem_side = system.l2bus.cpu_side_ports

system.l2cache.cpu_side = system.l2bus.mem_side_ports
system.l2cache.mem_side = system.membus.cpu_side_ports
```

### Classic vs Ruby Coherence

| Aspect | Classic | Ruby |
|--------|---------|------|
| Speed | Faster simulation | Slower, cycle-accurate |
| Flexibility | Fixed protocol | Custom SLICC protocols |
| Use case | Quick experiments | Protocol research |
| Config | Python cache objects | SLICC + Ruby config |

## CPU Models

| Model | Timing | Use Case |
|-------|--------|----------|
| `AtomicSimpleCPU` | Functional, no timing | Fast-forward, functional warming |
| `TimingSimpleCPU` | In-order, models memory timing | Simple timing studies |
| `MinorCPU` | In-order pipeline | In-order core research |
| `O3CPU` | Out-of-order superscalar | Detailed microarchitecture studies |

### CPU Switching (Fast-Forward)

```python
system.cpu = [AtomicSimpleCPU() for _ in range(N)]
system.switch_cpus = [DerivO3CPU() for _ in range(N)]
for i in range(N):
    system.switch_cpus[i].switched_out = True
    system.switch_cpus[i].workload = system.cpu[i].workload

m5.switchCpus(system, list(zip(system.cpu, system.switch_cpus)))
```

## ISA Definitions

ISA descriptions live in `src/arch/<isa>/isa/`. Key files:

| File | Purpose |
|------|---------|
| `decoder.isa` | Top-level decoder entry point |
| `formats/` | Instruction format templates |
| `operands.isa` | Register operand definitions |
| `bitfields.isa` | Instruction encoding bit fields |

### ISA Guards in C++

Use SConscript ISA tags (`tags='riscv isa'`) to avoid compiling dead code. When runtime guards are needed:

```cpp
#include "arch/generic/isa.hh"

#if THE_ISA == RISCV_ISA
    // RISC-V specific code
#elif THE_ISA == X86_ISA
    // x86 specific code
#endif
```

## Debug, Trace, and Statistics

### Trace Flags

Define in SConscript:

```python
DebugFlag('MyBuffer', "Trace output for MyBuffer")
CompoundFlag('MySystem', ['MyBuffer', 'Cache', 'MemCtrl'])
```

Use in C++:

```cpp
#include "debug/MyBuffer.hh"

DPRINTF(MyBuffer, "Processing request addr=%#x size=%d\n",
        pkt->getAddr(), pkt->getSize());
```

Run with tracing:

```bash
build/X86/gem5.opt --debug-flags=MyBuffer configs/my_config.py
build/X86/gem5.opt --debug-flags=MyBuffer,Cache configs/my_config.py
build/X86/gem5.debug --debug-flags=All configs/my_config.py  # everything
```

### Statistics

Register in `regStats()`, always call parent's `regStats()` first:

```cpp
statistics::Scalar numHits;
statistics::Scalar numMisses;
statistics::Formula hitRate;
statistics::Vector accessesByType;

void MyBuffer::regStats()
{
    SimObject::regStats();

    numHits.name(name() + ".hits").desc("Number of hits");
    numMisses.name(name() + ".misses").desc("Number of misses");

    hitRate.name(name() + ".hit_rate").desc("Hit rate").precision(4);
    hitRate = numHits / (numHits + numMisses);

    accessesByType.init(3)
        .name(name() + ".accesses_by_type")
        .desc("Accesses by type")
        .flags(statistics::total);
    accessesByType.subname(0, "read");
    accessesByType.subname(1, "write");
    accessesByType.subname(2, "prefetch");
}
```

Available stat types: `statistics::Scalar`, `Vector`, `Histogram` (`.init(buckets)`), `Formula`, `Distribution`.

## gem5 Style Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| C++ classes | `CamelCase` | `MyBuffer`, `TimingSimpleCPU` |
| C++ methods | `camelCase` | `recvTimingReq`, `getPort` |
| C++ members | `camelCase` | `numEntries`, `hitCount` |
| Python classes | `CamelCase` | `SimpleBoard`, `L1DCache` |
| Python functions | `snake_case` | `set_se_binary_workload` |
| File names | `snake_case` | `my_buffer.cc`, `my_buffer.hh` |
| Namespaces | `gem5::` wraps everything | `namespace gem5 { ... }` |

### Include Order

1. Corresponding header, 2. C system headers, 3. C++ stdlib, 4. gem5 headers.

```cpp
#include "sim/my_buffer.hh"    // corresponding header first
#include <cstdint>             // C system
#include <string>              // C++ stdlib
#include "base/logging.hh"     // gem5 headers
#include "debug/MyBuffer.hh"
```

### Logging

Use `panic()` (abort), `fatal()` (abort + trace), `warn()`, `inform()` — never raw `printf`.

## Quick Reference Checklist

Before submitting gem5 changes:

- [ ] SimObject registered: `SimObject('Foo.py', sim_objects=['Foo'])` in SConscript
- [ ] Source file registered: `Source('foo.cc')` in SConscript
- [ ] Python params class has correct `type`, `cxx_header`, `cxx_class`
- [ ] `PARAMS(Foo)` macro present in C++ header
- [ ] Constructor signature is `Foo(const Params& p)`
- [ ] `regStats()` calls parent's `regStats()` first (e.g., `SimObject::regStats()`)
- [ ] All stats have `.name()` and `.desc()`
- [ ] All ports connected in Python config — no dangling ports
- [ ] Events scheduled in `startup()`, not in constructor
- [ ] Debug flag defined in SConscript, included via `debug/FlagName.hh`
- [ ] `namespace gem5 { }` wraps all C++ code
- [ ] Include guards use `__PATH_TO_FILE_HH__` format
- [ ] No scheduling before clock is initialized (use `startup()`)
- [ ] Build tested: `scons build/<ISA>/gem5.opt -j$(nproc)`

## Version Notes

- **v21+**: stdlib introduced (`gem5.components`, `gem5.simulate.Simulator`)
- **v22+**: `PARAMS()` macro replaced older `typedef` pattern; `namespace gem5` required
- **v23+**: `statistics::` namespace (was `Stats::` before); resource download API changes
- **v24+**: stdlib is the recommended config approach; classic configs still supported
