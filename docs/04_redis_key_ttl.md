## Key Expiration

Key expiration in Redis allows setting a time-to-live (TTL) for a key, after which it is automatically deleted.

The TTL is specified using one of these commands:

| Command    | Description                                              | Example                         |
|------------|----------------------------------------------------------|---------------------------------|
| `EXPIRE`   | Sets a timeout (in seconds) for a key.                   | `EXPIRE mykey 60`               |
| `EXPIREAT` | Sets a key's expiration time using a UNIX timestamp.     | `EXPIREAT mykey 1672531199`     |
| `PEXPIRE`  | Sets a timeout (in milliseconds) for a key.              | `PEXPIRE mykey 60000`           |
| `PEXPIREAT`| Sets expiration time using a UNIX timestamp (ms).        | `PEXPIREAT mykey 1672531199000` |

If you are working with string data type, you can use these:

| Command               | Description                                           | Example                      |
|-----------------------|-------------------------------------------------------|------------------------------|
| `SETEX key sec value` | Set a key with a value and expiration time            | `SETEX name 10 "Alice"`      |
| `PSETEX key ms value` | Set a key with a value and expiration in milliseconds | `PSETEX name 5000 "Alice"`   |
| `SETNX key value`     | Set a key with a value only if it does not exist      | `SETNX name "Alice"`         |

`SETEX` is a combination of SET and EXPIRE.

It allows you to set a key with a string value and an expiration time in seconds in a single command.

To store a string that expires after a specific time:

    > SETEX tempkey 10 "This will expire in 10 seconds"

This key will be removed automatically after 10 seconds.

You can check the remaining TTL of a key using these commands:

| Command    | Description                                           | Example        |
|------------|-------------------------------------------------------|----------------|
| `TTL`      | Returns the time to live (TTL) for a key in seconds.  | `TTL mykey`    |
| `PTTL`     | Returns the TTL for a key in milliseconds.            | `PTTL mykey`   |

If a key has no expiration, TTL returns -1.

If a key does not exist, it returns -2.

To remove the expiration from a key, use the `PERSIST` command

Key expiration is useful for caching, session management, and temporary data storage.

This mechanism helps manage memory efficiently by removing unused or outdated data automatically.
