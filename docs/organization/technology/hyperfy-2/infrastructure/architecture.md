---
sidebar_position: 2
title: Architecture
description: Detailed system architecture and design patterns
---

# System Architecture

## Overview Diagram

```mermaid
graph TD
    subgraph VPC[Virtual Private Cloud]
        LB[Load Balancer] --> Services
        subgraph Services[ECS Services]
            S1[Service 1]
            S2[Service 2]
            SN[Service N]
        end
        Services --> EFS[Shared EFS]
    end
    Client --> LB
    Services --> S3[S3 Configs]
```

## Service Architecture

### Container Structure
```mermaid
graph TD
    subgraph Container[Docker Container]
        PM2[PM2 Process Manager]
        subgraph Processes[5 Processes]
            P1[Process 1:Port X]
            P2[Process 2:Port Y]
            P3[Process 3:Port Z]
            P4[Process 4:Port W]
            P5[Process 5:Port V]
        end
        PM2 --> Processes
    end
```

## Network Architecture

### Traffic Flow
- External requests through Load Balancer
- Subdomain-based routing
- Internal service communication

### Port Management
- Each service exposes multiple ports
- Port mapping through Load Balancer
- Internal service discovery

## Storage Architecture

### Shared Storage
- EFS mount points
- Access patterns
- Performance considerations

### Configuration Storage
- S3 bucket organization
- PM2 configuration files
- Environment variables 