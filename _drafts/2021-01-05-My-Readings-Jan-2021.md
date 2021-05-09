---
layout: post
title:  Summary of My Readings in January 2021
---

I read a lot of interesting articles on how tech companies are solving their problems. Most often i forget what i read and where i read. So, i have decided to write down my learings from those articles. This approach will help me in keep track of them and also helps other people through sharing. This post contains the summaries of my readings in January month of 2021.

## 1. Eliminating Task Processing Outages by Replacing RabbitMQ with Apache Kafka Without Downtime 
   This [article](https://doordash.engineering/2020/09/03/eliminating-task-processing-outages-with-kafka/) is originally published at DoorDash Engineering Blog.
- **Problems with RabbitMQ**
    - Availability is in question whenever there is huge load on RabbitMQ cluster
    - RabbitMQ does provide metrics (No of Consumers, Total Messages etc.) to track the health of the system but those are not sufficient for good observability
    - Performance is not upto the mark while scaling the system when the messages are durable

- **Reasons for Availability and Performance issues**
    - They observed sudden bursts of traffic becuase scheduled tasks are getting triggered at the same time at the end of their ETA. During this time task consumption is not upto the mark because of RabbitMQ [Flow Control](https://www.rabbitmq.com/flow-control.html) which reduces the speed connections which are publishing too quickly for queues to keep up. This sometimes leads to availability degradations but not always. When Flowcontrols kicks in, publishers see it as network latency. This will have a cascading effect on the upstream.
    - They also mentioned that they are facing availability issue because of [Connection Churn](https://www.rabbitmq.com/connections.html#high-connection-churn)

To resolve the availability issues, they decided to replace Celery and RabbitMQ with **Custom Kafka Solution** which has a lot of advantages over other solutions. 

Their strategy includes providing minimum viable product(MVP) to hit the ground running, reduce the challenges in developer adoption and incremental rollout with zero downtime.

- **Problems they faced after switching to Kafka**
    - **Head of line blocking**
        - This happens when processing of the message in the front of the partition is taking long, other messages has to wait which will cause unnecessary delays.
        - This can be resolved by using load balancing in single process. We use [this](https://github.com/JonCSykes/tightrope) GoLang library to avoid head of line blocking. 
        - Confluent also provides a [solution](https://www.confluent.io/blog/introducing-confluent-parallel-message-processing-client/) to solve this issues
    - **Delays during rebalancing**
        - Partition rebalance happens everytime you deploy your application
        - This will cause momentary slowdown in message processing as task consumption had to stop during rebalancing
        - This can be resolved by using [Incremental Co-operative Rebalancing](https://www.confluent.io/blog/incremental-cooperative-rebalancing-in-kafka/) but this is not yet supported by all client libraries.

They observed significant improvements in availability, performance and observability even though there were few issues after switching to Kafka from RabbitMQ.