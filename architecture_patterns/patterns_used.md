<!-- TOC -->
* [Architecture Patterns Used in the Marketplace System](#architecture-patterns-used-in-the-marketplace-system)
  * [1. Microservices Architecture](#1-microservices-architecture)
  * [2. Event-Driven Architecture (EDA)](#2-event-driven-architecture-eda-)
  * [3. CQRS (Command Query Responsibility Segregation) ️](#3-cqrs-command-query-responsibility-segregation-)
  * [4. Adapter Pattern](#4-adapter-pattern-)
  * [5. Pipeline Pattern](#5-pipeline-pattern-)
  * [6. Retry and Circuit Breaker Patterns](#6-retry-and-circuit-breaker-patterns-)
  * [7. Bulkhead Pattern](#7-bulkhead-pattern-)
  * [8. Idempotency Pattern](#8-idempotency-pattern-)
  * [10. Validation/Normalizer as Strategy Pattern](#10-validationnormalizer-as-strategy-pattern-)
  * [11. Outbox Pattern](#11-outbox-pattern-)
  * [12. Observer Pattern](#12-observer-pattern-)
<!-- TOC -->
# Architecture Patterns Used in the Marketplace System

## 1. Microservices Architecture
Decomposes the system into independently deployable services: Ingestion, Normalization, Validation, Repricer, Marketplace Dispatcher, etc.

Supports independent scaling, fault isolation, and flexibility in tech stacks.

## 2. Event-Driven Architecture (EDA) 
Core data flow is asynchronous via events on Kafka/SQS.

Decouples producers (ingestion) and consumers (processing services like Repricer,MarketPlace Dispatcher).

Enables high throughput, resilience, and event replay.

## 3. CQRS (Command Query Responsibility Segregation) ️
Although not fully explicit, the design separates write/update flows (ingestion/processing) from read-side consumers like Repricer and MarketPlace Updaters.

## 4. Adapter Pattern 
Used in the Data Ingestion Layer to abstract away vendor-specific protocols (CSV/FTP/API).
Also used in the Marketplace Dispatcher to handle different marketplace APIs (e.g., Amazon SP-API, eBay API).
Enables adding new vendors with minimal impact.

## 5. Pipeline Pattern 
Processing is done through a series of steps (Ingestion → Normalization → Validation → Repricing → Dispatch).

Each stage transforms or filters the data, promoting separation of concerns and testability.
Allows for the easy addition of new processing steps or modifications to existing ones.
Also supports parallel processing of events for scalability.

## 6. Retry and Circuit Breaker Patterns 
Used in error handling and downstream integration (marketplace APIs).
Retry with backoff for transient failures.
Circuit breaker (e.g., Resilience4j) prevents system overload from failing services.

## 7. Bulkhead Pattern 
Applies to vendor-specific queues or partitions to isolate failures.
Heavy-load vendors don't impact lighter ones. We can use separate Kafka topics or SQS queues for each vendor.
We can also have separate ingestion services for each vendor to isolate traffic and failures and improve throughput.


## 8. Idempotency Pattern 
Prevents duplicate processing in case of retries.

Crucial in marketplace updates, inventory changes, and repricing data.
We will maintain unique request keys (UUIDs) to ensure idempotent operations. We could internally generated UUIDs or use vendor-specific identifiers(vendor ID +SKU).

## 10. Validation/Normalizer as Strategy Pattern 
Vendor-specific normalization/validation logic can use **Strategy pattern** to plug in different implementations based on vendor ID.
We will also use this in the Marketplace Dispatcher to handle different marketplace APIs.

## 11. Outbox Pattern 
Used for reliable communication between internal services and external marketplaces.
We can replay events from the outbox in case of failures.
We can also track the status of each update event in the outbox.
Ensures exactly-once delivery and durability.

## 12. Observer Pattern 
Useful in DLQ/error tracking or audit logging.

Multiple systems (e.g., logging, monitoring) observe the same processing events.
The same event can trigger multiple actions (e.g., repricing, market place update , inventory update and so on).
We can also use Kafka Streams or similar libraries to implement this pattern for processing events in real-time.

