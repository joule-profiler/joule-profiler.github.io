# Quick Install

Install the latest version with a single command:

```bash
curl -fsSL https://raw.githubusercontent.com/joule-profiler/joule-profiler/main/install.sh | bash
```

This script automatically detects your operating system and architecture, downloads the correct pre-built binary, and installs it to `/usr/local/bin/`.

Or with cargo:
```bash
cargo install joule-profiler-cli

# Move it to /usr/local/bin to install it globally
sudo mv ~/.cargo/bin/joule-profiler /usr/local/bin/
```

> [!NOTE]
> By default, Joule Profiler is installed with all the sources and features, if you want to filter which sources to compile the profiler with, use `--no-default-features` and `--features` cargo flags: 
> ```bash
> # Install only the RAPL and perf_event sources
> cargo install joule-profiler-cli --no-default-features --features rapl,perf
> ```

A source is usually featured with its name but if you're struggling to activate one, check the different features in the [Cargo.toml](https://github.com/joule-profiler/joule-profiler/blob/main/cli/Cargo.toml) file.

# Custom Installation

For more control over the installation process, you can pass arguments to the installation script.

**Install to a custom directory**, useful for non-root users or specific environment configurations (e.g., `~/.local/bin`):

```bash
curl -fsSL https://raw.githubusercontent.com/joule-profiler/joule-profiler/main/install.sh | bash -s -- --dir ~/.local/bin
```

**Install a specific version:**

If you need to install a specific version depending on the features you need:

```bash
curl -fsSL https://raw.githubusercontent.com/joule-profiler/joule-profiler/main/install.sh | bash -s -- --version v3.0.0
```

**List available versions:**

You can see the list of available versions with the installer:

```bash
curl -fsSL https://raw.githubusercontent.com/joule-profiler/joule-profiler/main/install.sh | bash -s -- --list
```

**Non-interactive Mode**, for use in automated scripts or CI pipelines:

```bash
curl -fsSL https://raw.githubusercontent.com/joule-profiler/joule-profiler/main/install.sh | bash -s -- --yes
```