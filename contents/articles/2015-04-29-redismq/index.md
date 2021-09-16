---
title: RedisMQ
author: Liam McLennan
date: 2015-04-29 20:32
template: article.jade
---

Many people use Redis for a simple message broker. [RedisMQ](https://gist.github.com/liammclennan/5adf41090507ad6b0b0c) is a trivial layer on [Redis](http://redis.io/) and [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) that probably does not work. It provides two simple messaging patterns:

Publish / subscribe
---

A publisher publishes messages. 0, 1 or more consumers subscribe to the messages.

    var redis = ConnectionMultiplexer.Connect("192.168.85.128");
    redis.BroadcastSubscribe<string>("this-is-the-channel", message =>
        // do something with the message);
    redis.BroadcastPublish("this-is-the-channel", "42");

Competing consumer
---

A publisher publishes messages to a queue. Messages stay in the queue until someone reads them. Many subscribers may read from the same queue. Each message is processed by just one consumer.

    var redis = ConnectionMultiplexer.Connect("192.168.85.128");
    redis.CompetingConsumerSubscribe<int>("numbers", message =>
        // do something with the message);
    redis.CompetingConsumerPublish("numbers", 1);

Speed
---

RedisMQ is not optimised for performance. It is optimized for being simple for me to write. However, the following gives a rough indication of performance.

Strategy | # messages | Time (s)
-------- | ---------- | -------
Broadcast to 2 consumers | 100,000 | 21.3
Competing consumer with 1 subscriber | 100,000 | 37.7
Competing consumer with 2 subscribers | 100,000 | 23.7

Dependencies
----

    PM> Install-Package StackExchange.Redis
    PM> Install-Package Newtonsoft.Json
