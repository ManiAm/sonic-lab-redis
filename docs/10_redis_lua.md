# Redis Lua Scripting

Lua is a lightweight, high-level programming language designed for embedding in applications. It compiles source code to bytecode, which runs on a built-in virtual machine. Redis integrates Lua as a server-side scripting engine, allowing you to execute custom logic directly on the Redis server.

Because the script runs inside the Redis server, it avoids network round-trips for each command, resulting in faster execution. The entire script also runs as a **single atomic operation** — no other command can execute while the script is running (see [Transactions](09_redis_transaction.md) for background on atomicity in Redis). This makes Lua scripting ideal for implementing counters, rate limiting, conditional updates, and other multi-step logic.

## EVAL

The `EVAL` command executes a Lua script on the server:

    EVAL <lua_script> <num_keys> <key1> <key2> ... <arg1> <arg2> ...

Parameters:

- `<lua_script>` — The Lua script to execute.
- `<num_keys>` — The number of Redis keys the script will access.
- `<key1>`, `<key2>`, ... — Redis keys, accessible inside Lua as `KEYS[1]`, `KEYS[2]`, etc.
- `<arg1>`, `<arg2>`, ... — Additional arguments, accessible as `ARGV[1]`, `ARGV[2]`, etc.

Open an interactive session:

    $ redis-cli

Return a simple value (no keys or arguments):

    > EVAL "return 'Hello'" 0

    "Hello"

Pass two keys and three arguments:

    > EVAL "return { KEYS[1], KEYS[2], ARGV[1], ARGV[2], ARGV[3] }" 2 key1 key2 arg1 arg2 arg3

    1) "key1"
    2) "key2"
    3) "arg1"
    4) "arg2"
    5) "arg3"

The number `2` tells Redis that the first two positional arguments after the script are keys; the rest are treated as additional arguments.

Return `KEYS` and `ARGV` as separate arrays:

    > EVAL "return { KEYS, ARGV }" 2 key1 key2 arg1 arg2 arg3

    1) 1) "key1"
       2) "key2"
    2) 1) "arg1"
       2) "arg2"
       3) "arg3"

## Calling Redis Commands from Lua

To invoke Redis commands within a Lua script, use `redis.call()`:

    redis.call("COMMAND", key, arg1, arg2, ...)

For example, increment `mycounter` by 5:

    > EVAL "return redis.call('INCRBY', KEYS[1], ARGV[1])" 1 mycounter 5

    (integer) 5

Set a key and then retrieve its value in the same script:

    > EVAL "redis.call('SET', KEYS[1], ARGV[1]); return redis.call('GET', KEYS[1])" 1 mykey "Hello, Redis!"

    "Hello, Redis!"

Redis also provides `redis.pcall()`, which behaves identically to `redis.call()` except in how it handles errors. The difference is covered in [Error Handling](#error-handling).

## Running Scripts from Files

Inline scripts become hard to read as they grow. A better approach is to write the Lua script in a separate file and pass it to `redis-cli`.

Save the following as `script.lua`:

```lua
local value = redis.call("GET", KEYS[1])

if value then
    return "Value: " .. value
else
    return "Key does not exist"
end
```

Run it from the command line:

    $ redis-cli EVAL "$(cat script.lua)" 1 mykey
    "Value: Hello, Redis!"

    $ redis-cli EVAL "$(cat script.lua)" 1 nonexistent
    "Key does not exist"

## EVALSHA

Every time `EVAL` is called, Redis must parse and compile the script to bytecode. For scripts that run repeatedly, this overhead is unnecessary. `EVALSHA` solves this by executing a previously loaded script using its SHA1 hash.

First, load the script into Redis:

    $ redis-cli SCRIPT LOAD "$(cat script.lua)"
    "a1b2c3d4e5f6..."

Redis returns the SHA1 hash of the compiled script. Then execute it with `EVALSHA`:

    $ redis-cli EVALSHA a1b2c3d4e5f6... 1 mykey

| Command         | Description                                                        |
|-----------------|--------------------------------------------------------------------|
| **EVAL**        | Sends and executes the full script text every time.                |
| **EVALSHA**     | Executes a cached script by its SHA1 hash.                         |
| **SCRIPT LOAD** | Loads a script into the cache and returns its SHA1 hash.           |

Using `SCRIPT LOAD` + `EVALSHA` avoids transmitting large scripts over the network on each call and skips repeated compilation.

## Atomicity and Transactions

Lua scripts execute atomically — no other client command can run while a script is in progress. This means the `WATCH`, `MULTI`, and `EXEC` commands used in Redis transactions (see [Transactions](09_redis_transaction.md)) are unnecessary inside Lua scripts. In fact, Redis will return an error if you attempt to use them within a script.

A beginner might be tempted to write:

```lua
local balance = redis.call("GET", "balance")

if balance and tonumber(balance) >= 100 then
    redis.call("WATCH", "balance")       -- ERROR: not allowed in scripts
    redis.call("MULTI")
    redis.call("DECRBY", "balance", 100)
    return redis.call("EXEC")
else
    return "Insufficient funds"
end
```

The correct approach relies on the script's built-in atomicity:

```lua
local balance = redis.call("GET", "balance")

if balance and tonumber(balance) >= 100 then
    return redis.call("DECRBY", "balance", 100)
else
    return "Insufficient funds"
end
```

No other client can modify `balance` between the `GET` and the `DECRBY` because the entire script runs without interruption. This achieves the same guarantee with less complexity.

## Error Handling

`redis.call()` and `redis.pcall()` differ in how they handle runtime errors:

| Function         | On Error                                                    |
|------------------|-------------------------------------------------------------|
| `redis.call()`   | Terminates the script immediately and returns the error.    |
| `redis.pcall()`  | Returns an error object, allowing the script to continue.   |

**Important:** Redis does **not** roll back commands that succeeded before the error. Any writes that completed prior to the failure will persist. This is consistent with Redis transaction semantics — Redis never provides rollback.

Example with `redis.call()`:

```lua
redis.call("SET", "key1", "value1")   -- succeeds, persists
redis.call("INCR", "key2")            -- fails if key2 holds a non-numeric string
redis.call("SET", "key3", "value3")   -- never reached
```

If `key2` contains a non-numeric string, the script terminates at the second command. `key1` retains its new value, but `key3` is never set.

To handle errors gracefully, use `redis.pcall()`:

```lua
local result = redis.pcall("INCR", KEYS[1])

if result.err then
    return "Error occurred: " .. result.err
else
    return result
end
```

This allows the script to implement fallback logic or return meaningful error messages instead of terminating.
