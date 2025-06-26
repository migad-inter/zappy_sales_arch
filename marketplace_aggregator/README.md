# MarketPlace Aggregator Component Group
[Overall Architecture](./marketplace.txt)

This component group is responsible for using the UnifiedProduct DTO to aggregate data from multiple vendors and
updating the marketplace with the aggregated data.
</br> It will internally use the Repricer Service and the Inventory Service to update the marketplace.

### Key Design Principles:

- Reliable Updates: Ensure that updates to the marketplace are reliable and idempotent (no duplicates, no data loss).
- High Throughput: Handle large volumes of data efficiently. (10 Million+ SKUs,200k+ updates per day)
- Low Latency: Minimize the time taken to process updates and reflect them in the marketplace.
    - We should qualify this with an SLA, say 10 minutes for all updates to be reflected in the marketplace for a given
      vendor.
    - We could have Silver/Gold tiers for vendors, where Gold vendors have a lower latency SLA.
- Fault Tolerance: Handle failures gracefully without losing data or affecting the marketplace's availability.
- Scalability: Design the system to scale horizontally to handle an increased load.
- Extensibility: Allows for the easy addition of new marketplace platforms without significant changes to the codebase.

## **MarketPlace Aggregator Service**:

- Event based triggers from the Central Event Queue.(Kafka, RabbitMQ, SQS)
- Fetch current price from the Repricer Service.
- fetch current inventory from the Inventory Service.
- Use Asynchronous processing to update the marketplace.
- We can also use a Cache (like Redis) to store the current price and inventory for a given SKU to reduce the number of
  calls to the Repricer Service and Inventory Service.
- Bulk Fetch and Update:
    - Use bulk fetch and update APIs provided by the marketplace platforms to reduce the number of API calls. (not sure
      if this is possible for all marketplaces??)
    - This will also help in reducing the latency of updates to the marketplace.
- Use marketplace-specific adapters to handle the differences in API contracts and data formats.
    - Amazon SP-API, eBay API, etc.

### **Marketplace-Specific Dispatchers**:

- A set of services that will be used to dispatch the data to the marketplace platforms.
- Call each marketplace platform's API to update the price and inventory of the product asynchronously.
- we will need to handle throttling and rate limiting for each marketplace platform.

### **[Repricer and Pricing updates](./Repricer.md)**:
Refer to the [Repricer Service](./Repricer.md) for details on how pricing updates are handled.

### **Error Handling and Retries**:

- Use an outbox pattern where we store the events that need to be processed in a persistent store (could be database).
- This will allow us to retry the events in case of failures.
- We will use parallel workers / kafka consumer groups to process the events in parallel.
- These are horizontally scalable and can be scaled up or down based on the load.
- stateless workers.

### **Monitoring and Alerting**:
- capture status per update event 
- handle errors and retries
- give stats like last synced, last updated, number of updates processed, number of errors, etc. per sku ?
- we will need to prometheus,grafana for monitoring and alerting as well as ELK stack/Splunk for logging.

### **Tech Stack**:
| Layer                | Technology                                       |
| -------------------- |--------------------------------------------------|
| Event Queue          | Kafka, AWS SQS, Google Pub/Sub                   |
| Service Comm         | gRPC/REST (Repricer/Inventory), Spring WebClient |
| API Rate Control     | Resilience4j, Bucket4j                           |
| Persistence (Outbox) | PostgreSQL / DynamoDB / Redis Streams            |
| Marketplace APIs     | SDKs / REST clients                              |
| Retry/Backoff        | Kafka Retry Topics, Scheduled Executors          |
| Deployment           | Kubernetes / AWS ECS Fargate / GKE               |
