**KHÔNG ĐÚNG!** SWIM không phải VPN đơn thuần. Đây là hiểu lầm rất phổ biến. Để tôi giải thích rõ:

## **SWIM INFRASTRUCTURE LÀ GÌ:**

### **SWIM ≠ VPN:**
```
VPN:
- Chỉ là "tunnel" mã hóa
- Point-to-point connection
- Không có business logic
- Chỉ transport layer

SWIM:
- Service-oriented architecture  
- Message routing & transformation
- Business rule enforcement
- Service discovery & registry
- Security + routing + logic
```

### **SWIM là "Internet cho hàng không":**
```
Giống như Internet:
- Multiple nodes kết nối với nhau
- Standardized protocols (HTTP, SOAP, REST)
- Service discovery (như DNS)
- Message routing (như routers)
- Security certificates (như HTTPS)

Nhưng specific cho aviation:
- FIXM message format
- Flight plan routing rules
- Aviation security standards
- Regulatory compliance
```

## **TẠI SAO CẦN SWIM CHO QUERY SERVICE:**

### **Không có SWIM:**
```
❌ Airlines gọi trực tiếp VATM server
❌ Point-to-point connections everywhere  
❌ Mỗi airline cần riêng protocol
❌ Không tương thích với FF-ICE standard
❌ Không kết nối được với ANSPs khác
```

### **Có SWIM:**
```
✅ Standardized message exchange
✅ Airlines connect qua SWIM infrastructure
✅ Automatic routing tới đúng services
✅ Compatible với international systems
✅ Security & authentication tự động
```

## **SWIM ARCHITECTURE:**

### **SWIM Network Structure:**
```
┌─────────────────────────────────────────────────┐
│                SWIM Network                     │
│  ┌───────────────┬─────────────┬─────────────┐  │
│  │ Service       │ Message     │ Security    │  │  
│  │ Registry      │ Router      │ Manager     │  │
│  └───────────────┴─────────────┴─────────────┘  │
│         ↕              ↕              ↕         │
│  ┌─────────┬──────────┬──────────┬──────────┐   │
│  │ VATM    │Airlines  │Other     │Airports  │   │
│  │Services │          │ANSPs     │          │   │
│  └─────────┴──────────┴──────────┴──────────┘   │
└─────────────────────────────────────────────────┘
```

### **SWIM Components:**
```
1. Service Registry:
   - Discover available services
   - Service descriptions (WSDL)
   - Endpoint URLs

2. Message Router:  
   - Route messages to correct services
   - Transform message formats
   - Handle errors & retries

3. Security Infrastructure:
   - PKI certificates
   - Authentication & authorization
   - Message encryption

4. Monitoring & Management:
   - Service health monitoring
   - Performance metrics
   - Audit logging
```

## **QUAN HỆ VỚI QUERY SERVICE:**

### **Query Service cần SWIM để:**

#### **1. Receive requests:**
```
Thay vì:
Airline → Direct call → VATM Query Service

Có SWIM:
Airline → SWIM → Message routing → VATM Query Service
```

#### **2. Service discovery:**
```
Airlines không cần biết:
- VATM server IP address
- Authentication details  
- Message formats

SWIM handles:
- Service lookup
- Automatic routing
- Format conversion
```

#### **3. Security:**
```
SWIM provides:
- Certificate-based authentication
- Message encryption
- Access control
- Audit trails
```

## **TRIỂN KHAI THỰC TẾ CHO VATM:**

### **Option 1: Full SWIM Implementation**
```
VATM builds complete SWIM infrastructure:
- Service registry server
- Message routing server
- Security infrastructure
- Monitoring systems

Cost: $500K-1M
Timeline: 12-18 months
```

### **Option 2: Connect to existing SWIM**
```
VATM joins regional SWIM network:
- Connect tới ASEAN SWIM infrastructure
- Use existing SWIM services
- Focus on Query Service development

Cost: $50K-100K connection fees
Timeline: 3-6 months
```

### **Option 3: SWIM-compatible (Recommended):**
```
Build Query Service SWIM-compatible:
- Implement SWIM protocols
- Support FIXM messaging  
- Certificate-based security
- Can connect to SWIM later

Cost: $10K-20K extra development
Timeline: +1-2 months
```

## ** SWIM:**

### **" thực hiện SWIM test environment"**
```
Có nghĩa là:
✅ VATM đã có SWIM infrastructure cơ bản
✅ Test environment đã sẵn sàng
✅ Query Service có thể connect ngay

→ Bạn KHÔNG cần build SWIM from scratch!
```

### **Cần làm gì:**
```
1. Tìm hiểu SWIM setup của anh Đức
2. Query Service connect tới SWIM test environment
3. Test message exchange qua SWIM
4. Verify FIXM compatibility
```

