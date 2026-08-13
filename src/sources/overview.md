## Sources Overview

**Joule Profiler** is designed to be simple, portable, and focused on monitoring the components responsible for the majority of energy consumption, such as the CPU, GPU, and SoC. While it does not aim to support every possible energy source on every device, the profiler can be extended by implementing custom sources for specific devices or components.  

Also, it implements several source unrelated to energy consumption such as [**perf_event**](perf_event/introduction.md), [**cgroup**](cgroup/introduction.md) or [**procfs**](procfs/introduction.md).

### Supported Architectures

- **CPU:** The only target is Intel and AMD x86 architecture at the moment.
- **GPU:** We support NVIDIA and AMD GPUs.
- **OS:** Only Linux-based systems are officially supported at the moment.

### Available Sources

- **Intel RAPL:** Measures CPU energy consumption.
  - Available on most Intel CPUs since Sandy Bridge architecture in 2011, and most AMD CPUs since Zen microarchitecture in 2017.  
  - Implemented using either **perf_event** or **Powercap** on Linux systems.  
  - For details, see [**RAPL**](rapl/introduction.md).

- **perf_event:** Measures various performance counters like hardware or software on Linux systems, see [**perf_event**](perf_event/introduction.md).

- **Nvidia GPUs (NVML):** Provides energy and performance metrics for Nvidia GPUs through the *Nvidia Management Library*, see [**NVML**](nvml/introduction.md).

- **AMD GPUs (AMDSMI):** Provides energy and performance metrics for AMD GPUs using the AMDSMI library, see [**amdsmi**](amdsmi/introduction.md).

- **Cgroup:** Measures several CPU, memory and I/O metrics available with the CgroupV2 Linux API, see [**cgroup**](cgroup/introduction.md).

- **Procfs:** Provides memory and I/O metrics using the Linux /proc file system, see [**procfs**](procfs/introduction.md).

### Extending Joule Profiler

Users can implement new metric sources allowing the monitoring of additional devices or components beyond the default set. For guidance, see [adding a new source](../developer-guide/adding-source.md).  
