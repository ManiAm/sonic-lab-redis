## Persistence & Data Storage

Redis is designed for ultra-low-latency operations, making it one of the fastest key-value stores.

By keeping all data in RAM:

- Reads and writes are nearly instant (nanoseconds to microseconds).
- There is no disk I/O bottleneck (compared to databases that rely on HDD/SSD storage).
- Data structures (lists, sets, hashes) are optimized for in-memory access.

By default, Redis does not guarantee data persistence unless explicitly configured.

Without persistence, restarting the Redis service would wipe out all data because Redis primarily stores data in RAM for fast access.

Redis provides two main persistence mechanisms to avoid data loss: `RDB` and `AOF`

## RDB (Redis Database File - Snapshotting)

Creates point-in-time snapshots of the dataset at specified intervals.

Saves the database to disk in a compact binary format (`dump.rdb`).

Configured using the `SAVE` or `BGSAVE` directives in `redis.conf`.

| Command        | Description                                           |
|----------------|-------------------------------------------------------|
| `SAVE`         | Synchronously save the dataset to disk.               |
| `BGSAVE`       | Asynchronously save the dataset to disk.              |
| `LASTSAVE`     | Get the Unix timestamp of the last successful `SAVE`. |

For example, to save the dataset to disk every 15 minutes if at least one change is made:

    > SAVE 900 1

RDB Summary:

- Pros: Faster for backup and restore, minimal performance impact.
- Cons: Data loss can occur between snapshots (not real-time).

## AOF (Append-Only File)

Logs every write operation and appends it to a file (`appendonly.aof`).

To enable AOF persistence, edit the Redis configuration file (`redis.conf`) and look for the following directive:

    appendonly no

Change it to:

    appendonly yes

The `appendfsync` setting determines how frequently Redis writes AOF data to disk:

- appendfsync always      # Writes every operation immediately (slowest but safest)
- appendfsync everysec    # Writes once per second (recommended for a balance of speed and durability)
- appendfsync no          # Lets the OS decide when to write (fastest but risk of data loss)

After modifying `redis.conf`, restart Redis:

    $ sudo systemctl restart redis

To confirm AOF is enabled, run:

    $ redis-cli CONFIG GET appendonly

Over time, the AOF file grows large.

Redis periodically rewrites the file to remove redundant commands.

You can manually trigger this with:

    $ redis-cli BGREWRITEAOF

This runs the AOF rewrite in the background to optimize file size.

AOF Summary:

- Pros: More reliable, allows full recovery of recent data.
- Cons: Larger file size, slower than RDB due to continuous writes.

## Hybrid Approach (RDB + AOF)

Redis 4.0 introduced an option to use both `RDB` and `AOF` together (`aof-use-rdb-preamble`).

This provides the best of both worlds: efficient storage from `RDB` and durability from `AOF`.

## Disk-Based Storage with Redis Modules

Some modules (like Redis on Flash or Redis Enterprise) allow Redis to store less frequently accessed data on SSDs to reduce memory usage.
