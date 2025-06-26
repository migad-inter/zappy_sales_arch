# **Reprice Service**:
Here is a [flowchart](repricer.txt) of the Repricer Service:
### Important Constraints: 
- The Repricer Service should be able to handle a large number of requests per second.
- Upto 200k+ transactions per day.
- The Repricer Service should be able to handle a large number of SKUs (10 Million+).
- Pricing updates should be idempotent to avoid duplicate updates/incorrect pricing.
- Repricing and marketplace updates should be decoupled to allow for independent scaling.

### **Proposed Architecture**:
- Use SKU+timestamp as a unique key for idempotency. or use a UUID as a unique key for idempotency.
- So retries can be handled by checking if the SKU+timestamp or UUID already exists in the database.
- Use LRU cache , Redis or similar caching mechanism to store the current price and inventory for a given SKU.
- There should be a configurable business rules engine inside this service to handle different repricing strategies or potential changes.
- Use a message queue (like Kafka, RabbitMQ, or SQS) to handle the incoming repricing requests asynchronously.
- This decouples the repricing service from the marketplace update service, allowing for independent scaling.