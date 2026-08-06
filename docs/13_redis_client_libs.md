# Redis Client Libraries

Redis client libraries are language-specific packages that handle the communication between an application and a Redis server. They abstract the underlying RESP (REdis Serialization Protocol) wire format and provide idiomatic APIs for executing commands, managing data structures, and using features like pub/sub, pipelining, and transactions.

| **Language**  | **Client Library**    | **Description**                                                                                           |
|---------------|-----------------------|-----------------------------------------------------------------------------------------------------------|
| **C**         | `hiredis`             | Minimalist, high-performance C client library.                                                            |
| **C# (.NET)** | `StackExchange.Redis` | High-performance .NET client developed by the creators of Stack Overflow.                                 |
| **Go**        | `go-redis`            | Popular Go client with Cluster, Sentinel, and pipelining support.                                         |
| **Java**      | `Jedis`               | Lightweight Java client with straightforward connection management.                                       |
| **Java**      | `Lettuce`             | Reactive Java client with connection pooling, async I/O, and Cluster support.                             |
| **Node.js**   | `node-redis`          | Official Node.js client, optimized for speed and efficiency.                                              |
| **Node.js**   | `ioredis`             | Feature-rich Node.js client supporting Cluster, Sentinel, and Streams.                                    |
| **Perl**      | `Redis`               | Pure Perl client supporting standard commands and pub/sub.                                                |
| **PHP**       | `phpredis`            | Native PHP extension for fast, low-overhead Redis communication.                                          |
| **Python**    | `redis-py`            | Official Python client with pipeline, pub/sub, and Cluster support.                                       |
| **Ruby**      | `redis-rb`            | Official Ruby client covering the full Redis command set.                                                 |
| **Rust**      | `redis-rs`            | Rust client designed for memory safety and performance.                                                   |

The choice of library depends on factors such as performance requirements, feature support (Cluster, Sentinel, Streams), and ease of integration with the application stack.

## Shell Scripting (Bash)

Bash does not have a dedicated client library, but you can interact with Redis directly through `redis-cli`.

```bash
#!/bin/bash

redis-cli SET mykey "Hello, Redis!"
redis-cli GET mykey
redis-cli DEL mykey
```

A more advanced example using a here-document to send multiple commands over a single connection (similar to pipelining):

```bash
#!/bin/bash

redis-cli <<EOF
SET user:name "Alice"
SET user:age 30
EXPIRE user:name 3600
EXPIRE user:age 3600
EOF
```

**Important limitation:** Each `redis-cli` invocation opens a separate TCP connection, so `WATCH`/`MULTI`/`EXEC` transactions cannot span multiple invocations — a `WATCH` issued in one connection has no effect on a `MULTI` in another. For transactional logic, use a proper client library (see [Python Client](14_redis_client_python.md)) or Lua scripting (see [Lua Scripting](10_redis_lua.md)).
