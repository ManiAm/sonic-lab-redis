## Redis Client Libraries

Redis client libraries are software packages that provide an interface for applications to interact with a Redis database.

These libraries handle the communication between the application and the Redis server.

They allow developers to execute commands like setting and retrieving key-value pairs, managing data structures, and handling pub/sub messaging.

| **Language**  | **Client Library**    | **Description**                                                                                            |
|---------------|-----------------------|------------------------------------------------------------------------------------------------------------|
| **C**         | `hiredis`             | Minimalist and high-performance C client library for Redis.                                                |
| **Go**        | `go-redis`            | Popular Go client for Redis, supports Cluster, Sentinel, and pipelining.                                   |
| **Java**      | `Jedis`               | Lightweight and easy-to-use Java Redis client with simple connection management.                           |
| **Java**      | `Lettuce`             | Scalable, reactive Redis client for Java with advanced features like connection pooling and async support. |
| **C# (.NET)** | `StackExchange.Redis` | High-performance .NET client developed by the creators of Stack Overflow.                                  |
| **Python**    | `redis-py`            | Official Redis client for Python, supports pipelines, pub/sub, and cluster modes.                          |
| **Perl**      | `Redis`               | Pure Perl client for Redis, supporting standard commands and pub/sub messaging.                            |
| **Node.js**   | `ioredis`             | Feature-rich Node.js client supporting Redis Cluster, Sentinel, and Streams.                               |
| **Node.js**   | `node-redis`          | Official Redis client for Node.js, optimized for speed and efficiency.                                     |
| **Ruby**      | `redis-rb`            | Official Redis client for Ruby, supporting various Redis features.                                         |
| **Rust**      | `redis-rs`            | Rust client for Redis, designed for safety and performance.                                                |
| **PHP**       | `phpredis`            | Native PHP extension for Redis, providing fast and efficient communication.                                |

The choice of a client library depends on factors such as performance, feature support, and ease of integration with the application.

## Shell Scripting (Bash)

Shell scripting (Bash) does not have a dedicated Redis client library in the same way as other programming languages.

However, you can interact with Redis using `redis-cli`.

```bash
#!/bin/bash

# Set a key
redis-cli SET mykey "Hello, Redis!"

# Get a key
redis-cli GET mykey

# Delete a key
redis-cli DEL mykey
```

```bash
#!/bin/bash

redis-cli WATCH balance
balance=$(redis-cli GET balance)

if [ "$balance" -ge 100 ]; then
    redis-cli MULTI
    redis-cli DECRBY balance 100
    redis-cli EXEC
else
    redis-cli UNWATCH
    echo "Insufficient funds"
fi
```

Go over redis node.js examples

https://www.sitepoint.com/using-redis-node-js/
