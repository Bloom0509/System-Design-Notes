# System Design Notes

A practical, revision-friendly system design notebook with explanations, examples, trade-offs, and interview reminders.

## What is system design?
System design is the process of defining the architecture, components, modules and data flow of a software system so that it can handle users needs effectively, especially when it needs to scale.


## How to use this repo
- Start with [System Design Revision Notes](./SYSTEM_DESIGN_NOTES.md).
- Focus on the problem each component solves—not just definitions.
- Use the end-to-end architecture section to practise explaining how the pieces work together.
- Revisit the interview checklist before a system design round.

## Topics
- DNS, HTTP/HTTPS, and request lifecycle
- Vertical/horizontal scaling and load balancing (L4/L7)
- CDN and Redis caching strategies
- Database indexes, replication, sharding, and consistent hashing
- Message queues, event-driven design, Kafka vs RabbitMQ
- Object/blob storage and pre-signed URLs
- Rate limiting, polling, WebSockets, and idempotency
- CAP theorem and an end-to-end scalable architecture

## Core interview habit
For every design choice, explain **the bottleneck → the component → the benefit → the trade-off**.

These notes are a learning aid. Real systems differ by workload, consistency requirements, reliability goals, and operational constraints.
