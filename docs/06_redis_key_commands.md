## Object Command

The `OBJECT` command provides information about a key.

It has the following syntax:

    > OBJECT <subcommand> <key>

| Subcommand        | Description                                                                                                           |
|-------------------|-----------------------------------------------------------------------------------------------------------------------|
| `ENCODING <key>`  | Returns the internal representation used to store the value of the key.                                               |
| `FREQ <key>`      | Returns the access frequency index of the key, which is proportional to the logarithm of the recent access frequency. |
| `IDLETIME <key>`  | Returns the idle time of the key, i.e., the number of seconds since its last access.                                  |
| `REFCOUNT <key>`  | Returns the number of references to the value associated with the key.                                                |

## Database & Key Management

| Command    | Description                                                |
|------------|------------------------------------------------------------|
| `DBSIZE`   | Get the number of keys in the currently selected database. |
| `FLUSHALL` | Remove all keys from all databases.                        |
| `FLUSHDB`  | Remove all keys from the currently selected database.      |

## Key Manipulation

| Command    | Description                                       | Example                          |
|------------|---------------------------------------------------|----------------------------------|
| `RENAME`   | Renames a key.                                    | `RENAME mykey newkey`            |
| `RENAMENX` | Renames a key only if the new key does not exist. | `RENAMENX mykey newkey`          |
| `MOVE`     | Moves a key to another Redis database.            | `MOVE mykey 2`                   |
| `UNLINK`   | Removes a key asynchronously.                     | `UNLINK mykey`                   |
| `RESTORE`  | Restores a key from a serialized value.           | `RESTORE mykey 0 serializeddata` |
