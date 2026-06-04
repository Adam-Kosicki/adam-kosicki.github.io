---
layout: project
title: "WZDx & INRIX Travel Time Dashboard"
role: "Data Engineering Contractor | TxDOT"
order: 4
hero_metric: "Real-time Spatial-Temporal Traffic Analysis"
tech_stack: ["R Shiny", "PostgreSQL", "PostGIS", "REST APIs", "Leaflet"]
description: "Developed a full-stack data visualization application linking TxDOT Work Zone closures to INRIX roadway bounding boxes to dynamically evaluate traffic delays."
---

## The Challenge

Monitoring and assessing the impact of active highway construction on traffic flow is highly challenging. The Texas Department of Transportation (TxDOT) tracks lane closures and construction activities through the Work Zone Data Exchange (WZDx) feed. Meanwhile, high-frequency vehicle telemetry and travel times are provided by commercial vendors like INRIX. 

The core engineering challenge was **spatial-temporal integration**: 
1. **Disparate Datasets:** WZDx closures are represented as geospatial coordinates (points and lines), whereas INRIX traffic telemetry maps to proprietary, directional roadway segments.
2. **Dynamic Analysis:** We needed a fast way to dynamically compare the travel times on segments overlapping with active work zones against their historical "typical" travel times to quantify construction-induced delays.

To solve this, I engineered an end-to-end data visualization dashboard that aggregates these disparate data sources and enables interactive analysis of traffic delays around work zones.

---

## Technical Approach & Architecture

The application relies on a robust backend utilizing **R Shiny** to tie together PostgreSQL/PostGIS databases and internal REST APIs, producing a seamless front-end map.

### 1. Spatial Database Intersections (PostGIS)
To accurately identify which INRIX segments are affected by a given TxDOT work zone, standard identifier-based joins fail. I engineered a robust spatial query utilizing **PostGIS**. By running spatial intersections (`ST_Intersects`) between the boundary boxes (BBOX) of INRIX segments and the exact geometries of active WZDx construction sites, the pipeline filters the precise segments affected by closures across varying directions (Northbound/Southbound) and road decks. 

### 2. Internal REST API Integration
The system fetches real-time telemetry and metadata without direct, heavy database connections for every interaction. I developed R-based ingestion scripts utilizing `httr` and `jsonlite` to communicate with internal REST APIs. These APIs serve:
- Available date ranges for INRIX metrics and WZDx closures.
- Bounding coordinates for corridor segments.
- Travel time matrices.

By passing the spatially-joined segment IDs to the API, the system dynamically retrieves the "analysis" travel time (during construction) and the "typical" travel time (historical reference).

### 3. Interactive Data Visualization
The front-end user interface is built using a combination of **R Shiny**, **Leaflet**, and **DT**. 
- **Reactive Mapping:** When a planner selects a date range and highway corridor, the map dynamically renders the work zone boundaries. 
- **Delay Color-Coding:** The application calculates the travel time deviation (`analysis_tt - typical_tt`). If the delay is strictly positive, the segment is dynamically color-coded red on the Leaflet map; otherwise, it renders green, immediately alerting planners to severe congestion points.
- **Custom Event Bounding:** By injecting custom JavaScript via `shinyjs`, the application tracks map bounds (`getBounds`), allowing users to click and define their own ad-hoc start and end points for localized delay analysis.

---

## Impact and Key Accomplishments

- **Streamlined Workflow:** Provided TxDOT engineers with a unified visual tool to pinpoint the precise impact of work zone closures on daily commute times, replacing ad-hoc manual data extractions.
- **Spatial Accuracy:** Successfully fused unstructured point-surface geometries from WZDx with structured INRIX road segments utilizing fast spatial bounding box indexing in PostgreSQL.
- **Full-Stack Implementation:** Personally handled the entire development lifecycle—from backend PostGIS query optimization and REST API consumption to the deployment of the reactive, spinner-loaded R Shiny frontend layout.

---

## Visual Overview

<div class="project-video-container" style="text-align: center; margin: 2rem 0;">
  <div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; border-radius: 8px; border: 1px solid var(--card-border); box-shadow: 0 4px 20px rgba(0,0,0,0.15);">
    <iframe src="https://www.youtube.com/embed/bf5KoMFYjXE?autoplay=0&rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
  </div>
  <p style="font-size: 0.85rem; color: #888; margin-top: 0.5rem; font-style: italic;"><strong>Dashboard Demo:</strong> A live demonstration of the WZDx & INRIX Travel Time Dashboard in action.</p>
</div>

<div class="project-image-container" style="text-align: center; margin: 2rem 0;">
  <img src="{{ '/assets/images/wzdx_slide1.png' | relative_url }}" alt="Travel Times Estimations" style="max-width: 100%; border-radius: 8px; border: 1px solid var(--card-border); box-shadow: 0 4px 20px rgba(0,0,0,0.15);">
  <p style="font-size: 0.85rem; color: #888; margin-top: 0.5rem; font-style: italic;"><strong>Slide 1:</strong> Showing travel times estimations on March 26 using different methods, and corresponding method performance (IH 35 NB). The models shown in red and blue in the error summary panel are ML models, while the model shown in black corresponds to current practice. The estimation error of the latter is almost twice as large.</p>
</div>

<div class="project-image-container" style="text-align: center; margin: 2rem 0;">
  <img src="{{ '/assets/images/wzdx_slide2.png' | relative_url }}" alt="Average Model Performance" style="max-width: 100%; border-radius: 8px; border: 1px solid var(--card-border); box-shadow: 0 4px 20px rgba(0,0,0,0.15);">
  <p style="font-size: 0.85rem; color: #888; margin-top: 0.5rem; font-style: italic;"><strong>Slide 2:</strong> Average model performance by time of day since Jan 2024 and data availability. ML models are observed to have a lower error than current practice (naïve model), particularly during peak periods.</p>
</div>

<div class="project-image-container" style="text-align: center; margin: 2rem 0;">
  <img src="{{ '/assets/images/wzdx_slide3.png' | relative_url }}" alt="Summary Tab" style="max-width: 100%; border-radius: 8px; border: 1px solid var(--card-border); box-shadow: 0 4px 20px rgba(0,0,0,0.15);">
  <p style="font-size: 0.85rem; color: #888; margin-top: 0.5rem; font-style: italic;"><strong>Slide 3:</strong> Summary tab showing last travel time prediction for all active models and a summary of average model performance over the last three months to support data sharing decisions.</p>
</div>
