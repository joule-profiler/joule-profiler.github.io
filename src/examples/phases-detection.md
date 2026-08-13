# Phase Detection

This guide shows how to add phase markers to your programs for energy profiling.

## Basic Concept

In order to split the program's execution into phases, Joule Profiler uses a configurable regular expression pattern and checks for each line of the standard output if it contains the pattern.
The regular expression doesn't need to match the entire line but can match only a part of it. (e.g., a part of a line of log trace) 
The default token pattern is `__[A-Z0-9_]+__`.
The token pattern can be configured through the CLI:
```bash
joule-profiler profile --token-pattern <TOKEN> -- <COMMAND>
```

## Add Phase Markers

Here are several examples in different languages:

**Python:**
```python
print("__INIT__", flush=True)
load()

print("__PROCESSING__", flush=True)
process()
```

**C:**
```c
printf("__INIT__\n");
fflush(stdout);
load();

printf("__PROCESSING__\n");
fflush(stdout);
process();
```

**Rust:**
```rust
println!("__INIT__");
std::io::stdout().flush().unwrap();
load();

println!("__PROCESSING__");
std::io::stdout().flush().unwrap();
process();
```

> [!IMPORTANT]
> Always flush stdout immediately after printing tokens, buffered output may not be detected in time.

## Choose a Token Pattern

Create a regex pattern that matches your tokens:

```bash
# Simple pattern matching __WORD__
--token-pattern "__[A-Z_]+__"

# Custom pattern matching [WORD]
--token-pattern "\[A-Z_]+\]"

# Specific tokens only
--token-pattern "INIT|WORK|END"
```

Common patterns:
- `__[A-Z_]+__` - Matches `__INIT__`, `__WORK__`, `__END__`
- `<<<.*>>>` - Matches `<<<phase1>>>`, `<<<phase2>>>`
- `\[PHASE-[0-9]+\]` - Matches `[PHASE-1]`, `[PHASE-2]`

## Run the Profiler

```bash
joule-profiler profile --token-pattern "__[A-Z_]+__" -- <COMMAND>
```

Joule Profiler will make a measurement between each token to report energy and various metrics across phases.

## Complete Example

**script.py:**
```python
import time

print("__LOAD__", flush=True)
data = [i for i in range(1000000)]
time.sleep(0.5)

print("__COMPUTE__", flush=True)
result = sum(data)
time.sleep(0.5)

print("__DONE__", flush=True)
```

**Command:**
```bash
joule-profiler profile --token-pattern "__[A-Z_]+__" -- python3 script.py
```

**Output:**
```
__LOAD__
__COMPUTE__
__DONE__
╔════════════════════════════════════════════════╗
║  Command                                       ║
╚════════════════════════════════════════════════╝
  python3 script.py
 ────────────────────────────────────────────────
  Duration            :       1050 ms
  Exit code           :          0

╔════════════════════════════════════════════════╗
║  Phase: START -> __LOAD__                      ║
╚════════════════════════════════════════════════╝
  Duration            :         10 ms
  Start token         :      START
  End token           : __LOAD__ (line 0)

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :     140991 µJ
  PACKAGE-0           :     168334 µJ
  PSYS                :     304016 µJ
  UNCORE-0            :          0 µJ

╔════════════════════════════════════════════════╗
║  Phase: __LOAD__ -> __COMPUTE__                ║
╚════════════════════════════════════════════════╝
  Duration            :        520 ms
  Start token         : __LOAD__ (line 0)
  End token           : __COMPUTE__ (line 1)

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :     670959 µJ
  PACKAGE-0           :    1660888 µJ
  PSYS                :    3456726 µJ
  UNCORE-0            :      45166 µJ

╔════════════════════════════════════════════════╗
║  Phase: __COMPUTE__ -> __DONE__                ║
╚════════════════════════════════════════════════╝
  Duration            :        505 ms
  Start token         : __COMPUTE__ (line 1)
  End token           : __DONE__ (line 2)

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :     479736 µJ
  PACKAGE-0           :    1361267 µJ
  PSYS                :    2973632 µJ
  UNCORE-0            :      34912 µJ

╔════════════════════════════════════════════════╗
║  Phase: __DONE__ -> END                        ║
╚════════════════════════════════════════════════╝
  Duration            :         14 ms
  Start token         : __DONE__ (line 2)
  End token           :        END

┌────────────────────────────────────────────────┐
│ RAPL (perf_event)                              │
└────────────────────────────────────────────────┘
  CORE-0              :     207580 µJ
  PACKAGE-0           :     256774 µJ
  PSYS                :     436340 µJ
  UNCORE-0            :          0 µJ
```

If you encounter some issues with phases, see [troubleshooting](../troubleshooting/overview.md).