## **TIMELINE VỚI SWIM:**

### **Development với SWIM integration:**
```
Month 1: Basic Query Service (standalone)
Month 2: SWIM integration development
Month 3: Testing với SWIM environment
Month 4: Production deployment qua SWIM
```

### **Additional work needed:**
```
SWIM Client Library:
- SOAP/REST client for SWIM services
- Certificate management
- Message formatting
- Error handling

Estimate: +2-4 weeks development
```

## **RECOMMENDATION:**

### **Approach:**
```
1. Build Query Service standalone first
2. Add SWIM integration layer
3. Test với SWIM environment
4. Deploy production với SWIM connectivity

Benefits:
✅ Can test Query Service independently  
✅ SWIM integration is additional layer
✅ Reuse existing SWIM infrastructure
✅ Compatible với FF-ICE standards
```

**KẾT LUẬN:** 
- SWIM ≠ VPN, it's much more complex
- Nhưng anh Đức đã setup → bạn chỉ cần integrate
- Query Service + SWIM integration = 1-2 tháng extra work
- Absolutely necessary cho FF-ICE compliance


**TỐT!** Tự làm SWIM sẽ hiểu sâu hơn và kiểm soát tốt hơn. Để tôi hướng dẫn bạn build SWIM infrastructure từ đầu:

## **SWIM ARCHITECTURE - TECHNICAL DEEP DIVE:**

### **SWIM Core Components:**

#### **1. Service Registry (Như DNS cho services):**
```
Function:
- Store service descriptions (WSDL files)
- Service endpoint URLs  
- Service capabilities & interfaces
- Health status của services

Implementation:
- UDDI (Universal Description, Discovery, Integration)
- Hoặc custom REST-based registry
- Database: PostgreSQL để store service metadata
```

#### **2. Message Router/ESB (Enterprise Service Bus):**
```
Function:
- Route messages between services
- Transform message formats
- Handle retry logic
- Load balancing

Options:
- Apache Camel (Open source)
- Apache ServiceMix
- Mule CE (Community Edition)
- Custom Spring Boot router
```

#### **3. Security Infrastructure:**
```
Components:
- Certificate Authority (CA)
- PKI management
- Authentication service
- Authorization engine

Implementation:
- OpenSSL for CA
- LDAP/Active Directory for users
- OAuth2/JWT for tokens
```

## **SWIM MINIMAL IMPLEMENTATION:**

### **Phase 1: Basic SWIM Node (2-3 tháng)**

#### **Component 1: Service Registry**
```java
// Simple REST-based service registry
@RestController
public class ServiceRegistryController {
    
    @PostMapping("/registry/register")
    public void registerService(@RequestBody ServiceDescriptor service) {
        // Store service info in database
        serviceRepository.save(service);
    }
    
    @GetMapping("/registry/discover/{serviceType}")
    public List<ServiceDescriptor> discoverServices(@PathVariable String serviceType) {
        // Return matching services
        return serviceRepository.findByType(serviceType);
    }
}

// Database schema
CREATE TABLE service_registry (
    id SERIAL PRIMARY KEY,
    service_name VARCHAR(255),
    service_type VARCHAR(100),
    endpoint_url VARCHAR(500),
    wsdl_url VARCHAR(500),
    status VARCHAR(50),
    last_heartbeat TIMESTAMP
);
```

#### **Component 2: Message Router**
```java
// Simple message router với Apache Camel
@Component
public class SWIMMessageRouter extends RouteBuilder {
    
    @Override
    public void configure() throws Exception {
        
        // Route flight plan queries
        from("rest:get:/swim/query/flight/{gufi}")
            .to("bean:serviceDiscovery?method=findQueryService")
            .to("http://query-service/api/flight/${header.gufi}");
            
        // Route filing requests  
        from("rest:post:/swim/filing/submit")
            .to("bean:serviceDiscovery?method=findFilingService")
            .to("http://filing-service/api/submit");
    }
}
```

#### **Component 3: Security Manager**
```java
// Certificate-based authentication
@Component
public class SWIMSecurityManager {
    
    public boolean authenticateClient(X509Certificate clientCert) {
        // Validate certificate against CA
        return certificateValidator.isValid(clientCert);
    }
    
    public boolean authorizeRequest(String clientId, String serviceType) {
        // Check permissions in database
        return permissionRepository.hasAccess(clientId, serviceType);
    }
}
```

## **SWIM DEPLOYMENT ARCHITECTURE:**

