# DevOps Project Architecture Flow

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'fontSize': '18px',
    'fontFamily': 'Arial',
    'fontWeight': 'bold',
    'lineWidth': '3px'
  }
}}%%

graph TD
    %% Development
    Dev[<strong>Developer</strong>] -->|<strong>Git Push</strong>| Git[<strong>Git Repository</strong>]
    Git -->|<strong>Webhook Trigger</strong>| Jenkins[<strong>Jenkins CI/CD</strong>]
    
    %% CI/CD Pipeline
    subgraph Pipeline[<strong>CI/CD Pipeline</strong>]
        direction LR
        Jenkins -->|<strong>1. Checkout</strong>| Build[<strong>Build Stage</strong>]
        Build -->|<strong>2. Maven/NPM</strong>| Test[<strong>Test Stage</strong>]
        Test -->|<strong>3. Analysis</strong>| SonarQube[<strong>SonarQube</strong>]
        SonarQube -->|<strong>4. Scan</strong>| Security[<strong>Security Scan</strong>]
        Security -->|<strong>5. Build</strong>| Docker[<strong>Docker Build</strong>]
        Docker -->|<strong>6. Push</strong>| Registry[<strong>Docker Registry</strong>]
    end
    
    %% Deployment
    subgraph Deployment[<strong>Deployment Environment</strong>]
        Registry -->|<strong>Pull Images</strong>| K8s[<strong>Kubernetes Cluster</strong>]
        K8s -->|<strong>Deploy Pod</strong>| Frontend[<strong>Frontend Service</strong>]
        K8s -->|<strong>Deploy Pod</strong>| Backend[<strong>Backend API</strong>]
        K8s -->|<strong>StatefulSet</strong>| Database[(<strong>MongoDB</strong>)]
        
        Frontend -->|<strong>API Calls</strong>| Backend
        Backend -->|<strong>Query</strong>| Database
    end
    
    %% Monitoring Stack
    subgraph Monitoring[<strong>Monitoring & Logging</strong>]
        Frontend -->|<strong>App Metrics</strong>| Prometheus[<strong>Prometheus</strong>]
        Backend -->|<strong>API Metrics</strong>| Prometheus
        Database -->|<strong>DB Metrics</strong>| Prometheus
        K8s -->|<strong>Cluster Metrics</strong>| Prometheus
        
        Frontend -->|<strong>App Logs</strong>| ELK[<strong>ELK Stack</strong>]
        Backend -->|<strong>API Logs</strong>| ELK
        Database -->|<strong>DB Logs</strong>| ELK
        K8s -->|<strong>K8s Logs</strong>| ELK
        
        Prometheus -->|<strong>Metrics Display</strong>| Grafana[<strong>Grafana</strong>]
        ELK -->|<strong>Log Analysis</strong>| Kibana[<strong>Kibana</strong>]
        
        Prometheus -->|<strong>Alert Rules</strong>| AlertManager[<strong>Alert Manager</strong>]
        AlertManager -->|<strong>Notifications</strong>| Teams[<strong>Teams/Slack</strong>]
    end
    
    %% Load Balancing & Security
    subgraph Network[<strong>Network Layer</strong>]
        Internet((<strong>Internet</strong>)) -->|<strong>HTTPS</strong>| CloudFlare[<strong>CloudFlare</strong>]
        CloudFlare -->|<strong>SSL/WAF</strong>| Nginx[<strong>Nginx Ingress</strong>]
        Nginx -->|<strong>Route /app</strong>| Frontend
        Nginx -->|<strong>Route /api</strong>| Backend
    end
    
    %% Infrastructure
    subgraph Infrastructure[<strong>Infrastructure</strong>]
        Terraform[<strong>Terraform</strong>] -->|<strong>IaC</strong>| Cloud[<strong>Cloud Infrastructure</strong>]
        Ansible[<strong>Ansible</strong>] -->|<strong>Config</strong>| Cloud
        Cloud -->|<strong>Resources</strong>| K8s
    end
    
    %% Backup & Recovery
    subgraph Backup[<strong>Backup & DR</strong>]
        Database -->|<strong>Daily Backup</strong>| Backup_Storage[<strong>Backup Storage</strong>]
        K8s -->|<strong>State Backup</strong>| Velero[<strong>Velero Backup</strong>]
    end
    
    %% Style Definitions with Enhanced Colors
    classDef primary fill:#1a73e8,stroke:#0d47a1,color:white,stroke-width:3px;
    classDef secondary fill:#00c853,stroke:#1b5e20,color:white,stroke-width:3px;
    classDef monitoring fill:#e53935,stroke:#b71c1c,color:white,stroke-width:3px;
    classDef network fill:#ffd600,stroke:#ff6f00,color:black,stroke-width:3px;
    classDef storage fill:#6d4c41,stroke:#3e2723,color:white,stroke-width:3px;
    
    %% Apply Enhanced Styles
    class Dev,Git,Jenkins primary;
    class Frontend,Backend,K8s secondary;
    class Prometheus,Grafana,AlertManager,ELK monitoring;
    class Internet,CloudFlare,Nginx network;
    class Database,Backup_Storage storage;

    %% Global Styles
    style Pipeline fill:#f5f5f5,stroke:#212121,stroke-width:4px;
    style Deployment fill:#e3f2fd,stroke:#1565c0,stroke-width:4px;
    style Monitoring fill:#fce4ec,stroke:#c2185b,stroke-width:4px;
    style Network fill:#fff3e0,stroke:#ef6c00,stroke-width:4px;
    style Infrastructure fill:#e8f5e9,stroke:#2e7d32,stroke-width:4px;
    style Backup fill:#ede7f6,stroke:#4527a0,stroke-width:4px;
