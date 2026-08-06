# Key Management Commands

This section covers commands for managing databases, inspecting key internals, and renaming, moving, or removing keys. These complement the basic key operations (`SET`, `GET`, `DEL`, `EXISTS`, `TYPE`) introduced in [Data Types](02_redis_data_types.md) and the search commands (`KEYS`, `SCAN`) in [Key Scanning & Listing](04_redis_key_search.md).

## Database Commands

Redis supports multiple logical databases (numbered 0–15 by default), as described in [Getting Started](01_redis.md#redis-databases). The commands below operate at the database level.

| Command    | Description                                                |
|------------|------------------------------------------------------------|
| `DBSIZE`   | Get the number of keys in the currently selected database. |
| `FLUSHDB`  | Remove all keys from the currently selected database.      |
| `FLUSHALL` | Remove all keys from all databases.                        |

Example:

    > SELECT 0
    OK
    > DBSIZE
    (integer) 12

**Warning:** `FLUSHDB` and `FLUSHALL` permanently delete data. Use them with extreme caution in production environments.

## Key Inspection — OBJECT

The `OBJECT` command reveals how Redis internally stores a key's value. This is useful for debugging performance and understanding memory usage.

    OBJECT <subcommand> <key>

| Subcommand | Description                                                                        |
|------------|------------------------------------------------------------------------------------|
| `ENCODING` | Returns the internal encoding used to store the value (e.g., `int`, `embstr`, `raw`, `ziplist`, `hashtable`). |
| `IDLETIME` | Returns the number of seconds since the key was last read or written.              |
| `REFCOUNT` | Returns the reference count of the value object (used internally by Redis).        |
| `FREQ`     | Returns the logarithmic access frequency counter. Only available when the `maxmemory-policy` is set to an LFU variant (e.g., `allkeys-lfu`). |

Example — checking the encoding of different value types:

    > SET mykey "hello"
    > OBJECT ENCODING mykey
    "embstr"

    > SET counter 12345
    > OBJECT ENCODING counter
    "int"

    > LPUSH mylist "a" "b" "c"
    > OBJECT ENCODING mylist
    "listpack"

The encoding depends on the value's type and size. Redis automatically promotes to a larger encoding (e.g., `listpack` → `quicklist`) as the data grows.

## Key Manipulation

### Renaming Keys

| Command    | Description                                                                                     | Example                 |
|------------|-------------------------------------------------------------------------------------------------|-------------------------|
| `RENAME`   | Renames a key. If the destination key already exists, its value is overwritten.                  | `RENAME oldkey newkey`  |
| `RENAMENX` | Renames a key only if the destination key does not exist. Returns `1` on success, `0` otherwise. | `RENAMENX oldkey newkey`|

Example:

    > SET greeting "hello"
    > RENAME greeting message
    OK
    > GET message
    "hello"

Use `RENAMENX` when you need to avoid accidentally overwriting an existing key:

    > SET a "value_a"
    > SET b "value_b"
    > RENAMENX a b
    (integer) 0          # b already exists — rename did not happen

### Moving Keys Between Databases

`MOVE` transfers a key from the current database to another. The command fails if the key already exists in the target database.

    MOVE key db

Example:

    > SET temp "data"
    > MOVE temp 2        # moves "temp" to database 2
    (integer) 1
    > SELECT 2
    > GET temp
    "data"

### Serializing and Restoring Keys — DUMP & RESTORE

`DUMP` serializes a key's value into a Redis-specific binary format (including type information and a checksum). `RESTORE` deserializes the data back into a key. Together they enable migrating individual keys between Redis instances.

| Command   | Description                                                                 |
|-----------|-----------------------------------------------------------------------------|
| `DUMP`    | Returns a serialized binary representation of the key's value.              |
| `RESTORE` | Creates a key from a serialized value produced by `DUMP`, with an optional TTL. |

    DUMP key
    RESTORE key ttl serialized-value [REPLACE]

- `ttl` — expiration in milliseconds. Use `0` for no expiration.
- `REPLACE` — if provided, overwrites the key if it already exists.

Example:

    > SET mykey "hello"
    > DUMP mykey
    "\x00\x05hello\n\x00\x8f<r\xc3\xcaJ\xe7\xc2"

    > DEL mykey
    > RESTORE mykey 0 "\x00\x05hello\n\x00\x8f<r\xc3\xcaJ\xe7\xc2"
    OK
    > GET mykey
    "hello"

### Removing Keys Asynchronously — UNLINK

`UNLINK` removes one or more keys, similar to `DEL`, but reclaims memory in a background thread instead of blocking the server.

    UNLINK key [key ...]

Example:

    > UNLINK mykey otherkey
    (integer) 2

Prefer `UNLINK` over `DEL` when deleting large keys (e.g., a set with millions of members or a very long list), since `DEL` blocks the event loop until the memory is fully freed.
