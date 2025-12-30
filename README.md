# Behavioral Data Platform

## Overview
This repository hosts a comprehensive system for tracking, segmenting, and acting on user behavioral data from mobile E-commerce applications. The platform handles high-volume event ingestion, processes data for segmentation, and executes multi-channel delivery jobs (Push, Email).

## Tech Stack
The project is structured as a monorepo utilizing the following technologies:
- **Frontend**: Next.js
- **Backend**: Go
- **Infrastructure**:
  - **Compute**: Google Cloud Run, AWS ECS (Fargate)
  - **Messaging**: Google Cloud Pub/Sub
  - **Data Warehouse**: BigQuery
  - **Database**: Amazon RDS

## System Architecture

The following diagram illustrates the data flow from user action to message delivery:

```mermaid
graph TD
    %% Nodes
    User[Mobile User]
    Ingest[Cloud Run Ingestion API]
    PubSub[Cloud Pub/Sub]
    BQ[BigQuery]
    Consumer[ECS Consumer Service]
    RDS[(RDS Database)]
    SegmentJob[ECS Segmentation Job]
    DeliveryJob[ECS Delivery Scheduler]
    PushEmail[External Delivery Channels]

    %% Flow 1: Ingestion
    User -->|Action Data| Ingest
    Ingest --> PubSub
    PubSub --> BQ

    %% Flow 2: Processing
    PubSub --> Consumer
    Consumer -->|Write| RDS

    %% Flow 3: Segmentation & Action
    RDS -->|Read User Data| SegmentJob
    SegmentJob -->|Configure Jobs| RDS
    
    %% Flow 4: Delivery
    RDS -->|Read Jobs| DeliveryJob
    DeliveryJob -->|Push / Email| PushEmail

    %% Styling
    style User fill:none,stroke:#333,stroke-width:2px
    style Ingest fill:none,stroke:#333,stroke-width:2px
    style PubSub fill:none,stroke:#333,stroke-width:2px
    style BQ fill:none,stroke:#333,stroke-width:2px
    style Consumer fill:none,stroke:#333,stroke-width:2px
    style RDS fill:none,stroke:#333,stroke-width:2px
    style SegmentJob fill:none,stroke:#333,stroke-width:2px
    style DeliveryJob fill:none,stroke:#333,stroke-width:2px
```

### Data Flow Components
1. **Ingestion**: 
   - Mobile apps send behavioral data (e.g., cart additions, purchases) to the **Cloud Run** endpoint.
   - Data is published to **Pub/Sub** and archived in **BigQuery** for analysis.

2. **Processing**:
   - An **ECS Consumer** service subscribes to the Pub/Sub topic and persists relevant state to **RDS**.

3. **Segmentation**:
   - **ECS Segmentation Tasks** run periodically to analyze user data in RDS based on configuration settings.
   - Targeted delivery jobs are created and stored.

4. **Delivery**:
   - **ECS Delivery Schedulers** pick up ready jobs and trigger external delivery APIs for Push Notifications or Emails.