### **3-Tier Architecture:**
```
┌─────────────────────────────────────────────────┐
│                 SWIM Layer                      │
├─────────────────────────────────────────────────┤
│  Load Balancer (Nginx)                         │
│  ├── SWIM Gateway 1                            │
│  ├── SWIM Gateway 2                            │
│  └── SWIM Gateway 3                            │
├─────────────────────────────────────────────────┤
│  Service Layer                                  │
│  ├── Service Registry                          │
│  ├── Message Router                            │
│  ├── Security Manager                          │
│  └── Monitoring Service                        │
├─────────────────────────────────────────────────┤
│  Data Layer                                     │
│  ├── Service Registry DB                       │
│  ├── Security Credentials DB                   │
│  ├── Audit Logs DB                             │
│  └── Configuration DB                          │
└─────────────────────────────────────────────────┘
```

## **HARDWARE REQUIREMENTS CHO SWIM:**

### **Production SWIM Infrastructure (4 máy Ubuntu):**

#### **Machine 1: SWIM Gateway & Load Balancer**
```
Ubuntu Server 22.04 LTS
CPU: 8 cores (Intel Xeon hoặc AMD EPYC)
RAM: 32GB DDR4 ECC
Storage: 500GB NVMe SSD  
Network: Dual 10GbE
Role: Nginx LB + SWIM gateways
```

#### **Machine 2: Service Registry & Router**
```
Ubuntu Server 22.04 LTS  
CPU: 6 cores
RAM: 24GB DDR4 ECC
Storage: 1TB SSD
Network: Dual 1GbE
Role: Service discovery + message routing
```

#### **Machine 3: Security & PKI Infrastructure**
```
Ubuntu Server 22.04 LTS
CPU: 4 cores  
RAM: 16GB DDR4 ECC
Storage: 500GB SSD (encrypted)
Network: Dual 1GbE
Role: CA, authentication, certificates
```

#### **Machine 4: Database & Monitoring**
```
Ubuntu Server 22.04 LTS
CPU: 8 cores
RAM: 64GB DDR4 ECC  
Storage: 2TB NVMe SSD + 8TB HDD
Network: Dual 10GbE
Role: PostgreSQL cluster + monitoring
```

### **Development Environment (1 máy):**
```
Ubuntu Desktop 22.04 LTS
CPU: 6-8 cores
RAM: 32GB
Storage: 1TB SSD
Role: Development + testing all components
```

## **SOFTWARE STACK (100% FREE):**

### **SWIM Infrastructure Software:**
```
Operating System: Ubuntu Server 22.04 LTS
Load Balancer: Nginx
Application Runtime: OpenJDK 17
Framework: Spring Boot 3.x
Message Router: Apache Camel 4.x
Service Registry: Custom REST + PostgreSQL
Database: PostgreSQL 15
Monitoring: Prometheus + Grafana
Logging: ELK Stack (Elasticsearch, Logstash, Kibana)
Security: OpenSSL + custom PKI
```

### **Development Tools:**
```
IDE: IntelliJ IDEA Community
Version Control: Git
Build Tool: Maven/Gradle
Testing: JUnit, Postman, SoapUI
Documentation: Swagger/OpenAPI
```

## **IMPLEMENTATION TIMELINE:**

### **Month 1-2: Core Infrastructure**
```
Week 1-2: Setup hardware + Ubuntu installation
Week 3-4: Basic service registry development
Week 5-6: Message router implementation  
Week 7-8: Security infrastructure setup
```

### **Month 3-4: Integration & Testing**
```
Week 9-10: Component integration
Week 11-12: Security testing
Week 13-14: Performance testing
Week 15-16: Documentation + training
```

### **Month 5-6: Production Deployment**
```
Week 17-18: Production environment setup
Week 19-20: Service deployment
Week 21-22: Integration testing với Query Service
Week 23-24: Go-live preparation
```

## **DEVELOPMENT APPROACH:**

### **Start Simple, Scale Up:**

#### **Phase 1: Minimal SWIM (1 machine)**
```
Single Ubuntu server:
- Service registry (REST API)
- Basic message routing
- Simple authentication
- PostgreSQL database

Proof of concept trong 1 tháng
```

#### **Phase 2: Production SWIM (4 machines)**
```
Distribute components:
- High availability
- Load balancing  
- Security hardening
- Performance optimization
```

#### **Phase 3: Advanced Features**
```
- Cross-border connectivity
- Advanced routing rules
- Real-time monitoring
- Disaster recovery
```

## **TESTING STRATEGY:**

### **Unit Testing:**
```
- Service registry operations
- Message routing logic
- Security validation
- Database operations
```

### **Integration Testing:**
```
- End-to-end message flow
- Service discovery workflow
- Certificate authentication
- Error handling scenarios
```

### **Performance Testing:**
```
- Message throughput (target: 1000 msg/sec)
- Service discovery latency
- Database query performance
- Network bandwidth utilization
```

