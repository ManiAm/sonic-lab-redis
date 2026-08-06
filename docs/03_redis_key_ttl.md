# Key Expiration & TTL

Redis allows setting a time-to-live (TTL) on any key. Once the TTL elapses, the key is automatically deleted. This is the primary mechanism for managing temporary data and controlling memory usage.

## Setting Expiration on Existing Keys

Use the following commands to attach an expiration to a key that already exists:

| Command      | Unit         | Example                         |
|--------------|--------------|----------------------------------|
| `EXPIRE`     | Seconds      | `EXPIRE mykey 60`               |
| `PEXPIRE`    | Milliseconds | `PEXPIRE mykey 60000`           |
| `EXPIREAT`   | UNIX timestamp (seconds)      | `EXPIREAT mykey 1672531199`     |
| `PEXPIREAT`  | UNIX timestamp (milliseconds) | `PEXPIREAT mykey 1672531199000` |

Example — expire a key after 60 seconds:

    > SET mykey "hello"
    OK
    > EXPIRE mykey 60
    (integer) 1

## Checking Remaining TTL

After setting an expiration, you can inspect how much time a key has left:

| Command | Unit         | Example      |
|---------|--------------|--------------|
| `TTL`   | Seconds      | `TTL mykey`  |
| `PTTL`  | Milliseconds | `PTTL mykey` |

Return values:

- **Positive integer** — remaining time until expiration.
- **-1** — key exists but has no expiration set.
- **-2** — key does not exist.

## Removing Expiration

To make an expiring key persistent again (remove its TTL without deleting it), use `PERSIST`:

    > PERSIST mykey
    (integer) 1

## Setting a Value with Expiration

The `SET` command supports options that combine value assignment and expiration in a single
atomic operation:

    > SET mykey "hello" EX 60        # expires in 60 seconds
    > SET mykey "hello" PX 60000     # expires in 60000 milliseconds

| Option | Unit         | Equivalent Legacy Command |
|--------|--------------|---------------------------|
| `EX`   | Seconds      | `SETEX key sec value`     |
| `PX`   | Milliseconds | `PSETEX key ms value`     |

The legacy `SETEX` and `PSETEX` commands still work but `SET` with `EX`/`PX` is preferred
because it keeps all SET-related options in one command.

## Conditional Set with Expiration

The `NX` option on `SET` writes the key only if it does not already exist. This is commonly
used with `EX`/`PX` to implement distributed locks and caches:

    > SET mylock "acquired" NX EX 30
    OK          # key was set (did not exist)

    > SET mylock "acquired" NX EX 30
    (nil)       # key already exists — not set

Because both the write and the expiration happen in one atomic step, there is no window where
the key exists without a TTL (a risk when using separate `SETNX` + `EXPIRE` calls).

> **Note:** The standalone `SETNX` command is considered legacy. Prefer `SET ... NX` instead.

## Use Cases

- **Caching** — automatically invalidate stale cache entries.
- **Session management** — expire user sessions after a period of inactivity.
- **Rate limiting** — create counters that reset after a time window.
- **Distributed locks** — acquire a lock with an automatic expiration to prevent deadlocks.
- **Temporary data** — store one-time tokens, OTPs, or short-lived configuration.
