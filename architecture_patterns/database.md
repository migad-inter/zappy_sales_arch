# Database Architecture Patterns

Every service in the Zappy Sales architecture will have its own database to ensure modularity, scalability, and fault isolation.<br>
This allows each service to evolve independently and scale based on its specific needs.

- **[Inventory Database](inventory_database.md)**:
- **[MarketPlace Update DB](market_place_update_db.md)**:
- **[Repricer DB](repricer_db.md)**:
- **[Data Ingestion DB](data_ingestion_db.md)**:

## **Data Flow and Storage Patterns**

| Source              | Event                  | Destination         |
|---------------------|------------------------|---------------------|
| Data Ingestion      | NormalizedProductEvent | Inventory, Repricer |
| Repricer            | PriceComputedEvent     | Marketplace Updates |
| Inventory           | StockUpdatedEvent      | Marketplace Updates |
| Marketplace Updates | UpdateSentEvent        | Audit / Monitoring  |

## DB Technology Choices

| Service             | Recommended DB             |
|---------------------|----------------------------|
| Data Ingestion      | PostgreSQL + JSONB         |
| Inventory           | PostgreSQL / CockroachDB   |
| Repricer            | PostgreSQL + Redis (cache) |
| Marketplace Updates | PostgreSQL                 |

### **PostgreSQL**

- chosen for its robustness, support for JSONB, and strong community support.
- Excellent for complex queries and data integrity,transactions.

### **JSONB**

- ingested data can be stored in JSONB format for flexibility, as incoming data formats can vary. (CSV, JSON, XML, etc.)
- store normalized data in a structured format while allowing for schema evolution.
- query nested JSON structures efficiently.
- index JSONB fields for performance.
- 
### **CockroachDB** (alternative to PostgreSQL)

- distributed SQL database that provides horizontal scalability and strong consistency.
- excellent for high availability and fault tolerance, scaling out as needed.
- can query postgres-compatible SQL, making it easier to migrate from PostgreSQL if needed.

### **Redis**
- used as a caching layer for the Repricer service to store frequently accessed data like current prices and inventory.
- low latency lookups and high throughput for read-heavy workloads.

| Service             | Data Nature         | DB Chosen                | Reason                                                      |
|---------------------|---------------------|--------------------------|-------------------------------------------------------------|
| Data Ingestion      | Semi-structured     | PostgreSQL + JSONB       | Flexibility for vendor formats, transactional logging       |
| Inventory           | High concurrency    | PostgreSQL / CockroachDB | Strong consistency, scalable writes                         |
| Repricer            | Low-latency pricing | PostgreSQL + Redis       | Rules in SQL, prices cached in Redis for speed              |
| Marketplace Updates | Event tracking      | PostgreSQL               | Durable outbox, status tracking, retries, full traceability |