## **BUDGET ESTIMATE:**

### **Hardware (nếu mua mới):**
```
4 production servers: $15K-25K
1 development machine: $3K-5K
Network equipment: $5K-10K
UPS & cooling: $3K-5K

Total hardware: $26K-45K
```

### **Software:**
```
All open source: $0
Professional support (optional): $10K-20K/year
```

### **Human Resources:**
```
SWIM development team: 3-4 engineers × 6 months
Estimated cost: $100K-150K
```

**TOTAL PROJECT: $126K-195K** (so với mua commercial SWIM solution: $500K-1M)

## **NEXT STEPS:**

### **Week 1-2: Planning**
```
1. Finalize hardware procurement
2. Setup development environment  
3. Create detailed technical design
4. Setup project repository
```

### **Week 3-4: Foundation**
```
1. Install Ubuntu on all machines
2. Setup basic networking
3. Create service registry skeleton
4. Setup CI/CD pipeline
```

**Bạn sẵn sàng bắt đầu với approach này không? Tôi có thể detail từng component implementation.**
```mermaid
graph TB
    subgraph "SWIM Infrastructure - VATM Deployment"
        
        subgraph "Machine 1: SWIM Gateway & Load Balancer"
            M1[Ubuntu Server 22.04 LTS<br/>8 cores, 32GB RAM, 500GB SSD<br/>Dual 10GbE]
            M1 --> LB[Nginx Load Balancer<br/>Port 80/443]
            M1 --> GW1[SWIM Gateway 1<br/>Spring Boot App<br/>Port 8081]
            M1 --> GW2[SWIM Gateway 2<br/>Spring Boot App<br/>Port 8082]
            M1 --> GW3[SWIM Gateway 3<br/>Spring Boot App<br/>Port 8083]
        end
        
        subgraph "Machine 2: Service Registry & Router"
            M2[Ubuntu Server 22.04 LTS<br/>6 cores, 24GB RAM, 1TB SSD<br/>Dual 1GbE]
            M2 --> SR[Service Registry<br/>Spring Boot + REST API<br/>Port 8090]
            M2 --> MR[Message Router<br/>Apache Camel<br/>Port 8091]
            M2 --> SD[Service Discovery<br/>UDDI Interface<br/>Port 8092]
        end
        
        subgraph "Machine 3: Security & PKI"
            M3[Ubuntu Server 22.04 LTS<br/>4 cores, 16GB RAM, 500GB SSD<br/>Dual 1GbE]
            M3 --> CA[Certificate Authority<br/>OpenSSL + Custom PKI<br/>Port 443]
            M3 --> AUTH[Authentication Service<br/>OAuth2 + JWT<br/>Port 8093]
            M3 --> AUTHZ[Authorization Engine<br/>RBAC Service<br/>Port 8094]
        end
        
        subgraph "Machine 4: Database & Monitoring"
            M4[Ubuntu Server 22.04 LTS<br/>8 cores, 64GB RAM, 2TB SSD<br/>Dual 10GbE]
            M4 --> PG1[PostgreSQL Primary<br/>Service Registry DB<br/>Port 5432]
            M4 --> PG2[PostgreSQL Replica<br/>Security & Audit DB<br/>Port 5433]
            M4 --> PROM[Prometheus<br/>Metrics Collection<br/>Port 9090]
            M4 --> GRAF[Grafana<br/>Monitoring Dashboard<br/>Port 3000]
            M4 --> ELK[ELK Stack<br/>Elasticsearch + Kibana<br/>Port 9200, 5601]
        end
    end
    
    subgraph "External Connections"
        AIRLINES[Airlines<br/>Vietnam Airlines<br/>VietJet, Bamboo]
        ANSP[Other ANSPs<br/>Thailand, Singapore<br/>Cambodia]
        AIRPORTS[Airports<br/>ACV Systems<br/>Noi Bai, Tan Son Nhat]
        VATM_SVC[VATM Services<br/>Query Service<br/>Filing Service]
    end
    
    subgraph "Development Environment"
        DEV[Ubuntu Desktop 22.04<br/>6-8 cores, 32GB RAM, 1TB SSD<br/>Development Machine]
        DEV --> IDE[IntelliJ IDEA Community<br/>Java Development]
        DEV --> DOCKER[Docker<br/>Local Testing]
        DEV --> GIT[Git Repository<br/>Version Control]
        DEV --> TEST[Testing Tools<br/>Postman, SoapUI]
    end
    
    %% Network Connections
    AIRLINES -.->|HTTPS/SOAP| LB
    ANSP -.->|HTTPS/REST| LB
    AIRPORTS -.->|HTTPS/JSON| LB
    
    LB --> GW1
    LB --> GW2
    LB --> GW3
    
    GW1 -.->|Service Lookup| SR
    GW2 -.->|Service Lookup| SR
    GW3 -.->|Service Lookup| SR
    
    GW1 -.->|Message Routing| MR
    GW2 -.->|Message Routing| MR
    GW3 -.->|Message Routing| MR
    
    GW1 -.->|Authentication| AUTH
    GW2 -.->|Authentication| AUTH
    GW3 -.->|Authentication| AUTH
    
    MR -.->|Route to Services| VATM_SVC
    
    SR -.->|Database| PG1
    AUTH -.->|Database| PG2
    MR -.->|Database| PG1
    
    PG1 -.->|Replication| PG2
    
    PROM -.->|Collect Metrics| M1
    PROM -.->|Collect Metrics| M2
    PROM -.->|Collect Metrics| M3
    GRAF -.->|Query Metrics| PROM
    
    %% Development connections
    DEV -.->|SSH/Deploy| M1
    DEV -.->|SSH/Deploy| M2
    DEV -.->|SSH/Deploy| M3
    DEV -.->|SSH/Deploy| M4
    
    %% Styling
    classDef machine fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef service fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef database fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef external fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef dev fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    
    class M1,M2,M3,M4 machine
    class LB,GW1,GW2,GW3,SR,MR,SD,CA,AUTH,AUTHZ,PROM,GRAF,ELK service
    class PG1,PG2 database
    class AIRLINES,ANSP,AIRPORTS,VATM_SVC external
    class DEV,IDE,DOCKER,GIT,TEST dev
```

