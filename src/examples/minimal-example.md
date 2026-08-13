# Minimal Example

This guide shows the minimal example to use **Joule Profiler**.

## Minimal Measurement

Measure any program's energy consumption:

```bash
# Measure a simple command
joule-profiler profile -- sleep 1

# Measure a Python script, example at: examples/programs/workload.py
joule-profiler profile -- python3 examples/programs/workload.py

# Measure a program with arguments, example at: examples/programs/nbody.c
joule-profiler profile -- joule-profiler profile -- ./nbody 10000000
```

Use `sudo` if root privileges are required.
Some example programs are provided in the [examples](https://github.com/joule-profiler/joule-profiler/tree/main/examples/programs) directory in the project repository.

## Basic Phase Detection

Add phase markers to your program:

**Python example:**
```python
# script.py
import time

print("__START__", flush=True)
time.sleep(1)
print("__END__", flush=True)
```

```bash
joule-profiler profile -- python3 script.py
```


The profiler will measure energy separately for the `__START__` to `__END__` phase.