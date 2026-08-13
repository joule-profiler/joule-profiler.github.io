# Source Implementation

Implementing a new metric source in **Joule Profiler** is straightforward. By implementing the
`MetricReader` trait, you only need to define how your source is built (`from_config`), the core
measurement logic (`measure`, `retrieve`, `get_sensors`, `to_metrics`) and how it identifies itself
(`get_name`, `get_id`). You can optionally override `pre_init`, `init`, or `join` if your source
requires setup, polling or teardown logic. This design makes it easy to add new sources without
boilerplate.

A few things to keep in mind:

- `Type` is the snapshot type produced by `retrieve` and converted by `to_metrics`. It only has to be `Send`.
- `Error` must be `std::error::Error + Send + Sync`. Sources usually define their own `thiserror` enum
  and rely on `IntoMetricSourceError` to convert it; `MetricSourceError` can be used directly for simple sources.
- `Config` must be `Default + DeserializeOwned`. It is deserialized from the source's section of the
  profiler configuration file, keyed by `get_id()`.
- `get_name()` is the human readable name displayed in the results, while `get_id()` is the identifier
  used to enable the source and look up its configuration.

```rs
use joule_profiler_core::{
    sensor::{Sensor, Sensors},
    source::{MetricReader, MetricSourceError},
    types::{Metric, Metrics},
    unit::{MetricUnit, Unit, UnitPrefix},
};
use serde::Deserialize;

const MY_SOURCE_UNIT: MetricUnit = MetricUnit {
    prefix: UnitPrefix::None,
    unit: Unit::Count,
};

const MY_SOURCE_NAME: &str = "My Source";
const MY_SOURCE_ID: &str = "my_source";

/// Configuration of the source, deserialized from the profiler configuration file.
#[derive(Debug, Default, Deserialize)]
pub struct MySourceConfig {
    /// How much the counter is incremented at each measurement.
    #[serde(default)]
    pub step: u64,
}

#[derive(Default)]
pub struct MySource {
    step: u64,
    count: u64,
}

impl MetricReader for MySource {
    /// Snapshot type produced by `retrieve` and consumed by `to_metrics`.
    type Type = u64;

    /// Error type of the source, must be `std::error::Error + Send + Sync`.
    type Error = MetricSourceError;

    /// Config type of the source, must be `Default + DeserializeOwned`.
    type Config = MySourceConfig;

    fn from_config(config: Self::Config) -> Result<Self, Self::Error> {
        Ok(Self { step: config.step, count: 0 })
    }

    /// Optional: called once before the profiled process is spawned.
    async fn pre_init(&mut self) -> Result<(), Self::Error> {
        Ok(())
    }

    /// Optional: called once the profiled process is spawned, with its pid.
    async fn init(&mut self, _pid: i32) -> Result<(), Self::Error> {
        Ok(())
    }

    /// Optional: called when profiling is over, to release resources or stop background tasks.
    async fn join(&mut self) -> Result<(), Self::Error> {
        Ok(())
    }

    async fn measure(&mut self) -> Result<(), Self::Error> {
        self.count += self.step;
        Ok(())
    }

    async fn retrieve(&mut self) -> Result<Self::Type, Self::Error> {
        let count = self.count;
        self.count = 0;
        Ok(count)
    }

    fn get_sensors(&self) -> Result<Sensors, Self::Error> {
        let sensor = Sensor::new("value", MY_SOURCE_UNIT, Self::get_name());
        Ok(vec![sensor])
    }

    fn to_metrics(&self, count: Self::Type) -> Result<Metrics, Self::Error> {
        let metric = Metric::new("value", count, MY_SOURCE_UNIT, Self::get_name());
        Ok(vec![metric])
    }

    /// Human readable name of the source, displayed in the results.
    fn get_name() -> &'static str {
        MY_SOURCE_NAME
    }

    /// Identifier of the source, used to enable it and look up its configuration.
    fn get_id() -> &'static str {
        MY_SOURCE_ID
    }
}
```