```mermaid
graph TB
    subgraph "Google Cloud Platform VM"
        subgraph "Docker Environment"
            NGINX[/NGINX Reverse Proxy<br/>Port 80, 443/]
            
            subgraph "Services"
                API[/API Gateway<br/>Port 3000<br/>Node.js/]
                GO[/Go Service<br/>Port 4000/]
                NODE[/Node.js App<br/>Port 3000/]
            end
            
            subgraph "Monitoring Stack"
                GRAF[/Grafana<br/>Port 3001/]
                PROM[/Prometheus<br/>Port 9090/]
                MON[/Monitoring Script<br/>Python<br/>Every 30min/]
            end
            
            subgraph "Database"
                MONGO[/MongoDB<br/>Port 27017/]
            end
        end
    end
    
    USER[/External User/]
    EMAIL[/Gmail SMTP<br/>Alerts/]
    
    %% External connections
    USER -->|HTTPS| NGINX
    MON -->|Alerts| EMAIL
    
    %% Internal routing
    NGINX -->|Proxy| API
    API -->|/go/*| GO
    API -->|/api/*| NODE
    API -->|/grafana| GRAF
    API -->|/prometheus| PROM
    
    %% Monitoring connections
    PROM -->|Collect Metrics| API
    PROM -->|Collect Metrics| GO
    PROM -->|Collect Metrics| NODE
    GRAF -->|Query Metrics| PROM
    MON -->|Health Check| API
    MON -->|Health Check| GO
    MON -->|Health Check| NODE
    MON -->|Health Check| GRAF
    MON -->|Health Check| PROM
    
    %% Database connections
    NODE -.->|Read/Write| MONGO
    
    %% Styling
    classDef external fill:#f9f,stroke:#333,stroke-width:2px,color:#000
    classDef proxy fill:#85C1E9,stroke:#333,stroke-width:2px,color:#000
    classDef service fill:#82E0AA,stroke:#333,stroke-width:2px,color:#000
    classDef monitoring fill:#F8C471,stroke:#333,stroke-width:2px,color:#000
    classDef database fill:#BB8FCE,stroke:#333,stroke-width:2px,color:#000
    
    class USER,EMAIL external
    class NGINX proxy
    class API,GO,NODE service
    class GRAF,PROM,MON monitoring
    class MONGO database
```

# DevOps Project Architecture

## Components Description

### External Components
- **External User**: Accesses the application through HTTPS
- **Gmail SMTP**: Receives and sends monitoring alerts

### Infrastructure
- **Google Cloud VM**: Hosts all Docker containers
- **Docker**: Container platform running all services

### Core Services
1. **NGINX (Reverse Proxy)**
   - Ports: 80 (HTTP), 443 (HTTPS)
   - Handles SSL termination
   - Routes traffic to internal services

2. **API Gateway (Node.js)**
   - Port: 3000
   - Routes requests to appropriate services
   - Provides unified API interface
   - Endpoints:
     - `/` - API documentation
     - `/health` - Gateway health
     - `/metrics` - Prometheus metrics
     - `/api/*` - Node.js app routes
     - `/go/*` - Go service routes

3. **Go Service**
   - Port: 4000
   - Provides Go-based microservice
   - Endpoints:
     - `/health` - Service health
     - `/info` - Service information

4. **Node.js App**
   - Port: 3000
   - Main application service
   - Connected to MongoDB

5. **MongoDB**
   - Port: 27017
   - Primary database
   - Persistent storage for application data

### Monitoring Stack
1. **Prometheus**
   - Port: 9090
   - Metrics collection and storage
   - Monitors all services

2. **Grafana**
   - Port: 3001
   - Metrics visualization
   - Dashboard for monitoring

3. **Monitoring Script (Python)**
   - Runs every 30 minutes
   - Checks service health
   - Monitors memory usage
   - Sends email alerts for:
     - Service downtime
     - High memory usage (>75%)
     - Container issues

## Security Features
- HTTPS encryption
- Self-signed SSL certificates
- Containerized services
- Internal network isolation
- Secure email alerts (Gmail with app password)

## Data Flow
1. User requests come through HTTPS to NGINX
2. NGINX routes to API Gateway
3. API Gateway directs to appropriate service
4. Services interact with MongoDB as needed
5. Prometheus continuously collects metrics
6. Grafana visualizes metrics from Prometheus
7. Monitoring script checks health and sends alerts

## High Availability Features
- Container restart policies
- Health check endpoints
- Automated monitoring
- Real-time alerts
