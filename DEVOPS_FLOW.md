# DevOps Project Architecture Flow

```mermaid
graph TD
    %% Development
    Dev[Developer] -->|Git Push| Git[Git Repository]
    Git -->|Webhook| Jenkins[Jenkins CI/CD]
    
    %% CI/CD Pipeline
    subgraph Pipeline[CI/CD Pipeline]
        Jenkins -->|Build| Build[Build Stage]
        Build -->|Unit Test| Test[Test Stage]
        Test -->|Code Analysis| SonarQube[SonarQube Analysis]
        SonarQube -->|Quality Gate| Security[Security Scan]
        Security -->|Pass| Docker[Docker Build]
        Docker -->|Push| Registry[Docker Registry]
    end
    
    %% Deployment
    subgraph Deployment[Deployment Environment]
        Registry -->|Pull| K8s[Kubernetes Cluster]
        K8s -->|Deploy| Frontend[Frontend Service]
        K8s -->|Deploy| Backend[Backend API]
        K8s -->|Deploy| Database[(MongoDB)]
    end
    
    %% Monitoring Stack
    subgraph Monitoring[Monitoring & Logging]
        Frontend -->|Metrics| Prometheus[Prometheus]
        Backend -->|Metrics| Prometheus
        Database -->|Metrics| Prometheus
        K8s -->|Cluster Metrics| Prometheus
        
        Frontend -->|Logs| ELK[ELK Stack]
        Backend -->|Logs| ELK
        Database -->|Logs| ELK
        K8s -->|Cluster Logs| ELK
        
        Prometheus -->|Visualize| Grafana[Grafana Dashboards]
        ELK -->|Log Analysis| Kibana[Kibana]
        
        Prometheus -->|Alerts| AlertManager[Alert Manager]
        AlertManager -->|Notify| Teams[MS Teams/Slack]
    end
    
    %% Load Balancing & Security
    subgraph Network[Network Layer]
        Internet((Internet)) -->|HTTPS| CloudFlare[CloudFlare]
        CloudFlare -->|SSL| Nginx[Nginx Ingress]
        Nginx -->|Route| Frontend
        Nginx -->|Route| Backend
    end
    
    %% Infrastructure
    subgraph Infrastructure[Infrastructure]
        Terraform[Terraform] -->|Provision| Cloud[Cloud Infrastructure]
        Ansible[Ansible] -->|Configure| Cloud
        Cloud -->|Host| K8s
    end
    
    %% Backup & Recovery
    subgraph Backup[Backup & DR]
        Database -->|Backup| Backup_Storage[Backup Storage]
        K8s -->|State Backup| Velero[Velero Backup]
    end
    
    %% Style Definitions
    classDef primary fill:#4285F4,stroke:#2F528F,color:white;
    classDef secondary fill:#34A853,stroke:#0B5394,color:white;
    classDef monitoring fill:#EA4335,stroke:#990000,color:white;
    classDef network fill:#FBBC05,stroke:#B45F06,color:black;
    classDef storage fill:#47A248,stroke:#274E13,color:white;
    
    %% Apply Styles
    class Dev,Git,Jenkins primary;
    class Frontend,Backend,K8s secondary;
    class Prometheus,Grafana,AlertManager,ELK monitoring;
    class Internet,CloudFlare,Nginx network;
    class Database,Backup_Storage storage;
```

## Flow Description

### 1. Development Flow
- Developer pushes code to Git repository
- Git webhook triggers Jenkins pipeline
- Jenkins orchestrates the CI/CD process

### 2. CI/CD Pipeline
- Build: Compile and package application
- Test: Run unit and integration tests
- SonarQube: Code quality analysis
- Security: Vulnerability scanning
- Docker: Create container images
- Registry: Store container images

### 3. Deployment
- Kubernetes manages container orchestration
- Separate services for Frontend, Backend, and Database
- Automated deployment through CI/CD

### 4. Monitoring & Logging
- Prometheus collects metrics
- Grafana visualizes metrics
- ELK Stack for log aggregation
- Alert Manager for notifications
- Integration with MS Teams/Slack

### 5. Network & Security
- CloudFlare for DDoS protection
- Nginx Ingress for routing
- SSL/TLS encryption
- Network policies and security

### 6. Infrastructure
- Terraform for infrastructure as code
- Ansible for configuration management
- Cloud provider infrastructure

### 7. Backup & Recovery
- Database backups
- Kubernetes state backups using Velero
- Disaster recovery procedures
