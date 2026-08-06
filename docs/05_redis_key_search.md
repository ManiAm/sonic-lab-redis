## Key Scanning & Listing

| Command     | Description                                        | Example                          |
|-------------|----------------------------------------------------|----------------------------------|
| `KEYS`      | Lists all keys matching a pattern.                 | `KEYS *`                         |
| `SCAN`      | Iterates over keys in the database (cursor-based). | `SCAN 0 MATCH prefix:* COUNT 10` |
| `RANDOMKEY` | Returns a random key from the database.            | `RANDOMKEY`                      |
| `SORT`      | Sorts the elements in a list, set, or sorted set.  | `SORT mylist ASC`                |

**KEYS Command**

The `KEYS` command in Redis is used to search for keys that match a specific pattern.

The basic syntax is:

    KEYS pattern

`pattern` can include wildcards like `*`, `?`, and `[]` for pattern matching.

To get all keys:

    > KEYS *

Searching for all keys starting with `user:`:

    > KEYS user:*

Find keys that start with `config:` and contain a digit:

    > KEYS config:[0-9]*

Use `KEYS` only for debugging or when dealing with a small number of keys.

It is discouraged to use `KEYS` in production environments because it can be slow if there are a large number of keys.

Instead, using `SCAN` is recommended for better performance.

**SCAN Command**

The `SCAN` command is used for iterating over keys in the database incrementally.

The basic syntax is:

    SCAN cursor [MATCH pattern] [COUNT count]

- `cursor`: A numeric value that Redis returns in each scan cycle (initially 0).
- `MATCH pattern`: (Optional) Filters keys using a pattern like `user:*`.
- `COUNT count`: (Optional) Hints how many keys to return per iteration (default is 10)

Start scanning from cursor 0:

    > SCAN 0

Sample output:

    1) "15"
    2)  1) "message"
        2) "set2"
        3) "_kombu.binding.celery"
        4) "set1"
        5) "leaderboard"
        6) "myset"
        7) "mykey"
        8) "mylist"
        9) "mystream"
    10) "user:1001"

The first item (23) is the new cursor.

The second item is a list of keys found in this scan.

To continue scanning, use the new cursor:

    > SCAN 15

Repeat this until the cursor returns 0 (which means the iteration is complete).

By default, Redis returns 10 keys per scan.

You can increase this for better performance:

    > SCAN 0 COUNT 100

This tells Redis to return up to 100 keys per iteration (though it's not guaranteed).

To get only keys that start with `user:`

    > SCAN 0 MATCH user:*
