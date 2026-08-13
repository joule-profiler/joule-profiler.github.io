# Build from Source

You can build the profiler from the sources by cloning the repository. It can be useful if you want to access the latest features not yet released, compile Joule Profiler with only a subset of features, or if you intend to customize the source code.

```bash
# Clone the repository
git clone https://github.com/joule-profiler/joule-profiler.git
cd joule-profiler

# Build and install system-wide
cargo build --release
cp target/release/joule-profiler /usr/local/bin/

# Verify installation
joule-profiler --version
```

> [!TIP]
> System-wide installation (`/usr/local/bin/`) is recommended as the tool may require root privileges to use some features (e.g., [**cgroup**](../sources/cgroup/introduction.md), [**RAPL Powercap**](../sources/rapl/powercap.md)).

> [!NOTE]
> By default, Joule Profiler is compiled with all the sources and features, if you want to filter which sources to compile the profiler with, use `--no-default-features` and `--features` cargo flags: 
> ```bash
> # Compile only the RAPL and perf_event sources
> cargo build --release --no-default-features --features rapl,perf
> ```

A source is usually featured with its name but if you're struggling to activate one, check the different features in the [Cargo.toml](https://github.com/joule-profiler/joule-profiler/blob/main/cli/Cargo.toml) file.