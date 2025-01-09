```mermaid
flowchart TD
    %% External Entry Points
    Client([External Client]) -->|"HTTPS/HTTP\n443/80"| Nginx
    DevOps([DevOps Team]) -->|"Admin Access"| AdminUIs
    
    subgraph "Entry Layer"
        Nginx[Nginx Reverse Proxy\nImage: nginx:latest\nPorts: 80, 443]
        
        subgraph "Admin UIs" [Admin Interfaces]
            direction LR
            Konga[Konga UI\nImage: pantsel/konga\nPort: 31337]
            Grafana[Grafana\nImage: grafana/grafana\nPort: 3001]
            Kibana[Kibana\nImage: elastic/kibana:8.11.1\nPort: 5601]
        end
    end
    
    subgraph "API Gateway Layer"
        Nginx -->|"/api/*"| CustomGW[Custom API Gateway\nImage: devops-project-api-gateway\nPort: 3000]
        Nginx -->|"/k8s/*"| Kong[Kong Gateway\nPorts: 30080-30081]
        
        Kong -->|"Admin API\n8001/8444"| KongAdmin[Kong Admin API\nPorts: 31001-31002]
        KongAdmin --> Konga
    end
    
    subgraph "Service Layer"
        CustomGW -->|"Route & Auth"| Services
        Kong -->|"K8s Ingress"| Services
        
        subgraph "Services" [Microservices]
            NodeApp[Node.js App\nImage: devops-project-nodejs-app\nPort: 3000]
            GoService[Golang Service\nImage: devops-project-golang-service\nPort: 4000]
            TestService[HTTPBin Service\nImage: kennethreitz/httpbin\nPort: 80]
        end
    end
    
    subgraph "Data Layer"
        Services -->|"CRUD Operations"| Databases
        
        subgraph "Databases"
            MongoDB[(MongoDB\nImage: mongo:latest\nPort: 27017)]
            Elasticsearch[(Elasticsearch\nImage: elastic/elasticsearch:8.11.1\nPort: 9200)]
        end
    end
    
    subgraph "Monitoring Stack"
        Services -->|"Push Metrics"| Metrics
        Services -->|"Generate Logs"| Logs
        
        subgraph "Metrics" [Metrics Collection]
            AppMetrics[Application Metrics]
            NodeMetrics[Node Metrics\nPort: 9100]
            K8sMetrics[Kubernetes Metrics]
            
            AppMetrics & NodeMetrics & K8sMetrics -->|"Collect"| Prometheus[Prometheus\nImage: prom/prometheus\nPort: 9090]
            Prometheus -->|"Visualize"| Grafana
            Grafana -->|"Alert"| AlertManager[Alert Manager]
        end
        
        subgraph "Logs" [Log Collection]
            Filebeat[Filebeat\nImage: elastic/filebeat:8.11.1]
            Logstash[Logstash\nImage: elastic/logstash:8.11.1\nPorts: 5000,5044,9600]
            
            Filebeat -->|"Ship\n5044"| Logstash
            Logstash -->|"Index\n9200"| Elasticsearch
            Elasticsearch -->|"Visualize\n5601"| Kibana
        end
    end
    
    %% Styling
    classDef external fill:#f9f,stroke:#333,stroke-width:4px
    classDef gateway fill:#85C1E9,stroke:#333,stroke-width:2px
    classDef service fill:#81C784,stroke:#333,stroke-width:2px
    classDef database fill:#7986CB,stroke:#333,stroke-width:2px
    classDef monitoring fill:#FF8A65,stroke:#333,stroke-width:2px
    classDef logging fill:#FDD835,stroke:#333,stroke-width:2px
    classDef ui fill:#B39DDB,stroke:#333,stroke-width:2px
    
    class Client,DevOps external
    class Nginx,CustomGW,Kong,KongAdmin gateway
    class NodeApp,GoService,TestService service
    class MongoDB,Elasticsearch database
    class Prometheus,AppMetrics,NodeMetrics,K8sMetrics,AlertManager monitoring
    class Filebeat,Logstash logging
    class Konga,Grafana,Kibana ui

```

## System Architecture Details

### Docker Images & Versions
1. **Gateway & Proxy**
   - `nginx:latest` - Reverse Proxy
   - `devops-project-api-gateway:latest` - Custom API Gateway
   - Kong (Kubernetes Deployment)

2. **Applications**
   - `devops-project-nodejs-app:latest` - Node.js Application
   - `devops-project-golang-service:latest` - Golang Service
   - `kennethreitz/httpbin` - Test Service

3. **Databases**
   - `mongo:latest` - MongoDB
   - `elastic/elasticsearch:8.11.1` - Elasticsearch

4. **Monitoring & Logging**
   - `prom/prometheus:latest` - Prometheus
   - `grafana/grafana:latest` - Grafana
   - `elastic/filebeat:8.11.1` - Filebeat
   - `elastic/logstash:8.11.1` - Logstash
   - `elastic/kibana:8.11.1` - Kibana

### Network Flow
1. **External Access**
   - All external traffic through Nginx (80/443)
   - Admin access to management UIs through specific ports

2. **API Routing**
   - `/api/*` → Custom API Gateway
   - `/k8s/*` → Kong API Gateway
   - Internal service communication through Docker network

3. **Data Flow**
   - Services → MongoDB (CRUD operations)
   - Metrics → Prometheus → Grafana
   - Logs → Filebeat → Logstash → Elasticsearch → Kibana

### Port Mappings
1. **Public Ports**
   - Nginx: 80, 443
   - Kong: 30080, 30081
   - Grafana: 3001
   - Kibana: 5601

2. **Internal Ports**
   - MongoDB: 27017
   - Elasticsearch: 9200
   - Logstash: 5000, 5044, 9600
   - Node.js App: 3000
   - Golang Service: 4000
   - Prometheus: 9090

### Security Zones
1. **Public Zone**
   - Nginx Reverse Proxy
   - Kong API Gateway
   - Management UIs (with authentication)

2. **Service Zone**
   - Application Services
   - API Gateways
   - Service-to-service communication

3. **Data Zone**
   - Databases
   - Persistent Storage
   - Logging Infrastructure
