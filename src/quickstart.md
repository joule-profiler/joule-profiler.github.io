# Quick Start

Getting started with **Joule Profiler** is easy and straightforward.

First, install the latest version:
```bash
curl -fsSL https://raw.githubusercontent.com/joule-profiler/joule-profiler/main/install.sh | bash
```

Or using cargo:
```bash
cargo install joule-profiler-cli
```

To show the Joule Profiler helper:

```bash
joule-profiler -h # or --help
```

Each subcommand has its own helper, for example: 

```bash
joule-profiler profile -h # or --help
```

Some configuration arguments are to be passed before the subcommand, and some are specific to the command you use, you should refer to the helper for guidance.

## Basic Measurement

The simplest way to measure energy consumption is to run your program with the profiler:

```bash
joule-profiler profile -- <COMMAND>
```

> [!IMPORTANT]
> The command must be put after `--`.

This will execute your program and display energy consumption metrics in the terminal once it completes:

```
╔════════════════════════════════════════════════╗
║  Command                                       ║
╚════════════════════════════════════════════════╝
  python3 main.py
 ────────────────────────────────────────────────
  Duration            :       3013 ms
  Exit code           :          0

╔════════════════════════════════════════════════╗
║  Phase: START -> END                           ║
╚════════════════════════════════════════════════╝
  Duration            :       3013 ms
  Start token         :      START
  End token           :        END

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :    7299194 µJ
  PACKAGE-0           :   15104492 µJ
  PSYS                :   35101074 µJ
  UNCORE-0            :     355224 µJ
```

Some example programs are provided in the [examples](https://github.com/joule-profiler/joule-profiler/tree/main/examples/programs) directory in the project repository.

## Phases

In order to split the program's execution into phases, Joule Profiler uses a configurable regular expression pattern and checks for each line of the standard output if it contains the pattern.
The regular expression doesn't need to match the entire line but can match only a part of it. (e.g., a part of a line of log trace) 
The default token pattern is `__[A-Z0-9_]+__`.
The token pattern can be configured through the CLI:
```bash
joule-profiler profile --token-pattern <TOKEN> -- <COMMAND>
```

Put some prints separating your program's parts:

```py
...

print("__SETUP__", flush=True)

setup()

print("__WORKLOAD__", flush=True)

workload()

print("__CLEANUP__", flush=True)

cleanup()

...
```

Don't forget to flush the standard output after each print, see [troubleshooting](troubleshooting/overview.md).

Now, lauch Joule Profiler:

```bash
joule-profiler profile -- <COMMAND>
```

You will get something like:

```
__SETUP__
__WORKLOAD__
__CLEANUP__
╔════════════════════════════════════════════════╗
║  Command                                       ║
╚════════════════════════════════════════════════╝
  python3 main.py
 ────────────────────────────────────────────────
  Duration            :       3013 ms
  Exit code           :          0

╔════════════════════════════════════════════════╗
║  Phase: START -> __SETUP__                     ║
╚════════════════════════════════════════════════╝
  Duration            :         10 ms
  Start token         :      START
  End token           : __SETUP__ (line 0)

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :     184997 µJ
  PACKAGE-0           :     226806 µJ
  PSYS                :     387084 µJ
  UNCORE-0            :       4150 µJ

╔════════════════════════════════════════════════╗
║  Phase: __SETUP__ -> __WORKLOAD__              ║
╚════════════════════════════════════════════════╝
  Duration            :       1000 ms
  Start token         : __SETUP__ (line 0)
  End token           : __WORKLOAD__ (line 1)

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :    1856750 µJ
  PACKAGE-0           :    3949645 µJ
  PSYS                :    8571228 µJ
  UNCORE-0            :      85571 µJ

╔════════════════════════════════════════════════╗
║  Phase: __WORKLOAD__ -> __CLEANUP__            ║
╚════════════════════════════════════════════════╝
  Duration            :       1000 ms
  Start token         : __WORKLOAD__ (line 1)
  End token           : __CLEANUP__ (line 2)

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :    1398742 µJ
  PACKAGE-0           :    3311218 µJ
  PSYS                :    7377380 µJ
  UNCORE-0            :      58105 µJ

╔════════════════════════════════════════════════╗
║  Phase: __CLEANUP__ -> END                     ║
╚════════════════════════════════════════════════╝
  Duration            :       1003 ms
  Start token         : __CLEANUP__ (line 2)
  End token           :        END

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :    1468383 µJ
  PACKAGE-0           :    3395080 µJ
  PSYS                :    7314697 µJ
  UNCORE-0            :     137817 µJ
```

By default, only RAPL source is activated, use the `--sources` CLI flag to activate other sources:

```bash
joule-profiler --sources rapl,perf,nvml profiler -- <COMMAND>
```

For further information on the source configuration, see [CLI module](architecture/cli-module.md) and [sources documentation](sources/overview.md).

## Custom Output Format

By default, the output is shown in the terminal with a human-readable format. For further analysis or integration with other tools, you can export results in different formats:

```bash
# Export results as JSON for programmatic analysis
joule-profiler --output-format json profile -- <COMMAND>

# Export results as CSV for spreadsheet analysis
joule-profiler --output-format csv profile -- <COMMAND>
```

An output file will be generated at the end of the profiling.
These formats make it easy to process results with scripts or import them into visualization tools.

## Additional Options

### Specify an Output File

You can choose where to export your results:

```bash
# Save results to a file (only for json or csv formats)
joule-profiler -o results.json --output-format json profile -- <COMMAND>
```

### GPU Profiling

If your system has an Nvidia GPU and you want to measure GPU energy consumption alongside CPU:

```bash
# Include Nvidia GPU metrics in the measurement
joule-profiler --sources nvml profile -- <COMMAND>
```

Or for AMD GPU:
```bash
joule-profiler --sources amdsmi profile -- <COMMAND>
```

### Performance Counters

**Joule Profiler** supports [**perf_event**](sources/perf_event/introduction.md) performance counters on Linux systems, you can activate this feature by activating the `perf` source.

```bash
joule-profiler --sources perf profile -- <COMMAND>
```

GPU profiling requires the NVML library (part of NVIDIA driver installation).

### Choosing RAPL Backend

By default, **Joule Profiler** measures [**RAPL**](sources/rapl/introduction.md) counters using either [**perf_event**](sources/rapl/perf-event.md) (default) or [**powercap**](sources/rapl/powercap.md) backend depending on your system.

You can explicitly choose which backend to use:

```bash
# Use perf_event backend (default one, lower overhead, may require kernel configuration, see perf_event_paranoid documentation)
joule-profiler -D profiler.rapl_backend=perf profile -- <COMMAND>

# Use Powercap backend (requires root)
sudo joule-profiler -D profiler.rapl_backend=powercap profile -- <COMMAND>
```

The choice of backend can affect measurement granularity and permission requirements. See the [**RAPL**](sources/rapl/introduction.md) documentation for details on each backend.

---

For more advanced usage including phase-based profiling, iterations, and custom metric sources, see the [examples folder](examples/overview.md).