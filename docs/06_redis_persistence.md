# Persistence & Data Storage

Redis stores all data in RAM, enabling reads and writes in the nanosecond-to-microsecond range with no disk I/O bottleneck. The trade-off is that data is volatile by default — restarting the Redis process discards everything in memory.

To avoid data loss, Redis provides two persistence mechanisms: **RDB** (periodic snapshots) and **AOF** (write-ahead logging). They can be used independently or combined.

## RDB (Redis Database Snapshots)

RDB creates point-in-time snapshots of the entire dataset and writes them to a compact binary file (`dump.rdb`).

### Configuration

The snapshot schedule is defined in `redis.conf` using the `save` directive:

    save 900 1      # Snapshot after 900s if at least 1 key changed
    save 300 10     # Snapshot after 300s if at least 10 keys changed
    save 60 10000   # Snapshot after 60s if at least 10000 keys changed

### Manual Triggers

| Command    | Description                                                      |
|------------|------------------------------------------------------------------|
| `BGSAVE`   | Fork a child process to write the snapshot (non-blocking).       |
| `SAVE`     | Write the snapshot synchronously (blocks the server — avoid in production). |
| `LASTSAVE` | Return the Unix timestamp of the last successful save.           |

Example:

    > BGSAVE
    Background saving started

    > LASTSAVE
    (integer) 1739757732

### Trade-offs

| Advantage                                | Disadvantage                                      |
|------------------------------------------|---------------------------------------------------|
| Fast backup and restore (compact format) | Data written between snapshots is lost on crash   |
| Minimal runtime performance impact       | Not suitable when zero data loss is required      |

## AOF (Append-Only File)

AOF logs every write command to a file (`appendonly.aof`), producing a complete, replayable history of all modifications.

### Enabling AOF

In `redis.conf`, set:

    appendonly yes

Then restart the service:

    $ sudo systemctl restart redis

Verify the setting is active:

    $ redis-cli CONFIG GET appendonly

### Sync Policy

The `appendfsync` directive controls how often buffered writes are flushed to disk:

| Value      | Behaviour                                                        |
|------------|------------------------------------------------------------------|
| `always`   | Flush after every write (safest, highest latency)                |
| `everysec` | Flush once per second (recommended balance of safety and speed)  |
| `no`       | Let the OS flush at its discretion (fastest, risk of data loss)  |

### AOF Rewrite

Because every write is appended, the AOF file grows continuously. Redis periodically rewrites it in the background to remove redundant commands and compact the log. A manual rewrite can be triggered with:

    $ redis-cli BGREWRITEAOF

### Trade-offs

| Advantage                                         | Disadvantage                                 |
|---------------------------------------------------|----------------------------------------------|
| Near-zero data loss (with `everysec` or `always`) | Larger on-disk footprint than RDB            |
| Full recovery of recent writes                    | Slightly higher write latency                |

## Hybrid Persistence (RDB + AOF)

Since Redis 4.0, enabling `aof-use-rdb-preamble yes` combines both strategies: the AOF file begins with an RDB snapshot for fast startup, followed by incremental AOF entries for writes that occurred after the snapshot. This provides compact storage and quick restarts from RDB while retaining the durability guarantees of AOF for recent data.

## Disk-Based Storage with Redis Modules

Extensions such as Redis on Flash (available in Redis Enterprise) allow infrequently accessed values to be offloaded to SSD while hot data remains in RAM. This reduces memory costs for large datasets without changing the Redis API.