## **SOFTWARE DETAILS CHO TỪNG MÁY:**

### **Machine 1: SWIM Gateway & Load Balancer**
```bash
# Services running:
sudo systemctl status nginx          # Load balancer
sudo systemctl status swim-gateway-1 # Spring Boot app
sudo systemctl status swim-gateway-2 # Spring Boot app  
sudo systemctl status swim-gateway-3 # Spring Boot app

# Ports:
netstat -tlnp | grep :80    # Nginx HTTP
netstat -tlnp | grep :443   # Nginx HTTPS
netstat -tlnp | grep :8081  # Gateway 1
netstat -tlnp | grep :8082  # Gateway 2
netstat -tlnp | grep :8083  # Gateway 3
```

### **Machine 2: Service Registry & Router**
```bash
# Services running:
sudo systemctl status service-registry  # REST API
sudo systemctl status message-router    # Apache Camel
sudo systemctl status service-discovery # UDDI

# Ports:
netstat -tlnp | grep :8090  # Service Registry
netstat -tlnp | grep :8091  # Message Router
netstat -tlnp | grep :8092  # Service Discovery
```

### **Machine 3: Security & PKI**
```bash
# Services running:
sudo systemctl status openssl-ca        # Certificate Authority
sudo systemctl status oauth2-server     # Authentication
sudo systemctl status authz-engine      # Authorization

# Ports:
netstat -tlnp | grep :443   # CA HTTPS
netstat -tlnp | grep :8093  # OAuth2 server
netstat -tlnp | grep :8094  # Authorization engine
```

### **Machine 4: Database & Monitoring**
```bash
# Services running:
sudo systemctl status postgresql         # Primary DB
sudo systemctl status postgresql@replica # Replica DB
sudo systemctl status prometheus         # Metrics
sudo systemctl status grafana-server     # Dashboard
sudo systemctl status elasticsearch      # Log search
sudo systemctl status kibana             # Log visualization

# Ports:
netstat -tlnp | grep :5432  # PostgreSQL primary
netstat -tlnp | grep :5433  # PostgreSQL replica
netstat -tlnp | grep :9090  # Prometheus
netstat -tlnp | grep :3000  # Grafana
netstat -tlnp | grep :9200  # Elasticsearch
netstat -tlnp | grep :5601  # Kibana
```

## **NETWORK FLOW:**

```mermaid
sequenceDiagram
    participant A as Airline
    participant LB as Load Balancer
    participant GW as SWIM Gateway
    participant SR as Service Registry
    participant MR as Message Router
    participant QS as Query Service
    participant DB as Database

    A->>LB: HTTPS Request (eFPL Query)
    LB->>GW: Forward to Gateway
    GW->>SR: Discover Query Service
    SR->>DB: Lookup service endpoint
    DB-->>SR: Return endpoint URL
    SR-->>GW: Service endpoint
    GW->>MR: Route message
    MR->>QS: Forward query request
    QS->>DB: Execute query
    DB-->>QS: Return flight data
    QS-->>MR: eFPL response
    MR-->>GW: Route response back
    GW-->>LB: FIXM XML response
    LB-->>A: Return flight data
```

