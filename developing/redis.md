# Contents

- [basics](#basics)
- [usecase](#usecase)

# basics
    `Redis` is in-memory data structure store, used as key-value database,  
    cache and message broker.  
    All data is kept in memory, but also is stored on disk but only to reconstruct data back on  
    system restart.  
    
# usecase
Can be used as [Celery](./celery.md) *backend* (result stores) and *broker* (message transport).

As *backend* it is suitable if results persistence is unnecessary.  

As *broker* it is suitable for small messages, for longer messages [RabbitMQ](./rabbitmq.md)  
is more suitable.
