# Architecture Artifacts for Zappy Sales

A Repo to store Zappy Sales Architecture Artifacts

### Summary of the Patterns and architecture principles being Used

Check [here](architecture_patterns/patterns_used.md) for the patterns and architecture principles being used in the
Zappy Sales Architecture.

[Overall Architecture Diagram](overall_arch_with_sharding.txt)

### Main Constraints

- The architecture should be modular and scalable.
- We should be able to add new features without breaking existing functionality.
- We should be able to deploy new features without downtime.
- The architecture should be able to handle a large amount of data. (10 Million+ SKUs)
- A new vendor can be added without breaking existing functionality and without downtime and only requires a
  configuration change.
- It should be possible to add new vendors without changing the codebase.
- It should be possible to add new features without changing the codebase. By plugging in new microservices or
  components.
- It should be possible to trace the flow of every vendor update through the system.
-

### Main Assumptions

- There are no online users or concurrent users using a web application at the moment.
- The architecture is designed to handle batch processing of data.
- The architecture is designed to handle a large amount of data.
- The architecture is designed to handle a large number of SKUs (10 Million+).
- The architecture is designed to handle a large number of vendors (100+).
-

### Main Component Areas.

So we want to capture a cluster of components that will be used to build the Zappy Sales Architecture. The components
are grouped into the following main areas:

- **[Data Ingestion Component Group](data_ingestion_group/DataIngestion.md)**
- **[MarketPlace Aggregator](marketplace_aggregator/README.md)** 

### [Database Schemas](architecture_patterns/database_schemas.md)

