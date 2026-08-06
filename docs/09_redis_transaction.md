## Redis Transactions

Redis transactions allow executing multiple commands atomically using these commands:

| Command   | Description                                                                                            |
|-----------|--------------------------------------------------------------------------------------------------------|
| `MULTI`   | Marks the start of a transaction. Commands issued after `MULTI` are queued until `EXEC` is called.     |
| `EXEC`    | Executes all queued commands in the transaction.                                                       |
| `DISCARD` | Cancels a transaction and clears the command queue.                                                    |

Here is a simple Redis transaction:

    > MULTI              # Start a transaction
    > SET key1 "value1"
    > SET key2 "value2"
    > INCR counter
    > EXEC               # Execute all commands atomically

Sample output:

    OK
    QUEUED
    QUEUED
    QUEUED
    1) OK
    2) OK
    3) (integer) 1

If a command fails inside a transaction, Redis does not roll back automatically.

    > MULTI
    > SET key1 "value1"
    > INCR key1            # Error (key1 is not an integer)
    > SET key2 "value2"
    > EXEC

Only the successful commands before the error execute.

## WATCH

WATCH in Redis is used for optimistic locking, monitoring one or more keys for changes before executing a transaction.

If any of the watched keys are modified by another client before `EXEC` is called, the transaction is aborted, ensuring data consistency.

This is useful in scenarios where multiple clients might modify the same key, preventing race conditions.

`UNWATCH` can be used to remove the watch before executing other commands outside the transaction.

**Preventing Overwrites in a Counter**

Imagine a client is trying to increment `counter`.

```
WATCH counter     # Monitor the counter key
MULTI             # Start transaction
INCR counter      # Increment the counter
EXEC              # Commit the transaction
```

If another client changes `counter` after `WATCH` but before `EXEC`, the transaction is aborted.

**Safe Balance Deduction**

Scenario: Preventing race conditions when deducting money from an account.

Steps:

- `WATCH` the balance key
- Check if there are enough funds
- If yes, start a transaction (`MULTI`), deduct the amount, and `EXEC` it
- If the key changes before `EXEC`, the transaction fails

Here is a pseudocode showing the flow:

```
WATCH balance           # Monitor the balance key for changes
balance = GET balance   # Read the current balance
IF balance >= 100
    MULTI               # Start transaction
    DECRBY balance 100  # Deduct 100
    EXEC                # Commit the transaction
ELSE
    UNWATCH             # Stop watching if conditions aren't met
    PRINT "Insufficient funds"
```
