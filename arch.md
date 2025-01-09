flowchart TD
    %% External Users and Entry Points
    User([External User]) --> |"HTTPS/HTTP\n(443/80)"| Nginx
    Developer([Developer]) --> |"Admin Access"| Konga
    
    subgraph "Load Balancer & Entry"
        Nginx[Nginx Reverse Proxy\nPorts: 80, 443]
        Nginx -->|"API Requests\n/api/*"| ApiGateway
        Nginx -->|"K8s Services\n/k8s/*"| Kong
    end
    
    subgraph "Kubernetes Services"
        Kong[Kong API Gateway\nNodePorts: 30080-30081] -->|"Route Rules"| Ingress[Kong Ingress Controller]
        Kong -->|"Admin API\n8001/8444"| KongAdmin[Kong Admin API\nNodePorts: 31001-31002]
        KongAdmin -->|"UI Access\n1337"| Konga[Konga UI\nNodePort: 31337]
        Ingress -->|"Test Endpoint\n/test"| TestService[HTTPBin Test Service\nPort: 80]
        
        subgraph "K8s Monitoring"
            NodeExporter[Node Exporter\nPort: 9100]
            KubeMetrics[Kubernetes Metrics]
            NodeExporter -->|"Node Metrics"| PrometheusK8s[Prometheus\nPort: 9090]
            KubeMetrics -->|"K8s Metrics"| PrometheusK8s
            PrometheusK8s -->|"Metrics Query"| GrafanaK8s[Grafana\nPort: 3000]
        end
    end
    
    subgraph "Application Stack"
        ApiGateway[API Gateway\nPort: 3000] -->|"Auth & Route"| Services
        
        subgraph "Services"
            NodeApp[Node.js App\nPort: 3000]
            GoService[Golang Service\nPort: 4000]
        end
        
        Services -->|"CRUD Ops\n27017"| MongoDB[(MongoDB\nPort: 27017)]
        
        subgraph "App Monitoring"
            AppMetrics[Application Metrics]
            AppLogs[Application Logs]
            
            Services -->|"Push Metrics"| AppMetrics
            Services -->|"Generate Logs"| AppLogs
            
            AppMetrics -->|"Scrape\n9090"| Prometheus
            AppLogs -->|"Collect"| Filebeat
        end
    end
    
    subgraph "Monitoring & Logging"
        subgraph "Metrics Pipeline"
            Prometheus[Prometheus\nPort: 9090] -->|"Query"| Grafana[Grafana\nPort: 3001]
            Grafana -->|"Alerts"| AlertManager[Alert Manager]
        end
        
        subgraph "Logging Pipeline"
            Filebeat[Filebeat] -->|"Ship Logs\n5044"| Logstash[Logstash\nPorts: 5000,5044,9600]
            Logstash -->|"Index\n9200"| Elasticsearch[(Elasticsearch\nPort: 9200)]
            Elasticsearch -->|"Visualize\n5601"| Kibana[Kibana\nPort: 5601]
        end
    end
    
    %% External Access Routes
    Developer -->|"Metrics UI\n3001"| Grafana
    Developer -->|"Logs UI\n5601"| Kibana
    
    %% Styling
    classDef external fill:#f9f,stroke:#333,stroke-width:4px
    classDef gateway fill:#85C1E9,stroke:#333,stroke-width:2px
    classDef kong fill:#bbf,stroke:#333,stroke-width:2px
    classDef service fill:#81C784,stroke:#333,stroke-width:2px
    classDef database fill:#7986CB,stroke:#333,stroke-width:2px
    classDef monitoring fill:#FF8A65,stroke:#333,stroke-width:2px
    classDef logging fill:#FDD835,stroke:#333,stroke-width:2px
    classDef metrics fill:#4DB6AC,stroke:#333,stroke-width:2px
    
    class User,Developer external
    class Nginx,ApiGateway gateway
    class Kong,KongAdmin,Konga kong
    class NodeApp,GoService,TestService service
    class MongoDB,Elasticsearch database
    class Prometheus,PrometheusK8s,NodeExporter,KubeMetrics,AppMetrics monitoring
    class Filebeat,Logstash,Kibana,AppLogs logging
    class Grafana,GrafanaK8s,AlertManager metrics
