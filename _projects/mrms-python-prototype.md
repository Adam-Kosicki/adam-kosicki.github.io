---
layout: project
title: "TxDOT MRMS Weather-Crash Ingestion Engine (Python Prototype)"
role: "Engineering Research Scientist | UT Austin"
order: 2
hero_metric: "10k updates/min | TACC HPC"
tech_stack: ["Python", "PostgreSQL", "TACC Lonestar6", "NumPy", "GRIB2 / wgrib2", "AsyncIO", "Slurm", "AWS S3"]
description: "Architected the initial Python & PostgreSQL processing engine correlating statewide TxDOT crash records with high-resolution NOAA weather grids, discovering a 4.5% data quantization bias and laying the foundation for TxDOT's enterprise Snowflake migration."
---

## Project Overview & Background

Before migrating the Texas Department of Transportation's (TxDOT) weather-to-crash ingestion pipeline into a native Snowflake Enterprise Data Platform (EDP), I architected and benchmarked the initial high-throughput **Python & PostgreSQL Data Processing Engine**.

To assess weather impact on traffic safety across Texas, the pipeline fused statewide crash records (CRIS database) with NOAA’s high-resolution **Multi-Radar Multi-Sensor (MRMS)** precipitation fields. This prototype proved the technical feasibility of spatial weather-crash matching, identified critical data quality flaws in legacy weather formats, and ultimately secured full TxDOT enterprise ITD funding (DMND0003521) for the production Snowflake deployment.

---

## Technical Highlights & Key Innovations

### 1. Discovery of 4.5% Data Quantization Bias
During evaluation of third-party weather data feeds (IEM NetCDF), I identified a subtle but critical data quality issue:
- **Lossy PNG Conversion:** The legacy pipeline relied on NetCDF files derived from an 8-bit PNG image conversion process, limiting precipitation data to 256 discrete bins with a minimum threshold of 0.02 mm.
- **Trace Rainfall Under-Reporting:** Comparative statistical analysis against raw NOAA GRIB2 binary data revealed that **4.5% of crash events recorded as "zero rainfall" in legacy NetCDF actually experienced detectable trace precipitation**.
- **Architectural Shift:** Based on these findings, I authored the recommendation report to transition TxDOT directly to NOAA's authoritative floating-point **GRIB2 `PrecipRate` data stream**, preserving complete scientific precision.

### 2. High-Throughput Spatial & Temporal Alignment
- **Nearest Neighbor & Interpolation:** Implemented spatial lookup algorithms mapping ~600,000 statewide crash coordinates to 2-minute gridded weather vectors across Texas.
- **Async & Vectorized Execution:** Leveraged Python `asyncio`, `multiprocessing`, and vectorized `NumPy` array operations to bypass Python GIL limitations during heavy mathematical computations.
- **Database Access Optimization:** Optimized PostgreSQL batch operations and spatial index indexing to achieve a sustained throughput of **~10,000 database updates per minute** on a single developer node (processing a full month of statewide crash data in ~2.5 hours).

### 3. Supercomputing Scale-Up (TACC Lonestar6)
- **High-Performance Computing:** Scaled the processing pipeline onto the **Texas Advanced Computing Center (TACC) Lonestar6 supercomputer**.
- **Slurm Parallel Job Scheduling:** Orchestrated parallel Slurm worker arrays across multiple compute nodes, scaling pipeline execution to process **millions of historical NOAA weather files and multi-year TxDOT crash archives in under 48 hours**.

---

## Architectural Evolution: Prototype to Enterprise Snowflake

This Python/PostgreSQL prototype served as the critical precursor and proof-of-concept for the enterprise platform:

| Pipeline Dimension | Python & PostgreSQL Prototype (Initial) | TxDOT Enterprise Snowflake Platform (Evolution) |
| :--- | :--- | :--- |
| **Execution Host** | Local Workstations & TACC Lonestar6 HPC | TxDOT Enterprise Data Platform (Snowflake EDP) |
| **Compute Engine** | Async Python + PostgreSQL + Slurm | Native Snowflake Python Stored Procedures & Tasks |
| **Data Format** | IEM NetCDF & Direct NOAA S3 GRIB2 | Direct NOAA S3 GRIB2 / Parquet Ingestion |
| **Data Precision** | Uncovered & Eliminated 4.5% Quantization Bias | Full Floating-Point Precision (`PrecipRate` mm/hr) |
| **Processing Speed** | ~10k DB updates/min (2.5h / month) | Statewide Automated Daily EDP Ingestion |

---

## Strategic Impact & Outcomes

1. **Executive Alignment & Handoff:** Authored technical documentation and strategic architectural reports (`CTR-TxDOT_CRIS-MRMS_Ingestion_Strategy_v1.0`) delivered to TxDOT leadership and data maturity teams.
2. **Secured Statewide Funding:** Demonstrated system scalability and data precision improvements, leading TxDOT ITD to designate the solution as a statewide safety initiative and approve full funding (DMND0003521).
3. **Paved Way for Cloud Migration:** Provided the verified data schemas, spatial logic, and performance baselines that enabled seamless migration into native Snowflake stored procedures.