## **INSTALLATION COMMANDS:**

### **Machine 1 Setup:**
```bash
# Install Nginx
sudo apt update
sudo apt install nginx -y

# Install Java
sudo apt install openjdk-17-jdk -y

# Setup SWIM Gateway applications
mkdir -p /opt/swim/gateway
# Deploy Spring Boot JARs
sudo systemctl enable swim-gateway-1
sudo systemctl enable swim-gateway-2
sudo systemctl enable swim-gateway-3
```

### **Machine 2 Setup:**
```bash
# Install Java and Maven
sudo apt install openjdk-17-jdk maven -y

# Install Apache Camel
mkdir -p /opt/swim/router
# Deploy message router application
sudo systemctl enable service-registry
sudo systemctl enable message-router
```

### **Machine 3 Setup:**
```bash
# Install OpenSSL and security tools
sudo apt install openssl ca-certificates -y

# Setup PKI infrastructure
mkdir -p /opt/swim/pki/{ca,certs,private}
# Configure CA and security services
sudo systemctl enable openssl-ca
sudo systemctl enable oauth2-server
```

### **Machine 4 Setup:**
```bash
# Install PostgreSQL
sudo apt install postgresql-15 postgresql-client-15 -y

# Install monitoring stack
sudo apt install prometheus grafana -y

# Install ELK stack
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt install elasticsearch kibana -y
```

**Bạn muốn tôi detail implementation code cho component nào trước?**
Tuyệt! Để tôi bổ sung Query Service vào SWIM architecture:

```mermaid
graph TB
    subgraph "SWIM Infrastructure - VATM Deployment"
        
        subgraph "Machine 1: SWIM Gateway & Load Balancer"
            M1[Ubuntu Server 22.04 LTS<br/>8 cores, 32GB RAM, 500GB SSD<br/>Dual 10GbE]
            M1 --> LB[Nginx Load Balancer<br/>Port 80/443]
            M1 --> GW1[SWIM Gateway 1<br/>Spring Boot App<br/>Port 8081]
            M1 --> GW2[SWIM Gateway 2<br/>Spring Boot App<br/>Port 8082]
            M1 --> GW3[SWIM Gateway 3<br/>Spring Boot App<br/>Port 8083]
        end
        
        subgraph "Machine 2: Service Registry & Router"
            M2[Ubuntu Server 22.04 LTS<br/>6 cores, 24GB RAM, 1TB SSD<br/>Dual 1GbE]
            M2 --> SR[Service Registry<br/>Spring Boot + REST API<br/>Port 8090]
            M2 --> MR[Message Router<br/>Apache Camel<br/>Port 8091]
            M2 --> SD[Service Discovery<br/>UDDI Interface<br/>Port 8092]
        end
        
        subgraph "Machine 3: Security & PKI"
            M3[Ubuntu Server 22.04 LTS<br/>4 cores, 16GB RAM, 500GB SSD<br/>Dual 1GbE]
            M3 --> CA[Certificate Authority<br/>OpenSSL + Custom PKI<br/>Port 443]
            M3 --> AUTH[Authentication Service<br/>OAuth2 + JWT<br/>Port 8093]
            M3 --> AUTHZ[Authorization Engine<br/>RBAC Service<br/>Port 8094]
        end
        
        subgraph "Machine 4: Database & Monitoring"
            M4[Ubuntu Server 22.04 LTS<br/>8 cores, 64GB RAM, 2TB SSD<br/>Dual 10GbE]
            M4 --> PG1[PostgreSQL Primary<br/>Service Registry DB<br/>Port 5432]
            M4 --> PG2[PostgreSQL Replica<br/>Security & Audit DB<br/>Port 5433]
            M4 --> PROM[Prometheus<br/>Metrics Collection<br/>Port 9090]
            M4 --> GRAF[Grafana<br/>Monitoring Dashboard<br/>Port 3000]
            M4 --> ELK[ELK Stack<br/>Elasticsearch + Kibana<br/>Port 9200, 5601]
        end
    end
    
    subgraph "Machine 5: VATM Services - NEW!"
        M5[Ubuntu Server 22.04 LTS<br/>6 cores, 24GB RAM, 1TB SSD<br/>Dual 1GbE]
        M5 --> QS[Query Service<br/>Spring Boot Application<br/>Port 8100]
        M5 --> FS[Filing Service<br/>Spring Boot Application<br/>Port 8101]
        M5 --> QSDB[Flight Plans Database<br/>PostgreSQL<br/>Port 5434]
        M5 --> ADAPTER[SWIM Adapter<br/>FIXM Message Handler<br/>Port 8102]
    end
    
    subgraph "External Connections"
        AIRLINES[Airlines<br/>Vietnam Airlines<br/>VietJet, Bamboo]
        ANSP[Other ANSPs<br/>Thailand, Singapore<br/>Cambodia]
        AIRPORTS[Airports<br/>ACV Systems<br/>Noi Bai, Tan Son Nhat]
        LEGACY[VATM Legacy Systems<br/>FDPS, TopSky-ATC<br/>Radar Systems]
    end
    
    %% Network Connections
    AIRLINES -.->|HTTPS Query Request| LB
    ANSP -.->|HTTPS Query Request| LB
    AIRPORTS -.->|HTTPS Query Request| LB
    
    LB --> GW1
    LB --> GW2
    LB --> GW3
    
    GW1 -.->|Service Lookup| SR
    GW2 -.->|Service Lookup| SR
    GW3 -.->|Service Lookup| SR
    
    GW1 -.->|Message Routing| MR
    GW2 -.->|Message Routing| MR
    GW3 -.->|Message Routing| MR
    
    MR -.->|Route Query Requests| ADAPTER
    MR -.->|Route Filing Requests| ADAPTER
    
    ADAPTER -.->|Forward Query| QS
    ADAPTER -.->|Forward Filing| FS
    
    QS -.->|Database Access| QSDB
    FS -.->|Database Access| QSDB
    
    %% Service Registration
    QS -.->|Register Service| SR
    FS -.->|Register Service| SR
    
    %% Authentication
    QS -.->|Authenticate| AUTH
    FS -.->|Authenticate| AUTH
    
    %% Legacy Integration
    FS -.->|Sync Flight Data| LEGACY
    LEGACY -.->|Push Updates| FS
    
    %% Monitoring
    PROM -.->|Collect Metrics| M5
    
    %% Database connections
    SR -.->|Service Registry DB| PG1
    AUTH -.->|Security DB| PG2
    
    %% Styling
    classDef machine fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef service fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef database fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef external fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef vatm fill:#ffebee,stroke:#c62828,stroke-width:3px
    
    class M1,M2,M3,M4 machine
    class M5 vatm
    class LB,GW1,GW2,GW3,SR,MR,SD,CA,AUTH,AUTHZ,PROM,GRAF,ELK service
    class QS,FS,ADAPTER vatm
    class PG1,PG2,QSDB database
    class AIRLINES,ANSP,AIRPORTS,LEGACY external
```

