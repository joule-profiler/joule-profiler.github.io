# Metrics

**Joule Profiler** uses NVML to retrieve several metrics of GPU devices, such as energy consumption, GPU utilization or VRAM usage.

While NVML is capable of reporting various other telemetry data (such as fan speeds, temperature, and clock rates), **Joule Profiler** focuses specifically on the energy and usage metrics.

## Precision and Overflow

The energy counters provided by NVML are 64-bit unsigned integers representing energy in millijoules (mJ).

### Overflow Analysis

This unit allows energy to be measured over extremely long periods without realistic concern for integer overflow. For example, considering a GPU consuming approximately 300 W continuously:

$$P = 300~\text{W} = 300~\text{J/s} = 3.0 \times 10^5~\text{mJ/s}$$

$$
t_\text{overflow}
= \frac{E_\text{max}}{P}
= \frac{2^{64}~\text{mJ}}{3.0 \times 10^5~\text{mJ/s}}
\simeq 6.1 \times 10^{13}~\text{s}
\simeq 1.9 \times 10^6~\text{years}
$$


Consequently, overflow of the NVML energy counter is generally ignored in our design as the time required to trigger it exceeds any practical profiling duration. However, in the unlikely event that an overflow does occur, we utilize wrapping subtraction to ensure measurements remain consistent without panicking.