## Redis Lua

Lua is a lightweight, high-level, multi-paradigm programming language designed mainly for embedded use in applications.

Redis Lua is a scripting feature in Redis that allows users to execute custom logic directly on the Redis server using the Lua programming language.

Redis executes Lua scripts using the `EVAL` or `EVALSHA` commands:

| Command     | Description                                                                                                          |
|-------------|----------------------------------------------------------------------------------------------------------------------|
| **EVAL**    | Executes a Lua script in Redis. The script is provided as a string and can interact with Redis keys and arguments.   |
| **EVALSHA** | Executes a Lua script using its SHA1 digest instead of providing the script directly.                                |
|             | The script must have been previously loaded using `SCRIPT LOAD`.                                                     |

Lua facilitates server-side scripting.

It has faster execution since the script runs directly inside Redis, avoiding network latency.

Moreover, the entire script runs as a **single transaction** without interference from other commands.

This makes Redis Lua ideal for implementing transactional logic, counters, rate limiting, and other advanced use cases efficiently.

## EVAL

The generic syntax of the `EVAL` command in Redis is:

    EVAL <lua_script> <num_keys> <key1> <key2> ... <arg1> <arg2> ...

Parameters:

- `<lua_script>` → The Lua script to execute.
- `<num_keys>` → The number of keys the script will use.
- `<key1>`, `<key2>`, ... → The actual Redis keys that the script operates on.
- `<arg1>`, `<arg2>`, ... → Additional arguments passed to the script.

Open an interactive session.

    $ redis-cli

To print a message (no keys):

    > EVAL "return 'Hello'" 0

    "Hello"

Passing two keys and three arguments to the Lua script:

    > EVAL "return { KEYS[1], KEYS[2], ARGV[1], ARGV[2], ARGV[3] }" 2 key1 key2 arg1 arg2 arg3

    1) "key1"
    2) "key2"
    3) "arg1"
    4) "arg2"
    5) "arg3"

The number 2 in the command specifies that the first two arguments are keys and the rest are additional arguments.

`KEYS` contains the list of keys provided in the command (in this case, "key1" and "key2").

`ARGV` holds additional arguments ("arg1", "arg2", "arg3").

You can print `KEYS` and `ARGV`:

    > EVAL "return { KEYS, ARGV }" 2 key1 key2 arg1 arg2 arg3

    1) 1) "key1"
       2) "key2"
    2) 1) "arg1"
       2) "arg2"
       3) "arg3"

## Interacting with Redis

To call Redis commands from a Lua script, we can use either of these:

- `redis.call()`: Throws an error if a command fails.
- `redis.pcall()`: Returns an error message instead of failing.

Generic syntax of `redis.call` is:

    redis.call("COMMAND", key, arg1, arg2, ...)

To set `mykey` to "Hello from Lua":

    redis.call('SET', 'mykey', 'Hello from Lua')

To get value of `mykey`:

    redis.call('GET', 'mykey')

## Embedded Lua Scripts

To increment `mycounter` by 5:

    > EVAL "return redis.call('INCRBY', KEYS[1], ARGV[1])" 1 mycounter 5

To set `mykey` to "Hello, Redis!" and then get its value:

    > EVAL "redis.call('SET', KEYS[1], ARGV[1]); return redis.call('GET', KEYS[1])" 1 mykey "Hello, Redis!"

The syntax of the `EVAL` command in Redis, especially with embedded Lua scripts, can be hard to read when written inline.

A more readable approach is to write the Lua script separately and call `EVAL`.

Save the following Lua script into `script.lua`:

```lua
local value = redis.call("GET", KEYS[1])

if value then
    return "Value: " .. value
else
    return "Key does not exist"
end
```

Then run it with:

    $ redis-cli EVAL "$(cat script.lua)" 1 mykey
    "Value: Hello, Redis!"

    $ redis-cli EVAL "$(cat script.lua)" 1 mykey1
    "Key does not exist"

## EVALSHA

Using `EVALSHA` requires first loading the script into Redis and obtaining its SHA1 hash.

    $ redis-cli SCRIPT LOAD "$(cat script.lua)"

This will return a script hash (e.g., abcdef1234567890...).

Then, we can execute the script using `EVALSHA`:

    $ redis-cli EVALSHA <script_hash> 1 mykey

Using `SCRIPT LOAD` and `EVALSHA` is more efficient than `EVAL` because Redis caches the script.

The script can be reused without redefining it every time.

Moreover, this avoids transmitting large scripts over the network repeatedly.

## Transactional Operations

Lua scripts run atomically, meaning they execute as a single, uninterrupted operation, ensuring consistency without race conditions.

`WATCH`, `MULTI`, and `EXEC` are used in Redis transactions to prevent race conditions in client-side scripts.

However, they are unnecessary in Lua scripts.

Consider the following Lua script:

```lua
local balance = redis.call("GET", "balance")

if balance and tonumber(balance) >= 100 then
    redis.call("WATCH", "balance")
    redis.call("MULTI")
    redis.call("DECRBY", "balance", 100)
    return redis.call("EXEC")
else
    redis.call("UNWATCH")
    return "Insufficient funds"
end
```

It can be simplified to:

```lua
local balance = redis.call("GET", "balance")

if balance and tonumber(balance) >= 100 then
    return redis.call("DECRBY", "balance", 100)
else
    return "Insufficient funds"
end
```

Since Redis ensures atomic execution of Lua scripts, no other client can modify balance while the script is running.

This version achieves the same result with less complexity and better performance.

## Error Handling

When using `redis.call()`, if an error occurs, the script will immediately terminate with an error.

No further commands in the script will execute.

Redis treats Lua scripts as atomic transactions, meaning all changes made before the error will be discarded (i.e., rolled back).

The client executing the script will receive the error message, and the script won’t partially apply any changes.

To prevent this, you can use `redis.pcall()` instead.

If an error occurs, `pcall()` will return an error message instead of terminating the script.

```lua
local result = redis.pcall("INCR", KEYS[1])

if type(result) == "string" and string.find(result, "ERR") then
    return "Error occurred: " .. result
else
    return result
end
```
