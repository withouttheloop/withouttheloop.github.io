---
title: Ten Lessons From a Large IoT Project
author: Liam McLennan
date: 2015-09-02 20:32
template: article.jade
---

<img src="wc.jpg"/>

**Lesson 1**: Big data tools don't solve the intractable problems of distributed system. They leak complexity to the consumer.

**Lesson 2**: Peak performance often relies on even distribution across partitions. Hashing keys can help. Try SHA256.

**Lesson 3**: To prevent DOS attacks public cloud components like Event Hub throw exceptions if you exceed quotas. The client must take responsibility for throttling.

**Lesson 4**: [Azure web jobs are brilliant](https://azure.microsoft.com/en-us/documentation/articles/web-sites-create-web-jobs/). They provide a way to run a console application as a reliable service that can be scaled horizontally. Web jobs are simple and they work.

**Lesson 5**: [Distributed logs are a useful data structure for distributed systems](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying).

**Lesson 6**: To understand what is happening in a system you need centralised logging with a correlation scheme.  

**Lesson 7**: As metrics get larger we lose the ability to reason about them. 50/500/5000/50000 requests/s are very different. Those differences matter.

**Lesson 8**: Very large datasets (PB) drastically reduce data storage options.

**Lesson 9**: Consistency in the absence of transactions is hard. You will have to think about it.

**Lesson 10**: All actions must be idempotent. Work in batches. If an error occurs during a batch the idempotency allows the batch to be replayed.

**Lesson 11**: Tuning batch size substantially affects performance. Large batches and low error rates are the sweet spot. Higher error rates require smaller batches.
