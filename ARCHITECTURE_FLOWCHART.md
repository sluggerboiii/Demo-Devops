%%{init: {
  'theme': 'base',
  'themeVariables': {
    'fontSize': '16px',
    'fontFamily': 'arial',
    'lineWidth': '2px',
    'primaryColor': '#4285F4',
    'primaryTextColor': '#000000',
    'primaryBorderColor': '#2F528F',
    'secondaryColor': '#E8F0FE',
    'tertiaryColor': '#F4B400'
  }
}}%%

graph TB
    %% Main Components
    User(("🌐 End User"))
    style User fill:#4285F4,stroke:#2F528F,stroke-width:3px,color:white,font-weight:bold,font-size:18px
    
    User --> |"HTTPS Request"| LB["🔄 Load Balancer/Nginx"]
    
    subgraph GCP["☁️ GCP VM Instance (Ubuntu 20.04)"]
        style GCP fill:#E8F0FE,stroke:#2F528F,stroke-width:4px,color:#000000,font-weight:bold,font-size:18px
        
        %% Nginx Layer
        subgraph Nginx["🛡️ Nginx Reverse Proxy"]
            style Nginx fill:#F4B400,stroke:#B45F06,stroke-width:3px,color:#000000,font-weight:bold,font-size:16px
            LB --> |"Port 443"| SSL["🔒 SSL Termination"]
            SSL --> |"Port 80"| Proxy["⚡ Proxy Rules"]
        end

        %% Application Layer
        subgraph DockerEnv["🐳 Docker Environment"]
            style DockerEnv fill:#2496ED,stroke:#0D47A1,stroke-width:3px,color:white,font-weight:bold,font-size:16px
            
            %% Frontend Stack
            subgraph Frontend["💻 Frontend Stack (Port 3001)"]
                style Frontend fill:#34A853,stroke:#0B5394,stroke-width:3px,color:white,font-weight:bold,font-size:16px
                React["⚛️ React.js App"] --> |"API Calls"| Node["📦 Node.js Backend"]
            end

            %% Database Layer
            subgraph Database["🗄️ Database Layer"]
                style Database fill:#47A248,stroke:#274E13,stroke-width:3px,color:white,font-weight:bold,font-size:16px
                Mongo[("📀 MongoDB")]
                Mongo --> |"Persistence"| MongoVol[("💾 MongoDB Volume")]
            end

            %% Monitoring Stack
            subgraph Monitoring["📊 Monitoring Stack"]
                style Monitoring fill:#EA4335,stroke:#990000,stroke-width:3px,color:white,font-weight:bold,font-size:16px
                Prom["🔍 Prometheus"] --> |"Collect Metrics"| TSDB[("📈 Time Series DB")]
                Prom --> |"Alert Rules"| AlertMgr["🚨 Alert Manager"]
                Graf["📉 Grafana"] --> |"Query Metrics"| Prom
                Graf --> |"Store"| GrafanaVol[("💾 Grafana Volume")]
            end

            %% Docker Networks
            subgraph Networks["🌐 Docker Networks"]
                style Networks fill:#FBBC05,stroke:#B45F06,stroke-width:3px,color:#000000,font-weight:bold,font-size:16px
                FrontNet["🔵 Frontend Network"]
                DBNet["🟢 Database Network"]
                MonNet["🔴 Monitoring Network"]
            end
        end
    end

    %% Connections with bold lines and clear labels
    Proxy --> |"Route"| Frontend
    Node --> |"Queries"| Mongo
    Prom --> |"Scrape Metrics"| Node
    
    %% Network Connections with distinct styles
    Frontend -.-> FrontNet
    Mongo -.-> DBNet
    Prom -.-> MonNet
    Graf -.-> MonNet

    %% Enhanced Styling for Components
    classDef container fill:#E1F5FE,stroke:#01579B,stroke-width:3px,color:#000000,font-weight:bold,font-size:14px
    classDef network fill:#FFF3E0,stroke:#FF6F00,stroke-width:3px,color:#000000,font-weight:bold,font-size:14px
    classDef volume fill:#F1F8E9,stroke:#33691E,stroke-width:3px,color:#000000,font-weight:bold,font-size:14px
    classDef proxy fill:#FCE4EC,stroke:#880E4F,stroke-width:3px,color:#000000,font-weight:bold,font-size:14px
    
    %% Apply styles to specific nodes
    class React,Node,SSL,Proxy,AlertMgr container
    class FrontNet,DBNet,MonNet network
    class MongoVol,GrafanaVol,TSDB volume
    class LB proxy

    %% Style all text elements
    style React font-weight:bold,font-size:14px
    style Node font-weight:bold,font-size:14px
    style Mongo font-weight:bold,font-size:14px
    style Prom font-weight:bold,font-size:14px
    style Graf font-weight:bold,font-size:14px
    style AlertMgr font-weight:bold,font-size:14px

    %% Link Styling
    linkStyle default stroke-width:2px,fill:none,stroke:#2F528F
```

## Architecture Flow Description

### 1. 🌐 Request Flow
1. **User sends HTTPS request**
2. **Nginx Load Balancer receives request (Port 443)**
3. **SSL termination occurs**
4. **Request proxied to appropriate service**

### 2. 💻 Application Flow
1. **Frontend Stack**
   - ⚛️ React.js handles UI
   - 📦 Node.js processes API requests
   - 🔵 Communicates via Frontend Network

2. **Database Operations**
   - 📀 MongoDB handles data storage
   - 💾 Persistent volume maintains data
   - 🟢 Secured via Database Network

3. **Monitoring Flow**
   - 🔍 Prometheus collects metrics
   - 📉 Grafana visualizes data
   - 🚨 Alert Manager handles notifications
   - 🔴 Isolated on Monitoring Network

### 3. 🌐 Network Isolation
- **Frontend Network**: UI and API traffic
- **Database Network**: Data operations
- **Monitoring Network**: Metrics and alerts

### 4. 💾 Data Persistence
- **MongoDB Volume**: Database files
- **Grafana Volume**: Dashboards and configs
- **TSDB**: Time series metrics

### 5. 🔒 Security Layers
1. **External Layer**
   - SSL/TLS encryption
   - Load balancing
   - DDoS protection

2. **Internal Layer**
   - Network isolation
   - Container security
   - Volume encryption
