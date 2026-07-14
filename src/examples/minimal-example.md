# Minimal Example

This guide shows the minimal example to use **Joule Profiler**.

## Minimal Measurement

Measure any program's energy consumption:

```bash
# Measure a simple command
joule-profiler profile -- sleep 1

# Measure a Python script, example at: examples/programs/workload.py
joule-profiler profile -- python3 examples/programs/workload.py

# Measure a program with arguments
joule-profiler profile -- ./my-program arg1 arg2
```

Use `sudo` if root privileges are required.
Some example programs are provided in the [examples](https://github.com/joule-profiler/joule-profiler/tree/main/examples/programs) directory in the project repository.

## Basic Phase Detection

Add phase markers to your program:

**Python example:**
```python
# my_script.py
import time

print("__START__", flush=True)
time.sleep(1)
print("__END__", flush=True)
```

The profiler will measure energy separately for the `__START__` to `__END__` phase.