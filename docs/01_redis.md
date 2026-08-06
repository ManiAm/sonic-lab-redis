# Redis

## Installing Redis

On Ubuntu/Debian:

    $ sudo apt update
    $ sudo apt install redis-server

Make sure the Redis service is up and running:

    $ sudo systemctl status redis

    ● redis-server.service - Advanced key-value store
        Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor preset: enabled)
        Active: active (running)
        Docs: http://redis.io/documentation,
                man:redis-server(1)

To stop Redis service:

    $ sudo systemctl stop redis

To start Redis service:

    $ sudo systemctl start redis

If you don't want to install Redis manually, you can use Docker:

    $ docker run --name redis-container -d -p 6380:6379 redis

## redis-cli

`redis-cli` is the command-line interface for interacting with a Redis server. It supports both interactive mode (entering commands manually) and non-interactive mode (executing commands via scripts). By default, it connects to `localhost:6379`, but you can specify a different host or port with the `-h` and `-p` flags.

Install it on your host with:

    $ sudo apt install redis-tools

To connect to the Docker container from the previous section, use either approach:

    $ redis-cli -p 6380                          # from your host (mapped port)
    $ docker exec -it redis-container redis-cli  # from inside the container

To verify connectivity (returns `PONG` if the server is reachable):

    $ redis-cli -p 6380 ping

To get Redis server info (version, OS, uptime, and more):

    $ redis-cli -p 6380 info server

When no command is given, redis-cli starts in interactive mode:

    $ redis-cli -p 6380
    127.0.0.1:6380>

To return a list of all available commands in Redis along with their arity (number of arguments), flags, and other details:

    > COMMAND

For example, one entry in the output might look like this:

    79) 1) "lpop"
        2) (integer) -2
        3) 1) write
           2) fast
        4) (integer) 1
        5) (integer) 1
        6) (integer) 1

This tells you that `LPOP` (which removes and returns the first element from a list) takes at least 2 arguments: the command name itself and one key. The negative arity indicates a variable number of arguments — `LPOP` also accepts an optional count (e.g., `LPOP mylist 3` to pop three elements). The flags `write` and `fast` indicate it modifies data and runs in O(1) time.

You don't need to memorize this format — it's mainly useful for debugging or exploring unfamiliar commands. For day-to-day use, `COMMAND DOCS <command>` or the [official Redis command reference](https://redis.io/docs/latest/commands/lpop/) are easier to read:

    > COMMAND DOCS LPOP

## Redis Databases

Redis supports multiple logical databases (16 by default, numbered 0 through 15). They do not have names — you refer to them only by their numeric index. Each database starts empty, and keys in one database do not interfere with keys in another. The number of databases can be changed via the `databases` setting in `redis.conf`.

You can check the number of configured databases using:

    $ redis-cli -p 6380 CONFIG GET databases

    1) "databases"
    2) "16"

To switch to a different database within an interactive session, use the `SELECT` command:

    > SELECT 2
    OK

The prompt changes to reflect the selected database (e.g., `127.0.0.1:6380[2]>`). This selection is per connection and does not persist across reconnections.

To check how many keys each database holds:

    > SELECT 0
    > DBSIZE
    (integer) 0

    > SELECT 1
    > DBSIZE
    (integer) 0

### Limitations of Redis Databases

Although databases provide namespace separation, they share significant resources:

- **Single-threaded:** All databases share the same event loop (see [Architecture](../README.md#how-redis-works-architecture)), so a slow command on database 0 blocks database 1.
- **Shared memory:** All databases use the same memory pool with no per-database limits.
- **Same configuration:** Persistence, eviction policies, and access controls apply uniformly.
- **No cluster support:** Redis Cluster mode only supports database 0.

For simple use cases, databases with `SELECT` are sufficient. For anything requiring performance isolation or independent configuration, use multiple Redis instances instead.

## Multiple Redis Instances

You can run multiple independent Redis server instances on the same machine, each on a different port with its own configuration file. Each instance gets its own dedicated memory, event loop, persistence settings, and failure boundary — providing true isolation.

### Why Use Multiple Instances?

- **Performance isolation:** High-traffic databases get their own instance and cannot starve latency-sensitive ones.
- **Independent configuration:** Each instance can have different persistence, eviction, and memory settings.
- **CPU utilization:** Redis is single-threaded, so multiple instances on a multi-core machine utilize more cores.
- **Failure isolation:** If one instance crashes or runs out of memory, the others remain unaffected.

### Setting Up Multiple Instances

Each instance needs a unique port. The approach depends on whether you are running Redis natively or in Docker.

**Native installation** — create a separate configuration file for each instance (e.g., `/etc/redis/redis-6381.conf`):

    port 6381
    pidfile /var/run/redis/redis-server-6381.pid
    logfile /var/log/redis/redis-server-6381.log
    dir /var/lib/redis-6381/
    dbfilename dump-6381.rdb

Start and connect to it:

    $ redis-server /etc/redis/redis-6381.conf
    $ redis-cli -p 6381

**Docker** — run a separate container for each instance, mapping each to a different host port:

    $ docker run --name redis-6380 -d -p 6380:6379 redis
    $ docker run --name redis-6381 -d -p 6381:6379 redis

Each container runs Redis on its default port (6379) internally, but is exposed on a unique host port. Connect to them with:

    $ redis-cli -p 6380
    $ redis-cli -p 6381

You can run as many instances as needed, each on a different port.

## Key Prefixes for Namespace Separation

Whether using single or multiple instances, a common best practice is to use key prefixes (e.g., `app1:user:123`, `cache:session:abc`) for namespace separation within a single database. This avoids key collisions and makes key scanning with patterns like `SCAN 0 MATCH app1:*` straightforward.
