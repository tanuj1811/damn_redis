# damn_redis

## Introduction

Redis, which stands for Remote Dictionary Server. It is widely used as a database, cache, and message broker.

### Key Features
 
  - It's an **open-source,** library.
  - It is fast because of it's architecture.
  - Redis is single threaded but it can handle thread concurrently using redis config file or use command `redis-server --io-threads 4` (**concurrently v/s Parallelism in FYI**) .
  - It's uses **in-memory data store (RAM)** but also provide to store **Persistence** storage option to save data when server reloaded.
    - It uses **RDB (Redis Database) snapshots and AOF (Append Only File) log** to save on disk. (Read FYI below, for RDB and AOF).
  - **Redis supports a variety of data structures**, including strings, lists, sets, hashes and sorted set.
  - **Create multiple clusters** and **supports horizontal scaling** for large database. (Why??)
  - **Redis Cluster allows automatic sharding** of data across multiple nodes
  - It also support **Pub/Sub (Publishers/Subscribers)** feature. It allows **Decoupled Communication between multiple Redis clients**
    - **Publishers** send messages to channels without knowing who the subscribers are.
    - **Subscribers** receive messages from channels they are subscribed to, without knowing who the publishers are.
    - **More detail about pub/sub in FYI section** 

## Operations Keys

- EXPIRE key seconds: Sets a timeout on a key.
- TTL key: Gets the remaining time to live (TTL) of a key.



## LUA Script
Lua Script allows you to run custom scripts directly on the Redis server.

- It is executed atomically. This means that while a script is running, no other commands can be executed, ensuring that all operations within the script are completed without interference.
- Running LUA scripts directly on the server reduces the number of round trips between the application and Redis, improving performance for multi-step operations.
- It can call any Redis command using redis.call() or redis.pcall() (**Description in Error handling section**), allowing for flexible data manipulation.

### Use case
1. **Conditional Operations**
   
   ```lua
    EVAL "if redis.call('GET', KEYS[1]) == ARGV[1] then return redis.call('SET', KEYS[1], ARGV[2]) else return 0 end" 1 mykey oldvalue newvalue
   ```
    Above script is simple if else statement with Arguments (1 mykey oldvalue newvalue) 
   
     - '1': This indicates that there is one key being passed to the script.
     - 'mykey': This is the key being checked and potentially updated.
     - 'oldvalue': This is the expected current value of the key.
     - 'newvalue': This is the new value to set if the current value matches oldvalue.

2. **Rate Limiting**
   
   ```lua
    EVAL "local count = redis.call('INCR', KEYS[1]) if count == 1 then redis.call('EXPIRE', KEYS[1], ARGV[1]) end return count" 1 user:123:requests 60
   ```
   - '1': This indicates that there is one key being passed to the script.
   - user:123:requests: This is the key being incremented and potentially set with an expiration time.
   - 60: This is the expiration time (TTL) in seconds to set if the incremented value is 1.
  
3. **Complex Aggregations** 
   ```lua
    EVAL "return redis.call('GET', KEYS[1]) + redis.call('GET', KEYS[2])" 2 key1 key2
   ```
4. **Custom Command Creation**

## Error Handling

> Commonly used in Eval Script

- Use **redis.call()** for normal command execution. Errors will be returned directly to the client.
- Use **redis.pcall()** if you want to handle errors gracefully within the script context without stopping execution.

## Operation Commands

### Basic Commands

| **Command** | **Description**                    | **Usage Example**                                |
|-------------|------------------------------------|-------------------------------------------------|
| SET         | Sets the value of a key            | `SET mykey "Hello, World!"`                     |
| GET         | Gets the value of a key            | `GET mykey`                                     |
| DEL         | Deletes a key                      | `DEL mykey`                                     |
| EXPIRE      | Sets a timeout on a key            | `EXPIRE mykey 10`                               |

### Other Commands


### Transactions Commands

| **Command** | **Description**                                      | **Usage Example**                                |
|-------------|------------------------------------------------------|-------------------------------------------------|
| MULTI       | Marks the start of a transaction block               | `MULTI`                                         |
| EXEC        | Executes all commands issued after MULTI             | `SET key1 "val1"`<br>`SET key2 "val2"`<br>`EXEC`|
| DISCARD     | Discards all commands issued after MULTI             | `MULTI`<br>`SET key1 "val1"`<br>`DISCARD`       |

### Pub/Sub Commands

| **Command**   | **Description**              | **Usage Example**                                |
|---------------|------------------------------|-------------------------------------------------|
| PUBLISH       | Sends a message to a channel | `PUBLISH mychannel "Hello, subscribers!"`       |
| SUBSCRIBE     | Subscribes to a channel      | `SUBSCRIBE mychannel`                           |
| UNSUBSCRIBE   | Unsubscribes from a channel  | `UNSUBSCRIBE mychannel`                         |

### LUA Script Commands

| **Command** | **Description**                                         | **Usage Example**                                           |
|-------------|---------------------------------------------------------|------------------------------------------------------------|
| EVAL        | Evaluates a Lua script                                  | `EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 mykey "value"` |
| EVALSHA     | Evaluates a Lua script using the SHA1 hash of the script| `EVALSHA <sha1-hash> 1 mykey "value"`                       |