## **VỊ TRÍ QUERY SERVICE TRONG SWIM:**

### **Query Service là "Business Logic Layer":**
```
SWIM Infrastructure (Machines 1-4):
├── Transport & Security (gateways, auth)
├── Service Discovery & Routing
└── Monitoring & Management

VATM Services (Machine 5):
├── Query Service ← ĐÂY LÀ CORE BUSINESS LOGIC
├── Filing Service  
└── SWIM Adapter ← Interface với SWIM
```

### **Message Flow cho Query Service:**

```mermaid
sequenceDiagram
    participant A as Airline
    participant LB as Load Balancer
    participant GW as SWIM Gateway
    participant SR as Service Registry
    participant MR as Message Router
    participant SA as SWIM Adapter
    participant QS as Query Service
    participant DB as Flight DB

    Note over A,DB: Query Flight Information Flow
    
    A->>LB: Query Request<br/>(GUFI: VN123-20250912-001)
    LB->>GW: Route to Gateway
    
    Note over GW: Authentication & Authorization
    GW->>SR: Lookup "FlightQueryService"
    SR-->>GW: Return: Machine5:8100
    
    GW->>MR: Route message to Query Service
    MR->>SA: Forward query (FIXM format)
    
    Note over SA: FIXM Message Processing
    SA->>QS: Convert to internal format
    QS->>DB: SELECT * FROM flight_plans WHERE gufi='VN123-20250912-001'
    DB-->>QS: Return flight data
    QS-->>SA: Internal flight object
    
    Note over SA: Response Formatting
    SA-->>MR: Convert to FIXM XML
    MR-->>GW: Route response back
    GW-->>LB: FIXM response
    LB-->>A: Flight plan data (XML)
```

## **QUERY SERVICE CHI TIẾT:**

### **Machine 5 Configuration:**
```bash
# Services running on Machine 5:
sudo systemctl status query-service     # Port 8100
sudo systemctl status filing-service    # Port 8101  
sudo systemctl status swim-adapter      # Port 8102
sudo systemctl status postgresql        # Port 5434

# Directory structure:
/opt/vatm/
├── query-service/
│   ├── query-service.jar
│   ├── application.yml
│   └── logs/
├── filing-service/
│   ├── filing-service.jar
│   └── logs/
├── swim-adapter/
│   ├── swim-adapter.jar
│   └── fixm-schemas/
└── database/
    └── flight-plans/
```

