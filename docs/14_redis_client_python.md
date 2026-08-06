# Redis Python Client

Redis provides an official Python client called `redis-py` for full interaction with a Redis server. It communicates over TCP using RESP (REdis Serialization Protocol), a custom binary protocol — not REST, gRPC, or WebSockets.

## Installation

    $ pip install redis

## Connecting to Redis

```python
import sys
import redis

r = redis.Redis(host='localhost', port=6380, db=0)

if not r.ping():
    print("cannot connect to redis", file=sys.stderr)
    sys.exit(1)
```

### Response Encoding

By default, `redis-py` returns values as raw bytes because Redis itself stores data as bytes:

```python
r.set('name', 'Alice')
value = r.get('name')
print(value)  # b'Alice'
```

You can decode individual values manually:

```python
value = r.get('name').decode('utf-8')
print(value)  # Alice
```

Or configure the client to always return decoded strings by setting `decode_responses=True`:

```python
r = redis.Redis(host='localhost', port=6380, db=0, decode_responses=True)
```

This is the recommended approach if you always work with string data.

## Working with Data Types

### String

```python
r.set('greeting', 'Hello World')
print(r.get('greeting'))
print(r.strlen('greeting'))  # 11

r.set('counter', 10)
r.incr('counter')
r.decr('counter', 2)

r.set('temperature', 25.7)
r.incrbyfloat('temperature', 0.75)
```

In Redis, keys are always strings. Even if you pass an integer as a key in Python, it is converted to a string before being stored:

```python
r.set(1, 'Alice')
print(r.get(1))  # same as r.get('1')
```

### List

```python
r.lpush('list_key', 'Hello')
r.rpush('list_key', 'World')
print(r.lrange('list_key', 0, -1))
```

### Set

```python
r.sadd('set_key', 'Hello')
r.sadd('set_key', 'World')
r.sadd('set_key', 'World')  # duplicate, ignored
print(r.smembers('set_key'))
```

### Sorted Set

```python
r.zadd('zset_key', {'Hello': 1, 'World': 2})
print(r.zrange('zset_key', 0, -1, withscores=True))
```

### Hash

```python
r.hset('hash_key', 'field1', 'Hello')
r.hset('hash_key', 'field2', 'World')

r.hset('hash_key_2', mapping={'field1': 'Hello', 'field2': 'World'})

print(r.hgetall('hash_key'))
```

### Stream

```python
r.xadd('stream_key', {'field1': 'Hello', 'field2': 'World'})
print(r.xrange('stream_key', '-', '+'))
```

### HyperLogLog

```python
r.pfadd('hll_key', 'Hello')
r.pfadd('hll_key', 'World')
r.pfadd('hll_key', 'World')  # duplicate, ignored
print(r.pfcount('hll_key'))   # 2
```

### Bitmap

```python
r.setbit('bitmap_key', 5, 1)
print(r.getbit('bitmap_key', 5))  # 1
```

### Geospatial

```python
r.geoadd('geo_key', [15.913, 45.815, 'Zagreb', 14.508, 46.056, 'Ljubljana'])
print(r.geodist('geo_key', 'Zagreb', 'Ljubljana', unit='km'))
```

## Key Expiration

```python
r.set('temp_key', 'example')
r.expire('temp_key', 30)

seconds_left = r.ttl('temp_key')
print(f"'temp_key' will expire in {seconds_left} seconds.")
```

## Key Search

Use `scan_iter()` to iterate over keys matching a pattern. This is the Python equivalent of the `SCAN` command and is safe for production use, unlike `keys()` (equivalent of `KEYS`) which blocks the server while scanning the entire keyspace:

```python
for key in r.scan_iter(match='user:*', count=100):
    print(key)
```

To iterate over all keys (use with caution on large databases):

```python
for key in r.scan_iter():
    print(key)
```

## Publish/Subscribe

```python
import redis
import threading
import time

r = redis.Redis(host='localhost', port=6380, decode_responses=True)

def subscriber():
    pubsub = r.pubsub()
    pubsub.subscribe('channel1')

    for message in pubsub.listen():
        if message['type'] == 'message':
            print(f"Received: {message['data']}")
            break

def publisher():
    r.publish('channel1', 'Hello Subscribers!')

thread = threading.Thread(target=subscriber)
thread.start()

time.sleep(0.5)  # give the subscriber time to connect
publisher()
thread.join()
```

## Pipelines

Normally, each command sent to Redis requires a full network round-trip: the client sends the command, waits for the response, then sends the next. Pipelines batch multiple commands and send them in a single request, then read all responses at once. This reduces round-trips and can significantly improve throughput.

```python
pipeline = r.pipeline()

pipeline.set('foo', 'bar')
pipeline.get('foo')
pipeline.incr('baz')
pipeline.incr('baz')
pipeline.get('baz')

responses = pipeline.execute()
for response in responses:
    print(response)
```

## Transactions

In `redis-py`, transactions are built on top of pipelines. Calling `multi()` on a pipeline wraps the queued commands in a `MULTI`/`EXEC` block, ensuring they execute atomically — either all commands run or none do.

```python
import redis

client = redis.Redis(host='localhost', port=6380, decode_responses=True)

with client.pipeline() as pipe:
    pipe.multi()
    pipe.set("key1", "Hello")
    pipe.set("key2", "World")
    pipe.incr("counter")
    result = pipe.execute()

print(result)  # ['OK', 'OK', 1]
```

## Optimistic Locking with WATCH

`WATCH` enables optimistic locking for check-and-set operations. After watching a key, if any other client modifies it before the transaction executes, `redis-py` raises a `WatchError`, allowing you to retry:

```python
import redis

client = redis.Redis(host='localhost', port=6380, decode_responses=True)

key = "balance"
client.set(key, 100)

def transfer_funds(amount):
    with client.pipeline() as pipe:
        while True:
            try:
                pipe.watch(key)
                balance = int(client.get(key) or 0)

                if balance < amount:
                    print("Insufficient funds")
                    pipe.unwatch()
                    return

                pipe.multi()
                pipe.set(key, balance - amount)
                pipe.execute()
                print(f"Transaction successful. New balance: {client.get(key)}")
                break
            except redis.WatchError:
                print("Transaction failed due to concurrent modification, retrying...")
                continue

transfer_funds(30)
```