```

## Detailed Flow Description

### 1. **Development Workflow**
- **Code Push**: Developer commits and pushes code to Git repository
- **Webhook**: Automated trigger to Jenkins on code push
- **Pipeline Start**: Jenkins initiates the CI/CD pipeline

### 2. **CI/CD Pipeline Stages**
1. **Code Checkout**
   - Pull latest code from Git
   - Verify branch and credentials

2. **Build Process**
   - Install dependencies
   - Compile source code
   - Create artifacts

3. **Testing Phase**
   - Unit tests execution
   - Integration testing
   - Code coverage reports

4. **Code Quality**
   - SonarQube analysis
   - Code smells detection
   - Security vulnerabilities check

5. **Security Scanning**
   - Dependency scanning
   - Container security check
   - Compliance verification

6. **Containerization**
   - Build Docker images
   - Tag with version
   - Push to registry

### 3. **Deployment Process**
- **Kubernetes Operations**
  - Pull container images
  - Apply configurations
  - Update deployments
  - Health checks

- **Service Communication**
  - Frontend to Backend API calls
  - Backend to Database queries
  - Service mesh routing

### 4. **Monitoring & Alerting**
- **Metrics Collection**
  - Application performance
  - Resource utilization
  - Response times
  - Error rates

- **Log Management**
  - Centralized logging
  - Log parsing and indexing
  - Search and analysis

- **Alert Handling**
  - Threshold monitoring
  - Alert routing
  - Incident management

### 5. **Network & Security**
- **Traffic Flow**
  - CloudFlare DDoS protection
  - SSL/TLS termination
  - WAF rules
  - Load balancing

- **Access Control**
  - Authentication
  - Authorization
  - Network policies

### 6. **Infrastructure Management**
- **Resource Provisioning**
  - Infrastructure as Code
  - Environment configuration
  - Scaling policies

### 7. **Backup & Disaster Recovery**
- **Backup Procedures**
  - Automated daily backups
  - Point-in-time recovery
  - Backup verification

- **Disaster Recovery**
  - Recovery procedures
  - Failover testing
  - Business continuity
