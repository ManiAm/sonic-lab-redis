# Key Scanning & Listing

As your Redis database grows, you need ways to find and browse keys. Redis provides commands for pattern-based searching, incremental scanning, and sorting.

| Command     | Description                                        | Example                          |
|-------------|----------------------------------------------------|----------------------------------|
| `KEYS`      | Lists all keys matching a pattern.                 | `KEYS *`                         |
| `SCAN`      | Iterates over keys in the database (cursor-based). | `SCAN 0 MATCH prefix:* COUNT 10` |
| `RANDOMKEY` | Returns a random key from the database.            | `RANDOMKEY`                      |
| `SORT`      | Sorts the elements in a list, set, or sorted set.  | `SORT mylist ASC`                |

## KEYS Command

Returns all keys matching a given glob-style pattern.

    KEYS pattern

Supported wildcards:

- `*` matches any sequence of characters.
- `?` matches exactly one character.
- `[abc]` matches any single character in the brackets.

Examples:

    > KEYS *
    > KEYS user:*
    > KEYS config:[0-9]*

**Warning:** `KEYS` blocks the server while scanning the entire keyspace. Use it only for debugging or against small databases. For production use, prefer `SCAN`.

## SCAN Command

`SCAN` provides the same pattern-matching capability as `KEYS` but iterates incrementally using a cursor, so it never blocks the server for extended periods.

    SCAN cursor [MATCH pattern] [COUNT count]

- `cursor` — position in the iteration (start with `0`).
- `MATCH pattern` — optional glob-style filter (same syntax as `KEYS`).
- `COUNT count` — optional hint for how many keys to return per call (default `10`).

Start scanning from cursor 0:

    > SCAN 0
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

The first element (`"15"`) is the next cursor. The second element is the batch of keys found. Continue scanning with the returned cursor until it returns `0`, which signals completion:

    > SCAN 15

Increase the batch size for fewer round-trips:

    > SCAN 0 COUNT 100

Filter by pattern:

    > SCAN 0 MATCH user:*

Note: `COUNT` is a hint, not a guarantee — Redis may return slightly more or fewer keys per call.

## RANDOMKEY Command

Returns a random key from the currently selected database. Useful for sampling or quick debugging:

    > RANDOMKEY
    "user:1001"

Returns `(nil)` if the database is empty.

## SORT Command

Sorts elements stored in a list, set, or sorted set and returns the sorted result without modifying the original key.

    SORT key [ASC|DESC] [ALPHA] [LIMIT offset count]

- `ASC` (default) — ascending order; `DESC` — descending order.
- `ALPHA` — required when sorting string values (by default, `SORT` assumes numeric values).
- `LIMIT offset count` — return only `count` elements starting at `offset`.

Sort a list of numbers:

    > RPUSH scores 5 2 8 1 4
    > SORT scores
    1) "1"
    2) "2"
    3) "4"
    4) "5"
    5) "8"

Sort in descending order:

    > SORT scores DESC
    1) "8"
    2) "5"
    3) "4"
    4) "2"
    5) "1"

Sort strings alphabetically:

    > SADD names "Charlie" "Alice" "Bob"
    > SORT names ALPHA
    1) "Alice"
    2) "Bob"
    3) "Charlie"

Limit the result:

    > SORT scores LIMIT 0 3
    1) "1"
    2) "2"
    3) "4"

To persist the sorted result, use `SORT key STORE destination_key`.
