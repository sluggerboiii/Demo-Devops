```mermaid
flowchart TD
    Client([Client]) --> Nginx[Nginx Reverse Proxy\nPorts: 80, 443]
    
    subgraph "Kubernetes Cluster"
        Kong[Kong API Gateway\nNodePorts: 30080-30081]
        KongAdmin[Kong Admin API\nNodePorts: 31001-31002]
        Konga[Konga UI\nNodePort: 31337]
        Ingress[Kong Ingress Controller]
        TestService[HTTPBin Test Service\nPort: 80]
        
        KongAdmin --> Konga
        Kong --> Ingress
        Ingress --> TestService
    end
    
    subgraph "Docker Services"
        direction TB
        ApiGateway[API Gateway\nPort: 3000]
        NodeApp[Node.js App\nPort: 3000]
        GoService[Golang Service\nPort: 4000]
        MongoDB[(MongoDB\nPort: 27017)]
        
        subgraph "Monitoring Stack"
            Prometheus[Prometheus\nPort: 9090]
            Grafana[Grafana\nPort: 3001]
            NodeExporter[Node Exporter\nPort: 9100]
            
            Prometheus --> Grafana
            NodeExporter --> Prometheus
        end
        
        subgraph "ELK Stack"
            Elasticsearch[(Elasticsearch\nPort: 9200)]
            Logstash[Logstash\nPorts: 5000,5044,9600]
            Kibana[Kibana\nPort: 5601]
            Filebeat[Filebeat]
            
            Filebeat --> Logstash
            Logstash --> Elasticsearch
            Elasticsearch --> Kibana
        end
        
        ApiGateway --> NodeApp
        ApiGateway --> GoService
        NodeApp --> MongoDB
        GoService --> MongoDB
    end
    
    Nginx --> Kong
    Nginx --> ApiGateway
    
    style Client fill:#f9f,stroke:#333,stroke-width:4px
    style Nginx fill:#85C1E9,stroke:#333,stroke-width:2px
    style Kong fill:#bbf,stroke:#333,stroke-width:2px
    style KongAdmin fill:#ddf,stroke:#333,stroke-width:2px
    style Konga fill:#ddf,stroke:#333,stroke-width:2px
    style Ingress fill:#bfb,stroke:#333,stroke-width:2px
    style TestService fill:#fbb,stroke:#333,stroke-width:2px
    style ApiGateway fill:#FFB74D,stroke:#333,stroke-width:2px
    style NodeApp fill:#81C784,stroke:#333,stroke-width:2px
    style GoService fill:#64B5F6,stroke:#333,stroke-width:2px
    style MongoDB fill:#7986CB,stroke:#333,stroke-width:2px
    style Prometheus fill:#FF8A65,stroke:#333,stroke-width:2px
    style Grafana fill:#4DB6AC,stroke:#333,stroke-width:2px
    style NodeExporter fill:#FF8A65,stroke:#333,stroke-width:2px
    style Elasticsearch fill:#FDD835,stroke:#333,stroke-width:2px
    style Logstash fill:#FDD835,stroke:#333,stroke-width:2px
    style Kibana fill:#FDD835,stroke:#333,stroke-width:2px
    style Filebeat fill:#FDD835,stroke:#333,stroke-width:2px
```

## Architecture Overview

This diagram represents the complete architecture of the DevOps project, including all components:

1. **Frontend Entry Points**
   - Nginx Reverse Proxy (Ports 80, 443)
   - Routes traffic to either Kong API Gateway or the custom API Gateway

2. **Kubernetes Components**
   - Kong API Gateway (NodePorts 30080-30081)
   - Kong Admin API (NodePorts 31001-31002)
   - Konga UI (NodePort 31337)
   - Kong Ingress Controller
   - HTTPBin Test Service

3. **Application Services**
   - Custom API Gateway (Port 3000)
   - Node.js Application (Port 3000)
   - Golang Service (Port 4000)
   - MongoDB Database (Port 27017)

4. **Monitoring Stack**
   - Prometheus (Port 9090)
   - Grafana (Port 3001)
   - Node Exporter (Port 9100)

5. **Logging Stack (ELK)**
   - Elasticsearch (Port 9200)
   - Logstash (Ports 5000, 5044, 9600)
   - Kibana (Port 5601)
   - Filebeat

### Data Flow
- External requests come through Nginx reverse proxy
- Traffic is routed either to Kong API Gateway (for Kubernetes services) or to the custom API Gateway
- Application services (Node.js and Golang) interact with MongoDB
- Monitoring data is collected by Prometheus and visualized in Grafana
- Logs are collected by Filebeat, processed by Logstash, stored in Elasticsearch, and visualized in Kibana