### **Query Service Code Structure:**
```java
// Main Application
@SpringBootApplication
@EnableJpaRepositories
public class QueryServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(QueryServiceApplication.class, args);
    }
}

// REST Controller - Internal API  
@RestController
@RequestMapping("/api/query")
public class QueryController {
    
    @GetMapping("/flight/{gufi}")
    public ResponseEntity<FlightPlan> getFlightByGufi(@PathVariable String gufi) {
        FlightPlan flight = queryService.findByGufi(gufi);
        return ResponseEntity.ok(flight);
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<FlightPlan>> searchFlights(
            @RequestParam String flightNumber,
            @RequestParam LocalDate date) {
        List<FlightPlan> flights = queryService.searchFlights(flightNumber, date);
        return ResponseEntity.ok(flights);
    }
}
```

### **SWIM Adapter Code:**
```java
// SWIM Interface - FIXM Handler
@RestController
@RequestMapping("/swim")
public class SWIMAdapterController {
    
    @Autowired
    private QueryServiceClient queryServiceClient;
    
    @Autowired  
    private FIXMConverter fixmConverter;
    
    @PostMapping("/query")
    public ResponseEntity<String> handleSWIMQuery(@RequestBody String fixmQuery) {
        try {
            // 1. Parse FIXM query
            QueryRequest request = fixmConverter.parseQuery(fixmQuery);
            
            // 2. Call internal Query Service
            FlightPlan flight = queryServiceClient.getFlightByGufi(request.getGufi());
            
            // 3. Convert to FIXM response
            String fixmResponse = fixmConverter.toFIXM(flight);
            
            return ResponseEntity.ok(fixmResponse);
            
        } catch (Exception e) {
            return ResponseEntity.badRequest().body("Error: " + e.getMessage());
        }
    }
}
```

## **SERVICE REGISTRATION:**

### **Query Service tự đăng ký với SWIM:**
```java
@Component
public class ServiceRegistration {
    
    @Value("${swim.registry.url}")
    private String registryUrl;
    
    @PostConstruct
    public void registerWithSWIM() {
        ServiceDescriptor descriptor = new ServiceDescriptor();
        descriptor.setServiceName("VATM-FlightQueryService");
        descriptor.setServiceType("FF-ICE-Query");
        descriptor.setEndpointUrl("http://machine5:8102/swim/query");
        descriptor.setWsdlUrl("http://machine5:8102/swim/query?wsdl");
        descriptor.setProvider("VATM");
        descriptor.setVersion("1.0");
        
        // Register với Service Registry
        restTemplate.postForObject(registryUrl + "/register", descriptor, String.class);
    }
}
```

## **DATABASE SCHEMA CHO QUERY SERVICE:**

```sql
-- Machine 5 PostgreSQL Database
CREATE DATABASE vatm_flight_plans;

CREATE TABLE flight_plans (
    id SERIAL PRIMARY KEY,
    gufi VARCHAR(50) UNIQUE NOT NULL,
    flight_number VARCHAR(10) NOT NULL,
    aircraft_type VARCHAR(10),
    departure_airport VARCHAR(4),
    arrival_airport VARCHAR(4),
    departure_time TIMESTAMP,
    arrival_time TIMESTAMP,
    route TEXT,
    altitude INTEGER,
    speed INTEGER,
    fixm_data JSONB,  -- Store complete FIXM as JSON
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for fast querying
CREATE INDEX idx_gufi ON flight_plans(gufi);
CREATE INDEX idx_flight_number_date ON flight_plans(flight_number, DATE(departure_time));
CREATE INDEX idx_route ON flight_plans(departure_airport, arrival_airport);
CREATE INDEX idx_departure_time ON flight_plans(departure_time);
```

## **CONFIGURATION FILES:**

### **Query Service application.yml:**
```yaml
server:
  port: 8100

spring:
  datasource:
    url: jdbc:postgresql://localhost:5434/vatm_flight_plans
    username: query_user
    password: secure_password
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false

swim:
  registry:
    url: http://machine2:8090/registry
  adapter:
    url: http://localhost:8102
  
logging:
  level:
    com.vatm.queryservice: DEBUG
```

**TÓM LẠI VỊ TRÍ QUERY SERVICE:**
- ✅ **Machine 5**: Riêng biệt với SWIM infrastructure
- ✅ **Port 8100**: Internal business logic API
- ✅ **SWIM Adapter**: Interface với SWIM (port 8102)  
- ✅ **Auto-registration**: Tự đăng ký với Service Registry
- ✅ **Independent**: Có thể develop/test độc lập

**Query Service chạy như 1 microservice riêng, connect tới SWIM qua standardized interfaces!**