### Hashes Commands

| **Command** | **Description**                        | **Usage Example**                                |
|-------------|----------------------------------------|-------------------------------------------------|
| HSET        | Sets the value of a field in a hash    | `HSET myhash field1 "value1"`                   |
| HGET        | Gets the value of a field in a hash    | `HGET myhash field1`                            |
| HDEL        | Deletes a field in a hash              | `HDEL myhash field1`                            |

### Lists Commands

| **Command** | **Description**                                  | **Usage Example**                                |
|-------------|--------------------------------------------------|-------------------------------------------------|
| LPUSH       | Pushes a value onto the head of a list           | `LPUSH mylist "value1"`                         |
| RPUSH       | Pushes a value onto the tail of a list           | `RPUSH mylist "value2"`                         |
| LPOP        | Pops a value from the head of a list             | `LPOP mylist`                                   |
| RPOP        | Pops a value from the tail of a list             | `RPOP mylist`                                   |

### Sets Commands

| **Command** | **Description**                                  | **Usage Example**                                |
|-------------|--------------------------------------------------|-------------------------------------------------|
| SADD        | Adds a member to a set                           | `SADD myset "member1"`                          |
| SREM        | Removes a member from a set                      | `SREM myset "member1"`                          |
| SMEMBERS    | Gets all the members in a set                    | `SMEMBERS myset`                                |

### Sorted Sets Commands

| **Command** | **Description**                                                                | **Usage Example**                                |
|-------------|--------------------------------------------------------------------------------|-------------------------------------------------|
| ZADD        | Adds a member to a sorted set, or updates its score if it already exists        | `ZADD myzset 1 "member1"`                       |
| ZRANGE      | Returns a range of members in a sorted set, by index                           | `ZRANGE myzset 0 -1`                            |
| ZREM        | Removes a member from a sorted set                                             | `ZREM myzset "member1"`                         |

### Utility Commands

| **Command** | **Description**                                               | **Usage Example**                                |
|-------------|---------------------------------------------------------------|-------------------------------------------------|
| INFO        | Provides information and statistics about the server          | `INFO`                                          |
| PING        | Checks if the server is running                               | `PING`                                          |
| FLUSHDB     | Deletes all keys in the current database                      | `FLUSHDB`                                       |
| FLUSHALL    | Deletes all keys in all databases                             | `FLUSHALL`                                      |

## Competitors
Memcached

Apache Ignite

Hazelcast

Couchbase

## For Your Information (FYI)

### RDB (Redis Database) Snapshots:
- It's a one of the method to store data persistly in redis. This method **creates the binary file** (commonly named dump.rdb) that contains the state of the database at specific intervals.
- It's save the data on **interval bases or the number of changes made**. Example, might set it to save every 60 seconds if at least 1,000 keys have changed.

### AOF (Append Only File):
- It's logs every write operation received by Redis **in a sequential manner** in **plain text file**.
- Each time Redis executes a command that alters data (like SET or DEL), it appends that command to the AOF file.
- **Slower than RDB but safer than RDB**


### Pub/Sub (Publishers/Subscribers)

Pub/Sub (short for publish/subscribe) is a messaging technology that facilitates communication between different components in a distributed system.

It's provides a simple and efficient messaging system between clients. In Redis, clients can “publish” messages to a named channel, and other clients can “subscribe” to that channel to receive the messages.

#### Key Points
- It  is **synchronous**. Subscribers and publishers must be **connected at the same time** in order for the message to be delivered.
- It uses **Fire & Forget messaging pattern** where the sender **sends a message without expecting an explicit acknowledgment** from the receiver that the message was received. The sender simply sends the message and moves on to the next task, regardless of whether or not the message was actually received by the receiver.
- Similarly, **Pub/Sub Redis is fan-out** only, meaning that when a publisher sends a message, it is broadcast to all active subscribers. All subscribers **receive a copy of the message**, **regardless of whether they are specifically interested in the message or not**.
  
#### Communication Model
They are usually classified under **four models** based on the number of publishers and subscribers involved in the communication, which include **one-to-one, one-to-many, many-to-one, and many-to-many**.

![modals definition](./images//pub_sub_modal_desc.png)

#### Under the Hood (Redis) 

> Reference [[Medium Redis Pub/Sub in depth](https://medium.com/@joudwawad/redis-pub-sub-in-depth-d2c6f4334826#7b45)]

#### Use Cases
- Refreshing distributed caches
- Real-time notifications
- Sending messages between microservices
- Communicating between different parts of a single application
- Event-driven architectures Applications
- News updates and alerts


## Famous Questions

1. List the main operation keys of Redis in detail.
2. How can the durability of Redis be enhanced?
3. How can the Redis be used with the any application? Give example in sanic!
4. How can a Redis database be emptied?
5. Mention and describe some commands of Redis
6. Explain REPL and Redis-CLI.
7. What is meant by hashes in Redis?
8. What is meant by data modeling in Redis?
9. How can a Redis database be moved from one server to another server?
10. How can the array data be obtained from Redis?
11. What is meant by "Redis is Binary Safe"?
12. Which things should be kept in mind while using Redis?
13. What is meant by SUNION, SINTER, SUNIONSTORE and SINTERSTORE?
