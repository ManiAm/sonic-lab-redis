## Redis Python Client

Redis provides an official Python client called `redis-py`, which allows interaction with a Redis server.

It communicates with Redis using a custom binary protocol over TCP called `RESP` (REdis Serialization Protocol).

It does not use REST, gRPC, WebSockets or other common protocols.

To install the Redis Python client:

    $ pip install redis

Connecting to a Redis server:

```python
import sys
import redis

# Create a connection to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

# Ping the Redis server to check if the connection is successful
if not r.ping():
    print("cannot connect to redis", file=sys.stderr)
    sys.exit(1)
```

Now you can store and retrieve key-value pairs:

```python
r.set('mykey', 'Alice')
mykey = r.get('mykey')
print(mykey)   # b'Alice'
```

Note that Redis returns values as bytes by default when using the redis-py library.

This is because Redis stores data as raw bytes, and Python’s redis library does not automatically decode it into a string.

To get the value as a string instead of bytes, you can decode the result using `.decode('utf-8')`:

```python
mykey = r.get('mykey').decode('utf-8')  # Decoding to string
print(mykey)                            # Output: Alice
```

Alternatively, you can configure the Redis client to always return strings instead of bytes.

This is done by setting `decode_responses=True` when initializing the Redis instance.

```python
# Create a connection to Redis
# and automatically decodes responses
r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)
```

This is the recommended approach if you always want string responses instead of handling bytes manually.

## Working with Different Data Types

**String**

```python
r.set('mykey', 'Alice')
mykey = r.get('mykey')
print(mykey) 

len = r.strlen('mykey')
print(len)

r.set('counter', 10)
counter = r.get('counter')
print(counter) 

r.incr('counter')     # Increment by 1
r.decr('counter', 2)  # Decrement by 2

# Working with floating points
r.set('temperature', 25.7)
r.incrbyfloat('temperature', 0.75)

# Note that in Redis, keys are always strings. This means that even if you use
# an integer, float, or any other data type as a key in your application, it
# must be converted to a string before it can be used as a key in Redis.

r.set(1, 'Alice')
mykey = r.get(1)
print(mykey) 
```

**List**

```python
r.lpush('list_key', 'Hello') # left push
r.rpush('list_key', 'World') # right push
```

**Set**

```python
r.sadd('set_key', 'Hello')
r.sadd('set_key', 'World')
r.sadd('set_key', 'World') # Duplicate values are not added
```

**Zset**

```python
r.zadd('zset_key', {'Hello': 1}) # score of 1
r.zadd('zset_key', {'World': 2}) # score of 2
```

**Hash**

```python
# Hashes
r.hset('hash_key', 'field1', 'Hello')
r.hset('hash_key', 'field2', 'World')

# Use HSET to set multiple fields
r.hset('hash_key_2', mapping={'field1': 'Hello', 'field2': 'World'})
```

**Stream**

```python
# Streams
r.xadd('stream_key', {'field1': 'Hello', 'field2': 'World'})
```

**HyperLogLog**

```python
r.pfadd('hyperloglog_key', 'Hello')
r.pfadd('hyperloglog_key', 'World')
r.pfadd('hyperloglog_key', 'World')  # Duplicate values are ignored
```

**Bitmap**

```python
r.setbit('bitmap_key', 5, 1)  # Set the bit at offset 5 to 1
```

**Geospacial**

```python
r.geoadd('geo_key', [(15.913, 45.815, 'Zagreb'), (14.508, 46.056, 'Ljubljana')])
```

## Key Expiration

```python
# Set a key 'another_key' with a value 'example'
r.set('another_key', 'example')

# Make 'another_key' expire in 30 seconds
r.expire('another_key', 30)

# Check how many seconds 'temp_key' has left before it expires
seconds_left = r.ttl('temp_key')
print(f"'temp_key' will expire in {seconds_left} seconds.")
```

## Key Search

## Publish-Subscribe

