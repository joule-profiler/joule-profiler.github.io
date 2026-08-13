# Adding a Custom Source

Joule Profiler is built on a modular architecture designed for extensibility. This allows users to integrate new metric sources such as custom hardware sensors, software counters, or external APIs without modifying the core profiling engine.

## Trait Implementation

To create a valid source for **Joule Profiler**, your struct must implement the `MetricReader` trait.  
Some provided methods are optional.

See the [source implementation example](../examples/source-implementation.md) to understand with a minimal example how to implement a source.

```rs
pub trait MetricReader: Send + 'static {
    /// Type of metrics returned by the reader.
    type Type: Send;

    /// Error type produced by the reader.
    type Error: std::error::Error + Send + Sync;

    /// Config type for configuring the source.
    type Config: Default + DeserializeOwned;

    //
    // Mandatory methods
    //

    /// Build the source from its deserialized configuration.
    fn from_config(config: Self::Config) -> Result<Self, Self::Error>
    where
        Self: Sized;

    /// Perform a measurement and update internal state.
    fn measure(&mut self) -> impl Future<Output = Result<(), Self::Error>> + Send;

    /// Retrieve the current metrics snapshot.
    fn retrieve(&mut self) -> impl Future<Output = Result<Self::Type, Self::Error>> + Send;

    /// Return all sensors provided by this source.
    fn get_sensors(&self) -> Result<Sensors, Self::Error>;

    /// Convert a snapshot into Joule Profiler metrics.
    fn to_metrics(&self, result: Self::Type) -> Result<Metrics, Self::Error>;

    /// Return the human readable name of the metric source.
    fn get_name() -> &'static str;

    /// Return the identifier of the metric source.
    fn get_id() -> &'static str;

    //
    // Optional methods
    //

    /// Pre-initialize the source, called before the profiled process is spawned.
    fn pre_init(&mut self) -> impl Future<Output = Result<(), Self::Error>> + Send {
        async { Ok(()) }
    }

    /// Initialize the source once the profiled process is spawned, with its pid.
    fn init(&mut self, _pid: i32) -> impl Future<Output = Result<(), Self::Error>> + Send {
        async { Ok(()) }
    }

    /// Cleanup or join logic, called when profiling is over.
    /// There is no `Drop` implementation because the source is reusable.
    fn join(&mut self) -> impl Future<Output = Result<(), Self::Error>> + Send {
        async { Ok(()) }
    }
}
```

### Associated Types

- `Type` is the snapshot type produced by `retrieve` and consumed by `to_metrics`. It only has to be `Send`.
- `Error` must be `std::error::Error + Send + Sync`. Sources usually define their own `thiserror` enum,
  but `MetricSourceError` can be used directly for simple sources.
- `Config` must be `Default + DeserializeOwned`. It is deserialized from the source's section of the
  configuration file, keyed by `get_id()`. Use `type Config = ()` if your source has nothing to configure.

### Name and Identifier

`get_name()` is the human readable name displayed in the results, while `get_id()` is the identifier used
to enable the source and to look up its configuration section (`[sources.<id>]`).

### Lifecycle

The three optional methods are called at distinct moments of a profiling session, so pick the one matching
what your source needs:

| Method | When | Typical use |
| --- | --- | --- |
| `pre_init` | Before the profiled process is spawned | Create a cgroup, open file descriptors, reserve resources the process will need |
| `init(pid)` | Once the process is spawned, before the measurements | Attach to the process, start a polling task |
| `join` | When profiling is over | Release resources, stop background tasks |

`from_config` runs earlier still, at source construction, and should stay cheap: anything that touches the
system belongs in `pre_init` or `init`.

### Best Practices

When implementing a metric source, keep the `measure` method lightweight and fast; any heavy computation or data processing should be done in `to_metrics` to ensure measurements do not introduce overhead or slow down profiling.

## How to Register a Source

### From the library

Build your source with `from_config` and register it with the `JouleProfiler` instance before profiling.

```rs
use std::time::Duration;

use joule_profiler_core::JouleProfiler;
use joule_profiler_core::config::ProfileConfig;
use joule_profiler_core::source::MetricReader;
use my_custom_source::{MySource, MySourceConfig};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // 1. Create the profiler
    let mut profiler = JouleProfiler::new();

    // 2. Build your custom source from its configuration
    let my_source = MySource::from_config(MySourceConfig::default())?;

    // 3. Register the source
    profiler.add_source(my_source);

    // 4. Start profiling
    let profile_config = ProfileConfig {
        stdout_file: None,
        cmd: vec!["sleep".into(), "1".into()],
        token_pattern: "__[A-Z0-9_]+__".into(),
        use_root: false,
        init_timeout: Duration::from_secs(1),
    };
    let results = profiler.profile(&profile_config).await?;

    // 5. Use the results as you need
    println!("{results:#?}");

    Ok(())
}
```

### From the CLI

Sources built into the CLI are registered through `register_source`, which reads the `[sources.<id>]`
section of the configuration file, deserializes it into `Self::Config`, and calls `from_config` only if
the source is enabled:

```rs
use joule_profiler_cli::config::source::register_source;
use my_custom_source::MySource;

register_source::<MySource>(&mut profiler, &mut config_table)?;
```

The source is then configured through the configuration file, under the key returned by `get_id()`:

```toml
[sources.my_source]
ignore_on_failure = true
step = 2
```

`ignore_on_failure` is handled by the profiler itself: when set, a `from_config` failure is logged as a
warning and the source is skipped instead of aborting the run.
