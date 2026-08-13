# Sensors Listing Example

This example shows how to list all available sensors using Joule Profiler.

## Minimal Example

```bash
joule-profiler list-sensors
```

And it shows:
```
╔════════════════════════════════════════════════╗
║  Available Sensors                             ║
╚════════════════════════════════════════════════╝

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  Name                 | Unit 
 ────────────────────────────────────────────────
  PACKAGE-0            | µJ
  CORE-0               | µJ
  UNCORE-0             | µJ
  PSYS                 | µJ
```

## With GPU support

You can also list the GPU devices available:

For NVIDIA GPU:
```bash
joule-profiler --sources nvml list-sensors
```

For AMD GPU:
```bash
joule-profiler --sources amdsmi list-sensors
```


```
╔════════════════════════════════════════════════╗
║  Available Sensors                             ║
╚════════════════════════════════════════════════╝

┌────────────────────────────────────────────────┐
│ NVML                                           │
└────────────────────────────────────────────────┘
  Name                 | Unit 
 ────────────────────────────────────────────────
  GPU-0                | mJ

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  Name                 | Unit 
 ────────────────────────────────────────────────
  PACKAGE-0            | µJ
  CORE-0               | µJ
  UNCORE-0             | µJ
  PSYS                 | µJ
```

## With perf_event support

List perf_event counters:

```bash
joule-profiler --sources perf list-sensors
```

```
╔════════════════════════════════════════════════╗
║  Available Sensors                             ║
╚════════════════════════════════════════════════╝

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  Name                 | Unit 
 ────────────────────────────────────────────────
  PACKAGE-0            | µJ
  CORE-0               | µJ
  UNCORE-0             | µJ
  PSYS                 | µJ

┌────────────────────────────────────────────────┐
│ perf_event                                     │
└────────────────────────────────────────────────┘
  Name                 | Unit 
 ────────────────────────────────────────────────
  CPU_CYCLES           | count
  INSTRUCTIONS         | count
  CACHE_MISSES         | count
  BRANCH_MISSES        | count
```