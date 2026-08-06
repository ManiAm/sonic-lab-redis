## Publish/Subscribe (Pub/Sub)

Redis Publish/Subscribe allows real-time messaging between applications.

A publisher sends messages to a channel, and subscribers listening on that channel receive them `instantly`.

Pub/Sub Commands:

| Command                | Description                                     | Example                               |
|------------------------|-------------------------------------------------|---------------------------------------|
| `SUBSCRIBE channel`    | Subscribe to a channel                          | `SUBSCRIBE notifications`             |
| `PUBLISH channel msg`  | Publish a message to a channel                  | `PUBLISH notifications "Hello!"`      |
| `PSUBSCRIBE pattern`   | Subscribe to channels matching a pattern        | `PSUBSCRIBE notif*`                   |
| `PUNSUBSCRIBE pattern` | Unsubscribe from channels matching a pattern    | `PUNSUBSCRIBE notif*`                 |
| `UNSUBSCRIBE channel`  | Unsubscribe from a channel                      | `UNSUBSCRIBE notifications`           |
| `PUBSUB subcommand`    | Inspect the state of the Pub/Sub system         | `PUBSUB CHANNELS`                     |

Open three separate terminal windows and execute the `redis-cli` command in each to initiate three independent Redis CLI sessions.

Invoke the following command in two terminals:

    > SUBSCRIBE news

The client enters subscription mode, and it will only receive messages published to "news" channel.

This is a blocking call. The client cannot execute other commands until it unsubscribes.

In the third terminal publish a message to the "news" channel:

    > PUBLISH news "Breaking News: Redis 7.0 Released!"

Other two subscribers will receive the message.

Redis Pub/Sub does not provide buffering or message persistence.

If a publisher sends a message to a channel without any active subscribers, the message is lost.

Messages are only delivered to active subscribers at the moment of publishing.

Redis Pub/Sub works like a broadcast system:

- The publisher sends a message.
- Only active subscribers receive it.
- No backlog or history is stored.

It is useful for event notifications, real-time feeds, and inter-service communication.

If you need buffering, persistence, or message history, use Redis `Streams` instead.

Streams store messages and allow consumers to retrieve them later.
