%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '16px', 'fontFamily': 'arial', 'lineWidth': '2px' }}}%%
flowchart TD
    %% External Entry Points
    Client([<b>External Client</b>]) -->|"HTTPS/HTTP\n443/80"| Nginx
    DevOps([<b>DevOps Team</b>]) -->|"Admin Access"| AdminUIs
    
    subgraph "Entry Layer"
        Nginx[<b>Nginx Reverse Proxy</b>\nImage: nginx:latest\nPorts: 80, 443]
        
        subgraph "Admin UIs" [Admin Interfaces]
            direction LR
            Konga[<b>Konga UI</b>\nImage: pantsel/konga\nPort: 31337]
            Grafana[<b>Grafana</b>\nImage: grafana/grafana\nPort: 3001]
            Kibana[<b>Kibana</b>\nImage: elastic/kibana:8.11.1\nPort: 5601]
        end
    end
    
    subgraph "API Gateway Layer"
        Nginx -->|"/api/*"| CustomGW[<b>Custom API Gateway</b>\nImage: devops-project-api-gateway\nPort: 3000]
        Nginx -->|"/k8s/*"| Kong[<b>Kong Gateway</b>\nPorts: 30080-30081]
        
        Kong -->|"Admin API\n8001/8444"| KongAdmin[<b>Kong Admin API</b>\nPorts: 31001-31002]
        KongAdmin --> Konga
    end
    
    subgraph "Service Layer"
        CustomGW -->|"Route & Auth"| Services
        Kong -->|"K8s Ingress"| Services
        
        subgraph "Services" [Microservices]
            NodeApp[<b>Node.js App</b>\nImage: devops-project-nodejs-app\nPort: 3000]
            GoService[<b>Golang Service</b>\nImage: devops-project-golang-service\nPort: 4000]
            TestService[<b>HTTPBin Service</b>\nImage: kennethreitz/httpbin\nPort: 80]
        end
    end
    
    subgraph "Data Layer"
        Services -->|"CRUD Operations"| Databases
        
        subgraph "Databases"
            MongoDB[(<b>MongoDB</b>\nImage: mongo:latest\nPort: 27017)]
            Elasticsearch[(<b>Elasticsearch</b>\nImage: elastic/elasticsearch:8.11.1\nPort: 9200)]
        end
    end
    
    subgraph "Monitoring Stack"
        Services -->|"Push Metrics"| Metrics
        Services -->|"Generate Logs"| Logs
        
        subgraph "Metrics" [Metrics Collection]
            AppMetrics[<b>Application Metrics</b>]
            NodeMetrics[<b>Node Metrics</b>\nPort: 9100]
            K8sMetrics[<b>Kubernetes Metrics</b>]
            
            AppMetrics & NodeMetrics & K8sMetrics -->|"Collect"| Prometheus[<b>Prometheus</b>\nImage: prom/prometheus\nPort: 9090]
            Prometheus -->|"Visualize"| Grafana
            Grafana -->|"Alert"| AlertManager[<b>Alert Manager</b>]
        end
        
        subgraph "Logs" [Log Collection]
            Filebeat[<b>Filebeat</b>\nImage: elastic/filebeat:8.11.1]
            Logstash[<b>Logstash</b>\nImage: elastic/logstash:8.11.1\nPorts: 5000,5044,9600]
            
            Filebeat -->|"Ship\n5044"| Logstash
            Logstash -->|"Index\n9200"| Elasticsearch
            Elasticsearch -->|"Visualize\n5601"| Kibana
        end
    end
    
    %% Styling with stronger colors and thicker borders
    classDef external fill:#ff9cce,stroke:#333,stroke-width:4px,color:#000,font-weight:bold
    classDef gateway fill:#85C1E9,stroke:#333,stroke-width:3px,color:#000
    classDef service fill:#81C784,stroke:#333,stroke-width:3px,color:#000
    classDef database fill:#7986CB,stroke:#333,stroke-width:3px,color:#000
    classDef monitoring fill:#FF8A65,stroke:#333,stroke-width:3px,color:#000
    classDef logging fill:#FDD835,stroke:#333,stroke-width:3px,color:#000
    classDef ui fill:#B39DDB,stroke:#333,stroke-width:3px,color:#000
    
    class Client,DevOps external
    class Nginx,CustomGW,Kong,KongAdmin gateway
    class NodeApp,GoService,TestService service
    class MongoDB,Elasticsearch database
    class Prometheus,AppMetrics,NodeMetrics,K8sMetrics,AlertManager monitoring
    class Filebeat,Logstash logging
    class Konga,Grafana,Kibana ui

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
