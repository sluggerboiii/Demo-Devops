```mermaid
graph TB
    %% Main Components
    User((End User)) --> |HTTPS Request| LB[Load Balancer/Nginx]
    
    subgraph GCP["GCP VM Instance (Ubuntu 20.04)"]
        %% Nginx Layer
        subgraph Nginx["Nginx Reverse Proxy"]
            LB --> |Port 443| SSL[SSL Termination]
            SSL --> |Port 80| Proxy[Proxy Rules]
        end

        %% Application Layer
        subgraph DockerEnv["Docker Environment"]
            %% Frontend Stack
            subgraph Frontend["Frontend Stack (Port 3001)"]
                React[React.js App] --> |API Calls| Node[Node.js Backend]
            end

            %% Database Layer
            subgraph Database["Database Layer"]
                Mongo[(MongoDB)]
                Mongo --> |Persistence| MongoVol[(MongoDB Volume)]
            end

            %% Monitoring Stack
            subgraph Monitoring["Monitoring Stack"]
                Prom[Prometheus] --> |Collect Metrics| TSDB[(Time Series DB)]
                Prom --> |Alert Rules| AlertMgr[Alert Manager]
                Graf[Grafana] --> |Query Metrics| Prom
                Graf --> |Store| GrafanaVol[(Grafana Volume)]
            end

            %% Docker Networks
            subgraph Networks["Docker Networks"]
                FrontNet[Frontend Network]
                DBNet[Database Network]
                MonNet[Monitoring Network]
            end
        end
    end

    %% Connections
    Proxy --> |Route| Frontend
    Node --> |Queries| Mongo
    Prom --> |Scrape Metrics| Node
    
    %% Network Connections
    Frontend -.-> FrontNet
    Mongo -.-> DBNet
    Prom -.-> MonNet
    Graf -.-> MonNet

    %% Styling
    classDef container fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef network fill:#fff3e0,stroke:#ff6f00,stroke-width:2px
    classDef volume fill:#f1f8e9,stroke:#33691e,stroke-width:2px
    classDef proxy fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    
    class Frontend,Database,Monitoring container
    class FrontNet,DBNet,MonNet network
    class MongoVol,GrafanaVol,TSDB volume
    class Nginx,LB,SSL,Proxy proxy

    %% Click actions
    click Frontend "https://github.com/yourusername/frontend" "View Frontend Code"
    click Database "https://github.com/yourusername/database" "View Database Config"
    click Monitoring "https://github.com/yourusername/monitoring" "View Monitoring Setup"
```

## Architecture Flow Description

### 1. Request Flow
1. User sends HTTPS request
2. Nginx Load Balancer receives request (Port 443)
3. SSL termination occurs
4. Request proxied to appropriate service

### 2. Application Flow
1. Frontend Stack
   - React.js handles UI
   - Node.js processes API requests
   - Communicates via Frontend Network

2. Database Operations
   - MongoDB handles data storage
   - Persistent volume maintains data
   - Secured via Database Network

3. Monitoring Flow
   - Prometheus collects metrics
   - Grafana visualizes data
   - Alert Manager handles notifications
   - Isolated on Monitoring Network

### 3. Network Isolation
- Frontend Network: UI and API traffic
- Database Network: Data operations
- Monitoring Network: Metrics and alerts

### 4. Data Persistence
- MongoDB Volume: Database files
- Grafana Volume: Dashboards and configs
- TSDB: Time series metrics

### 5. Security Layers
1. External Layer
   - SSL/TLS encryption
   - Load balancing
   - DDoS protection

2. Internal Layer
   - Network isolation
   - Container security
   - Volume encryption