```python
# Publisher
def publisher():
    r.publish('channel1', 'Hello Subscribers!')

# Subscriber
def subscriber():
    pubsub = r.pubsub()
    pubsub.subscribe('channel1')

    for message in pubsub.listen():
        if message['type'] == 'message':
            print(f"Received: {message['data'].decode()}")
            break  # Stop listening after first message

# Run subscriber in another thread/process before calling publisher
```

## Redis Transaction

Transactions can be performed using pipelines.

```python
with r.pipeline() as pipe:
    pipe.set('foo', 'bar')
    pipe.incr('counter')
    pipe.execute()
```

```python
# pipeline

# Normally, when you issue a command to Redis, the process follows a request-response cycle
# for each command: the client sends a command, waits for the server to execute it and
# send back a response, and then sends the next command.

# Redis pipelines are a way to send multiple commands to the server in a single request,
# reducing the round-trip time (RTT) between the client and the server. This can
# significantly increase performance, especially when executing a large number of
# commands or when operating in a high-latency network environment. Pipelining
# batches the commands together, sends them to Redis in one go, and then reads
# all the responses in a single step.

# Create a pipeline
pipeline = r.pipeline()

# Queue some commands

pipeline.set('foo', 'bar')
pipeline.get('foo')

pipeline.incr('baz')
pipeline.incr('baz')
pipeline.get('baz')

# Execute the pipeline
responses = pipeline.execute()

# Print the responses
for response in responses:
    print(response)
```

```python
## transactions

import redis

# Connect to Redis
client = redis.Redis(host='localhost', port=6379, decode_responses=True)

# Start a transaction
with client.pipeline() as pipe:
    pipe.multi()  # Begin transaction
    pipe.set("key1", "Hello")
    pipe.set("key2", "World")
    pipe.incr("counter")
    result = pipe.execute()  # Execute transaction

print(result)  # Output: ['OK', 'OK', 1]
```

## Redis Watch

```python
# Suppose you have a key in Redis called 'balance' representing a user's account balance,
# and you want to update this balance in a safe manner, ensuring no other changes to
# balance occur while you're calculating the new balance.

# Key to watch
key = 'balance'

# Start by watching the key
r.watch(key)

# Imagine 'balance' is initially 100
current_balance = int(r.get(key)) if r.get(key) else 0

# Calculate new balance, simulating a transaction (e.g., adding 50)
new_balance = current_balance + 50

# Start a transaction
with r.pipeline() as pipe:

    while True:

        try:

            # Since we've already watched 'balance', if 'balance' changes
            # before this transaction executes, it will fail

            # Marks the start of the transaction block
            pipe.multi()

            pipe.set(key, new_balance)

            # Execute the transaction.
            # If any watched key (balance in this case) was modified after the
            # call to .watch() and before the call to .execute(), a WatchError is raised.
            pipe.execute()

            # If successful, break from the loop
            break

        except redis.WatchError:

            # If a WatchError is caught, it means the 'balance' changed
            # before the transaction could execute, so we retry the loop
            continue

        finally:

            # Regardless of success or failure, unwatch the key
            pipe.unwatch()

# Assuming no errors, 'balance' is now updated to 150, safely
print("Transaction completed. New balance:", r.get(key))

```

```python
# watch

import redis

client = redis.Redis(host='localhost', port=6379, decode_responses=True)

key = "balance"

def transfer_funds(amount):
    with client.pipeline() as pipe:
        while True:
            try:
                pipe.watch(key)  # Watch balance key
                balance = int(client.get(key) or 0)

                if balance < amount:
                    print("Insufficient funds")
                    pipe.unwatch()
                    return

                pipe.multi()  # Start transaction
                pipe.set(key, balance - amount)
                pipe.execute()  # Execute transaction
                print("Transaction successful")
                break
            except redis.WatchError:
                print("Transaction failed, retrying...")
                continue

# Initialize balance
client.set("balance", 100)
transfer_funds(30)  # Deduct 30
```
