# Pravega Concepts

   * [Introduction](#introduction)
   * [Streams](#streams)
   * [Events](#events)
   * [Writers Readers Reader Groups](#writers-readers-reader-groups)
   * [Stream Segments](#stream-segments)
      - [Events in a Stream Segment](#events-in-a-stream-segment)
      - [Stream Segments and Connection Pooling](#stream-segments-and-connection-pooling)
      - [Elastic Streams Auto Scaling](#elastic-streams-auto-scaling)
      - [Events Stream Segments and Auto scaling](#events-stream-segments-and-auto-scaling)
      - [Stream Segments and Reader Groups](#stream-segments-and-reader-groups)
      - [Ordering Guarantees](#ordering-guarantees)
      - [Reader Group Checkpoints](#reader-group-checkpoints)
   * [Transactions](#transactions)
   * [State Synchronizers](#state-synchronizers)
   * [Architecture](#architecture)
   * [Putting the Concepts Together](#putting-the-concepts-together)
   * [A Note on Tiered Storage](#a-note-on-tiered-storage)
      - [Stream Retention Policies](#stream-retention-policies)


## Stream Segments and Connection pooling

When an Event is written to the Stream by the Pravega Client it is written into one of the Stream Segments based on the Event Routing Key. These Segments which are a part of Segment Containers are managed by the different [Segment Store](segment-store-service.md) Service instances.

At present the Pravega creates new connections to different Segment Stores for every Segment it is writing to. A new connection to a Segment Store is created even when multiple Segments are owned by the same Segment Store. Every Segment being read by the Pravega client maps to a new connection.
The number of connections created increases if the user is writing and reading from multiple Streams.
The goal of **connection pooling** is to ensure a common pool of connections between the client process and the Segment Stores, which does not require a linear growth of the number of connections with the number of Segments.
