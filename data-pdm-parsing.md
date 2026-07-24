# azfr-data-pdm-parsing

## Repository Overview

This document summarizes the **azfr-data-pdm-parsing** repository based
exclusively on the uploaded source.

## Purpose

`azfr-data-pdm-parsing` is a CDC (Change Data Capture) application that
consumes events from Kafka (Confluent), retrieves JSON schemas from the
Schema Registry, converts them into PyArrow schemas and stores the data
into Delta Lake.

It maintains four logical datasets:

-   Events
-   Live
-   Snapshot
-   History

The project also provides maintenance operations such as compaction,
vacuum and GDPR retention.

## Main Architecture

    Kafka
       │
       ▼
    Schema Registry
       │
       ▼
    Kafka Consumer
       │
       ▼
    Batching
       │
       ├── Events Delta Table
       ├── Live Delta Table
       ├── Snapshot
       └── History

## Main Modules

  Module          Responsibility
  --------------- -------------------------------
  parsing.py      Main application entry point
  consumer.py     Kafka consumer initialization
  batching.py     Buffering strategy
  converter.py    JSON Schema → PyArrow
  events.py       Events Delta table
  live.py         Live table merge/delete
  operations.py   Snapshot/history/maintenance
  schema.py       Schema Registry access
  helpers.py      Shared utilities

## Technologies

-   Python
-   Kafka / Confluent
-   Schema Registry
-   Delta Lake
-   PyArrow
-   Polars
-   Trino
-   Azure Data Lake Storage
-   Pydantic

## Processing Flow

1.  Load application configuration.
2.  Retrieve latest schema from Schema Registry.
3.  Build PyArrow schema.
4.  Initialize Kafka consumer.
5.  Read events.
6.  Validate JSON messages.
7.  Buffer according to strategy.
8.  Write Events Delta table.
9.  Merge into Live table.
10. Commit Kafka offsets.
11. Periodically create Snapshot and History.
12. Optimize Delta tables and apply retention.

## Repository Highlights

-   Buffered Kafka consumption
-   Graceful shutdown
-   Retry with exponential backoff
-   Delta merge operations
-   Automatic snapshot generation
-   Historical versioning
-   Delta optimization
-   GDPR cleanup
-   Dockerized deployment
-   GitHub Actions CI

## Source

This document was generated from the uploaded repository dump for
**azfr-data-pdm-parsing**.
