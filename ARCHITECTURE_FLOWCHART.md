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

flowchart TB
    %% Main Components
    User(("🌐 End User"))
    style User fill:#4285F4,stroke:#2F528F,stroke-width:3px,color:white,font-weight:bold,font-size:18px
    
    User --> |"HTTPS Request"| LB["🔄 Load Balancer/Nginx"]
    
    subgraph GCP["☁️ GCP VM Instance"]
        style GCP fill:#E8F0FE,stroke:#2F528F,stroke-width:4px,color:#000000,font-weight:bold,font-size:18px
        
        %% Nginx Layer
        subgraph Nginx["🛡️ Nginx Proxy"]
            style Nginx fill:#F4B400,stroke:#B45F06,stroke-width:3px,color:#000000,font-weight:bold,font-size:16px
            LB --> |"443"| SSL["🔒 SSL"]
            SSL --> |"80"| Proxy["⚡ Proxy"]
        end

        %% Docker Environment
        subgraph Docker["🐳 Docker"]
            style Docker fill:#2496ED,stroke:#0D47A1,stroke-width:3px,color:white,font-weight:bold,font-size:16px
            
            %% Frontend
            subgraph Front["💻 Frontend"]
                style Front fill:#34A853,stroke:#0B5394,stroke-width:3px,color:white,font-weight:bold,font-size:16px
                React["⚛️ React"] --> |"API"| Node["📦 Node.js"]
            end

            %% Database
            subgraph DB["🗄️ Database"]
                style DB fill:#47A248,stroke:#274E13,stroke-width:3px,color:white,font-weight:bold,font-size:16px
                Mongo[("📀 MongoDB")]
                Mongo --> |"Data"| MongoVol[("💾 Volume")]
            end

            %% Monitoring
            subgraph Monitor["📊 Monitoring"]
                style Monitor fill:#EA4335,stroke:#990000,stroke-width:3px,color:white,font-weight:bold,font-size:16px
                Prom["🔍 Prometheus"]
                Graf["📉 Grafana"]
                Alert["🚨 Alerts"]
                Prom --> Alert
                Graf --> Prom
            end
        end
    end

    %% External Connections
    Proxy --> Front
    Node --> Mongo
    Prom --> Node

    %% Styling
    classDef default font-size:14px,font-weight:bold
    classDef container fill:#E1F5FE,stroke:#01579B,stroke-width:3px
    classDef network fill:#FFF3E0,stroke:#FF6F00,stroke-width:3px
    classDef volume fill:#F1F8E9,stroke:#33691E,stroke-width:3px

    %% Apply styles
    class React,Node,SSL,Proxy container
    class Mongo,MongoVol volume
    class Prom,Graf,Alert default
```

## Component Description

### 🌐 Request Flow
- **User** → **Load Balancer** → **SSL** → **Proxy** → **Frontend**

### 💻 Frontend Stack
- **React.js**: UI Components
- **Node.js**: API Backend
- **Port**: 3001

### 🗄️ Database
- **MongoDB**: Document Store
- **Volume**: Persistent Storage
- **Port**: 27017

### 📊 Monitoring
- **Prometheus**: Metrics Collection
- **Grafana**: Visualization
- **Alerts**: Notifications

### 🔒 Security
- SSL/TLS Encryption
- Network Isolation
- Container Security
