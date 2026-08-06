# Memory & Performance Monitoring

| Command    | Description                                                     |
|------------|-----------------------------------------------------------------|
| `MEMORY`   | Get memory usage statistics and memory-related diagnostics.     |
| `SLOWLOG`  | View the Redis slow query log.                                  |
| `LATENCY`  | Measure and diagnose Redis command latencies.                   |
| `MONITOR`  | Stream all commands processed by the Redis server in real-time. |

## MEMORY

The `MEMORY` command provides insight into how Redis allocates and consumes memory.

To check how much memory a specific key uses (in bytes):

    > MEMORY USAGE mykey

To run a built-in diagnostic that reports potential memory issues:

    > MEMORY DOCTOR

    Sam, I have no memory problems

To view detailed memory allocation statistics:

    > MEMORY STATS

## SLOWLOG

The slow log is an internal list that records commands whose execution time exceeds a configurable threshold. It measures only the time spent executing the command on the server, not network or I/O latency.

To check the current threshold (in microseconds):

    > CONFIG GET slowlog-log-slower-than

The default threshold is 10000 microseconds (10 ms). To lower it:

    > CONFIG SET slowlog-log-slower-than 5000

To view all slow log entries:

    > SLOWLOG GET

To view only the last N entries:

    > SLOWLOG GET 5

To check how many entries are currently stored:

    > SLOWLOG LEN

To clear the slow log:

    > SLOWLOG RESET

## LATENCY

The `LATENCY` subsystem samples and records latency spikes across different event types (e.g., command execution, fork operations, AOF writes). Unlike `SLOWLOG`, which records individual slow commands, `LATENCY` tracks system-level events over time.

Latency monitoring is disabled by default. To enable it, set a threshold in milliseconds — any event exceeding this value will be recorded:

    > CONFIG SET latency-monitor-threshold 100

To view the most recent latency spike for each event type:

    > LATENCY LATEST

To view the full history of a specific event type:

    > LATENCY HISTORY command

To reset all recorded latency data:

    > LATENCY RESET

## MONITOR

`MONITOR` is a debugging command that streams every command processed by the server in real-time. It is useful for troubleshooting unexpected behavior and understanding client interactions.

Since `MONITOR` logs every command, it imposes significant performance overhead and should not be left running in production.

This is a blocking call — it keeps the connection open and continuously outputs data:

    $ redis-cli monitor

Sample output:

    OK
    1739757732.123456 [0 127.0.0.1:54321] "SET" "mykey" "hello"
    1739757733.654321 [0 127.0.0.1:54321] "GET" "mykey"

Each line shows the Unix timestamp, database index, client address, and the full command with arguments.

## Redis GUI

Redis GUIs provide a graphical interface for managing, monitoring, and debugging Redis databases. They offer features such as key browsing, real-time monitoring, data editing, and performance metrics.

Popular options include:

- **RedisInsight** (by Redis)

    The official GUI with advanced visualization, key management, and analytics.

    Link: https://redis.io/insight/

- **Another Redis Desktop Manager**

    A cross-platform Redis desktop manager compatible with Linux, Windows, and Mac.

    Link: https://github.com/qishibo/AnotherRedisDesktopManager
