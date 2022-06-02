---
title: "Redis"
date: 2022-03-26T09:33:13+03:00
---
# Getting started
Redis is a distributed in-memory data structure store used as cache, database, and message broker.
## Installation
To install redis follow the [official instructions](https://redis.io/docs/getting-started/)  
Redis server and redis-cli tool will be installed
## Making sure redis-cli is installed
1. Open up your shell (terminal, console, wsl whatever).
2. Typing in `redis-cli --version` should show installed version of redis.

```
$ redis-cli --version
redis-cli 6.2.6
```
## Connecting to redis instance
Specify `-h` parameter to connect with needed host.
```
$ redis-cli -h redis.example.com
redis.example.com:6379>
```
If host parameter is omitted then redis will connect to localhost.
```
$ redis-cli
127.0.0.1:6379>
```
## Getting info about redis server
```
redis.example.com:6379> info
# Server
redis_version:6.2.6
...
```
# Switching databases
Redis has 16 databases which are numbered from 0 to 15.

To switch database in redis shell:
```
127.0.0.1:6379> select 5
OK
127.0.0.1:6379[5]>
```
To specify database when connecting to instance:
```
$ redis-cli -n 5
127.0.0.1:6379[5]>
```
# Keys, values and time to live (ttl)
To get all keys in current database:
```
127.0.0.1:6379> keys *
```
If we know that key should consist something, then we could try searching with pattern matcher:
```
127.0.0.1:6379> keys *duck*
1) "cache:ducks:europe"
...
```
If we know that it starts or ends with specific string then beginning or ending asterisk can be omitted.

To view key value we can use `get key:name`
```
127.0.0.1:6379> get cache:ducks:europe
"[{\"name\":\"Donald\"}]"
```
We were lucky that the key value was saved in a readable format. But sometimes values are encoded and might not be human-readable at all.

Some keys are self-expiring. After set time they will be deleted automatically. To view keys time to live (TTL) we could use `ttl` command:
```
127.0.0.1:6379> ttl cache:ducks:europe
(integer) 3503
```
Received value is in seconds. To get remaining minutes out of it, we could divide the number by 60. To get hours then we need to divide by 3600.  
Some keys are non-expiring and their ttl is returned as -1.
# Deleting keys
To delete a key we could use `del` command:
```
127.0.0.1:6379> del cache:ducks:europe
(integer) 1
```
We can also specify multiple keys:
```
127.0.0.1:6379> del cache:ducks:europe cache:ducks:australia cache:ducks:america
(integer) 2
```
Notice that it returns number of keys deleted (that existed and were deleted).
## Deleting multiple keys with key patterns
Sometimes it is annoying to specify all the keys one by one that need to be deleted. And redis does not support key patterns when using `del` command.  
Luckily we could use some bash functionality to get it done (make sure that You are not executing it from redis shell this time):
```
$ redis-cli keys cache:ducks:* | xargs redis-cli del
```
This will also return number of keys deleted.

In production instead of using keys make sure You use scan as scan is a non-blocking compared to keys.
```
$ redis-cli --scan --pattern cache:ducks:* | xargs redis-cli del
```
# Publishers and subscribers - streams
Listing all available channels:
```
127.0.0.1:6379> PUBSUB CHANNELS
 1) "stream:ducks:europe"
...
```
Subscribing to channel:
```
127.0.0.1:6379> subscribe stream:ducks:europe
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "stream:ducks:europe"
3) (integer) 1
1) "message"
2) "stream:ducks:europe"
3) "[{\"name\":\"Donald\"}]"