# CLI Module

The CLI Module is the straightforward entry point for users.

It parses command-line input and configure the profiler to be usable quickly and easily while providing several configuration options.
The CLI acts as an adapter between the user, the core domain, and the sources.
Because of this separation, the CLI can evolve independently from the core logic and sources.

Here's a summary of all the CLI arguments:

| Argument | Default | Description |
|---|---|---|
| `-v` or `--verbose` | ERROR level | Verbosity level. Stack for more detail: `-v`, `-vv`, `-vvv`. |
| `--output-format` | `terminal` | Specify the output format in which to export the data. (Available: terminal, csv, json) |
| `-o` or `--output-file` | `data<TIMESTAMP>.csv/json` | Custom output file path for CSV / JSON export. |
| `--sources` | `rapl` | A comma-separated list of the activated sources. (e.g., `rapl,perf,nvml`) |
| `-D` or `--define` || Override a configuration key, repeatable. (e.g., `-D profiler.rapl_backend=powercap`) |
| `--config` || TOML configuration file. Every key it sets can also be set with `-D`. |
| `command` | None (required) | The program to execute and profile. |

The subcommand `profile` has also some arguments:

| Argument | Default | Description |
|---|---|---|
| `--token-pattern` | `__[A-Z0-9_]+__` | Regex to detect phase tokens in the program's stdout. |
| `--stdout-file` | None | Redirect the profiled program's stdout to a file. |
| `--use-root` | false | Executes the profiled command with root privileges if true and Joule Profiler is launched as root. |
| `--init-timeout` | 1s | Duration before aborting sources initialization. (e.g., 1s, 10s, 1m) |

The profiled command is provided after `--` at the end of the arguments.