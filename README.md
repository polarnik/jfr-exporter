# JfrExporter

Send JFR events to time series databases.

Makes use of [JFR Event Streaming](https://openjdk.org/jeps/349) as found in hotspot based JDK 14+.

Steps:
1. create a database in influx or use an existing one 
2. put the agent jar on the classpath, point it to InfluxDB
3. create a datasource in Grafana that points to the InfluxDB database
4. import the Grafana dashboard from the `dashboards` directory, choose InfluxDB datasource
5. start the application
6. optional: start load test

## Download

Download the latest release from the [releases page](https://github.com/perfana/jfr-exporter/releases).

## Agent

To use JfrExporter simply add `-javaagent:/path/to/jfr-exporter.jar` to your JVM arguments.

There is no need to enable JFR in the JVM arguments (`-XX:StartFlightRecording`).

Options can be sent as `-javaagent:/path/to/jfr-exporter.jar=option1=value1,option2=value2`.

Example: `-javaagent:/path/to/jfr-exporter.jar=debug,application=afterburner`, which will enable
debug logging and set the application name to "afterburner".

When used as agent, there is no need to add JFR activation to the JVM arguments.

Be aware that some shells require a backslash before the equal signs.

## Standalone

JfrExporter can also be used as a standalone application, for example to monitor a running process.

The monitored process needs to be started with JFR enabled: `-XX:StartFlightRecording`

```bash
Usage: java JfrExporter 
 --debug,-d 
 --processId,-p <processId> 
 --duration <ISO-duration> 
 --application,-a <application>
 --bigObjectThreshold <bytes>
 --influxUrl <influxUrl> 
 --influxDatabase <influxDatabase>
 --influxUser <influxUser> 
 --influxPassword <influxPassword>
```

The default InfluxDB database name is `jfr`.

Example to connect to process with id 1234 and send events with application name afterburner:
```bash
java -jar jfr-exporter.jar --processId 1234 --application afterburner \
  --duration PT30s --influxUrl http://localhost:8086
```

To enable extra monitoring, such as safe points, or different thresholds, 
use a saved JFR profile in the JDK used, for example saved as `mySettings`: `-XX:StartFlightRecording:settings=mySettings`

## Events

Currently a subset of JFR events are processed. 
* CPU load
* Garbage Collection
* Memory (heap usage, large allocations)
* Safepoints
* Threads
* Classloaders

For reference: [list of JFR events](https://bestsolution-at.github.io/jfr-doc/index.html).

## Dashboard

A Grafana dashboard can be imported to view the JFR metrics.
Import the dashboard in the `dashboards` directory into Grafana and
connect to an InfluxDB datasource that points to the `jfr` database.

## Troubleshoot

Use `-Dio.perfana.jfr.debug=true` to enable debug logging or `--debug` as argument.

For tracing (more debug logging) use: `-Dio.perfana.jfr.trace=true`

Debug and tracing will output a lot of data, so only use for troubleshooting.