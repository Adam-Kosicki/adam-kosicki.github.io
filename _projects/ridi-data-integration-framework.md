---
layout: project
title: "RIDI: Real-Time Data Integration Framework"
role: "Engineering Scientist | TxDOT / UT Austin"
order: 3
hero_metric: "API of APIs for Real-Time Telemetry"
tech_stack: ["FastAPI", "PostgreSQL", "SQLAlchemy", "Redis", "Celery", "Docker"]
description: "Architected a multi-stage ETL data lake and high-throughput asynchronous REST API to process massive streams of connected vehicle telemetry and C2C traffic sensors for the Texas Department of Transportation."
---

## The Challenge

Transportation networks generate massive, continuous streams of data from disparate sources—including connected vehicle telemetry (INRIX, Waze, Wejo), traffic sensors (GRIDSMART, C2C feeds), and work zone statuses (WZDx). 

The Texas Department of Transportation (TxDOT) required a unified data integration framework (RIDI) that could handle both **offline historical analysis** ("Data-1") and **low-latency real-time streaming** ("Data-2"). The goal was to build a highly scalable, fault-tolerant "API of APIs" capable of ingesting raw telemetry, standardizing it to unified schemas (e.g., FHWA ARC-IT), and rapidly serving it to downstream consumers like dashboards and autonomous V2X infrastructure.

---

## Data Engineering Architecture

To meet the high-throughput requirements, I engineered a robust, modern backend architecture utilizing asynchronous Python and a modular ETL design.

### 1. Asynchronous FastAPI & SQLAlchemy
The core API is built on **FastAPI** utilizing an asynchronous **SQLAlchemy** (2.0) database layer (`async_sessionmaker`, `AsyncSession`). This non-blocking architecture allows the system to sustain high concurrency when querying millions of historical records or ingesting high-frequency live data streams. Database schemas and migrations are strictly version-controlled using **Alembic**.

### 2. Multi-Stage ETL Data Lake
Drawing inspiration from modern Data Lake patterns, the ingestion pipeline processes raw telemetry in distinct stages:
- **Intake:** Raw data (e.g., GRIDSMART XML or INRIX JSON) is captured and stored.
- **Canonicalization:** The data is transformed into structured, standardized JSON blobs.
- **Aggregation:** Data is temporally and spatially aggregated for rapid dashboard consumption.

### 3. Distributed Task Queues & Caching
To prevent redundant processing of expensive geospatial queries, I integrated **Redis** and **Celery**. 
- **Redis Caching:** Intermediate JSON processing states and query results are cached in Redis. If a downstream module requests recent data, it is served directly from the cache, bypassing the database layer.
- **Celery Workers:** Heavy analytical operations, such as PostGIS spatial bounding box intersections, are offloaded to distributed Celery workers, ensuring the main FastAPI event loop remains unblocked and responsive.

### 4. Containerization & CI/CD
The entire platform is orchestrated using **Docker Compose** for seamless deployment across local, development, and production environments. Automated testing suites (Pytest) and CI/CD pipelines run on GitHub Actions to ensure continuous integration stability.

---

## Impact and Key Accomplishments

- **High-Throughput Data Bridge:** Engineered an asynchronous data pipeline that streams live JSON telemetry directly into Pandas DataFrames, optimizing bulk upserts into PostgreSQL.
- **Reduced Latency:** The implementation of Redis caching drastically reduced redundant upstream data extractions, enabling the platform to achieve the strict low-latency requirements for "Data-2" realtime streaming.
- **Scalable Infrastructure:** Designed the system to be deployable on edge computing nodes (like NVIDIA Jetsons) as well as TACC supercomputing clusters using distributed message passing.
- **Enterprise Adoption:** RIDI now acts as the central data nervous system for validating connected vehicle technologies and powering statewide operational dashboards.

---
