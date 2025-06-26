# **Data Ingestion Group**:

A batch application that allows users to interact with the system.


## Modular Microservices-based Ingestion Pipeline:

A set of microservices that will be used to process the data ingested by the application.

### **Vendor Specific Ingestion Services**:

- A set of services that process the data ingested.
- So we have a set of vendor-specific ingestion services that will be used to process the data ingested by the
  overall application. These services will be responsible for processing the data and storing it in the database
  for that vendor.

### **Transport-Specific Adapters**:

A set of adapters that will be used to transport the data ingested by our application from the vendor-specific ingestion
services.

- FTP Ingestor
- Email/CSV Ingestor
- API Poller or Webhook Ingestor

#### **Notes**:

- Use Spring Boot or Quarkus to build the microservices.
- Use ScheduledJobs / Event Triggers (Quartz, Cloud Scheduler)
-

### **Normalization and Enrichment Services**:

A set of services that will be used to normalize and enrich the
data ingested by the application.

- Converts Vendor-Specific Data to a Common Format (like a UnifiedProduct DTO). Incoming formats can be
  CSV, JSON, XML, etc. But our application should deal with a canonical format.
- Validates and enriches the data, like checking for missing fields, adding default values, business
  rules.
-

### **Central Event Queue**:

Push Validated and Enriched Data to a Central Event Queue (like Kafka, RabbitMQ, Amazon SQS.) for further processing.

- This allows for decoupling of the ingestion pipeline from the processing pipeline.
- Use Partitions in Kafka by SKU category/ Vendor ID to allow consumers to scale horizontally.

#### **Idempotent Event Processors**:

Push Validated and Enriched Data to a Central Event Queue (like Kafka,
RabbitMQ, Amazon SQS.) for further processing.

- Update staging inventory in the database.
- Trigger Repricer Service to update the price of the product.

#### Notes

- Idempotency with unique request Keys (UUID,internally generated)
- Retry Mechanism for failed events by using Dead Letter Queues on Kafka or SQS.

#### **Tech Stack**:

- Ingestion Services: Spring Boot or Quarkus, AWS Lambda, or Azure Functions.
- File Ingestion: Apache Camel or custom file ingestion service.
- CSV Parsing: Apache Commons CSV or OpenCSV.
- Queue: Apache Kafka, RabbitMQ, or Amazon SQS.
- Monitoring: Prometheus, Grafana, or ELK Stack.

## Main Design Considerations

### **Scalability**:

- Stateless ingestion services that can be scaled horizontally in autoscaling groups.

### **Modularity**:

- Plug and play new vendor support via Apache Camel configuration.

### **Resilience**:

- Circuit Breaker(Resilience4J), retries, kubernetes fallbacks.

### **Handling Uneven Vendor Transaction Volume**:

- We can use Vendor-Specific Ingestion Services and Queues / Kafka topics to handle uneven transaction volume.
- Allows fine-grained control over thru-put and prioritization.
- We can also shard and parellize by vendor with vendorID = shard key.
- if using Kafka, we can use partitions and consumer groups to handle uneven transaction volume.
