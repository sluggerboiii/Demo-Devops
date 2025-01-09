graph TD
    User((End User))
    LB[Load Balancer/Nginx]
    SSL[SSL Termination]
    Proxy[Proxy Rules]
    React[React.js Frontend]
    Node[Node.js Backend]
    Mongo[(MongoDB)]
    Prom[Prometheus]
    Graf[Grafana]
    Alert[Alert Manager]
    
    %% Main Flow
    User --> |HTTPS| LB
    LB --> |443| SSL
    SSL --> |80| Proxy
    Proxy --> React
    React --> Node
    Node --> Mongo
    
    %% Monitoring Flow
    Prom --> Node
    Prom --> Alert
    Graf --> Prom
    
    %% Grouping
    subgraph GCP[GCP VM Instance]
        subgraph Nginx[Nginx Layer]
            LB
            SSL
            Proxy
        end
        
        subgraph Docker[Docker Environment]
            subgraph Frontend[Frontend Stack]
                React
                Node
            end
            
            subgraph Database[Database Layer]
                Mongo
            end
            
            subgraph Monitoring[Monitoring Stack]
                Prom
                Graf
                Alert
            end
        end
    end
    
    %% Styling
    classDef user fill:#4285F4,stroke:#2F528F,color:white
    classDef nginx fill:#F4B400,stroke:#B45F06,color:black
    classDef frontend fill:#34A853,stroke:#0B5394,color:white
    classDef database fill:#47A248,stroke:#274E13,color:white
    classDef monitoring fill:#EA4335,stroke:#990000,color:white
    
    class User user
    class LB,SSL,Proxy nginx
    class React,Node frontend
    class Mongo database
    class Prom,Graf,Alert monitoring
```

## Architecture Components

### Request Flow
- End User sends HTTPS request
- Load Balancer receives on port 443
- SSL termination
- Proxy to frontend application

### Application Stack
- Frontend: React.js UI
- Backend: Node.js API (Port 3001)
- Database: MongoDB (Port 27017)

### Monitoring
- Prometheus: Metrics collection
- Grafana: Visualization
- Alert Manager: Notifications

### Security
- SSL/TLS encryption
- Network isolation
- Container security
