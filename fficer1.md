```bash
Hãy đứng vai trò là chuyên gia đặc tả phần mềm giúp tôi xây dựng mục tiêu và yêu cầu cho team để xây dựng Filing Service, Query Service để team nắm được mục tiêu, yêu cầu, tiêu chuẩn cần tuân thủ để xây dựng hoàn thiện 2 chức năng bắt buộc trong ffice
```

# **TECHNICAL SPECIFICATION: FF-ICE MANDATORY SERVICES**
## **For VATM Development Team**

---

## **1. PROJECT OVERVIEW**

### **1.1 Objective**
Implement two mandatory FF-ICE services to enable VATM transition from traditional ANSP (aASP) to enhanced ANSP (eASP), supporting next-generation collaborative flight planning and real-time flight data exchange.

### **1.2 Business Goals**
- **Primary**: Enable VATM compliance with ICAO FF-ICE/R1 standards
- **Secondary**: Provide modern flight data services to airlines and regional ANSPs
- **Success Metrics**: 
  - 99.5% service availability
  - <5 second query response time
  - Support 500+ daily flight operations
  - Regional ANSP integration capability

### **1.3 Scope**
```
✅ IN SCOPE:
- FF-ICE Filing Service (mandatory)
- FF-ICE Flight Plan Query Service (mandatory)
- FIXM 4.2.0 message processing
- SWIM-compliant interfaces
- Basic security and authentication

❌ OUT OF SCOPE:
- Optional FF-ICE services (Update, Cancel, Trial, Publication)
- Legacy AMHS integration
- Advanced 4D trajectory optimization
- Multi-language user interfaces
```

---

## **2. SERVICE SPECIFICATIONS**

### **2.1 Filing Service Requirements**

#### **2.1.1 Functional Requirements**
| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|-------------------|
| FS-F01 | Accept eFPL submissions | **CRITICAL** | Accept valid FIXM 4.2.0 eFPL via HTTPS POST |
| FS-F02 | Validate eFPL format | **CRITICAL** | Reject invalid FIXM with specific error codes |
| FS-F03 | Generate GUFI | **CRITICAL** | Create globally unique flight identifier |
| FS-F04 | Store flight plans | **CRITICAL** | Persist eFPL data in database |
| FS-F05 | Return filing status | **CRITICAL** | Response: ACCEPTED/REJECTED/PENDING |
| FS-F06 | Business rule validation | **HIGH** | Check Vietnamese airspace rules |
| FS-F07 | Duplicate detection | **HIGH** | Prevent duplicate GUFI submissions |

#### **2.1.2 Technical Requirements**
```python
# API Specification
POST /swim/filing/submit
Content-Type: application/xml
Authorization: Bearer <JWT_TOKEN>

Request Body: FIXM 4.2.0 eFPL XML
Response: 
{
    "status": "ACCEPTED|REJECTED|PENDING",
    "gufi": "VN123-20250315-001",
    "timestamp": "2025-03-15T08:00:00Z",
    "errors": [...] // if REJECTED
}
```

#### **2.1.3 Validation Rules**
```yaml
Mandatory Fields:
  - gufi: Pattern "[A-Z0-9]{3,8}-[0-9]{8}-[0-9]{3}"
  - flightNumber: 2-8 characters
  - aircraftType: Valid ICAO aircraft code
  - departure.aerodrome: Valid ICAO airport code
  - arrival.aerodrome: Valid ICAO airport code
  - departureTime: ISO 8601 format

Vietnamese Specific Rules:
  - VVNB, VVTS, VVDN must be handled
  - Flight level restrictions in Vietnamese airspace
  - Military coordination requirements
  - Slot validation for major airports
```

### **2.2 Query Service Requirements**

#### **2.2.1 Functional Requirements**
| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|-------------------|
| QS-F01 | Query by GUFI | **CRITICAL** | Return complete eFPL for valid GUFI |
| QS-F02 | Query by flight number | **CRITICAL** | Return flights matching callsign + date |
| QS-F03 | Query by route | **HIGH** | Return flights on specified route |
| QS-F04 | Query by time range | **HIGH** | Return flights in time window |
| QS-F05 | Handle not found | **CRITICAL** | Return appropriate error for missing flights |
| QS-F06 | Return FIXM format | **CRITICAL** | Response in valid FIXM 4.2.0 XML |
| QS-F07 | Support filtering | **MEDIUM** | Filter by status, aircraft type, etc. |

#### **2.2.2 API Specifications**
```python
# Query by GUFI
GET /swim/query/flight/{gufi}
Response: FIXM 4.2.0 XML or 404 error

# Search flights
GET /swim/query/search?flight_number=VN123&date=2025-03-15
Response: Array of FIXM flight plans

# Query by route
GET /swim/query/route?departure=VVNB&arrival=VVTS&date=2025-03-15
Response: Array of matching flights
```

---

## **3. TECHNICAL STANDARDS COMPLIANCE**

### **3.1 ICAO Standards**
- **ICAO Doc 9965**: FF-ICE Implementation Guidelines
- **FIXM 4.2.0**: Flight Information Exchange Model
- **Service interface**: RESTful HTTP/HTTPS
- **Message format**: XML with UTF-8 encoding
- **Error handling**: Standardized error codes (FPF001-FPF999, FPQ001-FPQ999)

### **3.2 EUROCONTROL SWIM Standards**
- **Transport**: HTTPS with TLS 1.3 minimum
- **Authentication**: X.509 certificate-based
- **Service discovery**: SWIM registry integration
- **Message size**: Maximum 2MB per eFPL
- **Timeout**: 30 seconds maximum response time

### **3.3 Data Formats**
```xml
<!-- Example FIXM eFPL Structure -->
<FlightPlan xmlns="http://www.fixm.aero/flight/4.2"
             xmlns:fb="http://www.fixm.aero/base/4.2">
  <gufi>VN123-20250315-001</gufi>
  <flight>
    <flightNumber>VN123</flightNumber>
    <aircraft>
      <aircraftType>A321</aircraftType>
      <registration>VN-A123</registration>
    </aircraft>
    <departure>
      <aerodrome>VVNB</aerodrome>
      <scheduledTimeOfDeparture>2025-03-15T08:00:00Z</scheduledTimeOfDeparture>
    </departure>
    <arrival>
      <aerodrome>VVTS</aerodrome>
      <estimatedTimeOfArrival>2025-03-15T10:30:00Z</estimatedTimeOfArrival>
    </arrival>
    <routeTrajectory>
      <element>
        <elementStartPoint>
          <position>
            <lat>21.221944</lat>
            <lon>105.807222</lon>
          </position>
        </elementStartPoint>
      </element>
    </routeTrajectory>
  </flight>
</FlightPlan>
```

---

## **4. TECHNOLOGY STACK REQUIREMENTS**

### **4.1 Backend Development**
```python
# Required Technologies
Language: Python 3.11+
Framework: FastAPI 0.104+
Database: PostgreSQL 15+
Cache: Redis 7+
XML Processing: lxml 4.9+
Authentication: JWT + X.509 certificates
Testing: pytest 7.4+

# Project Structure
swim-services/
├── app/
│   ├── api/
│   │   ├── filing_service.py
│   │   └── query_service.py
│   ├── core/
│   │   ├── fixm_processor.py
│   │   ├── validation.py
│   │   └── security.py
│   ├── models/
│   │   └── flight_plan.py
│   └── tests/
├── schemas/
│   └── fixm-4.2.0/
└── docker-compose.yml
```

### **4.2 Database Schema**
```sql
-- Core tables
CREATE TABLE flight_plans (
    id SERIAL PRIMARY KEY,
    gufi VARCHAR(20) UNIQUE NOT NULL,
    flight_number VARCHAR(10) NOT NULL,
    aircraft_type VARCHAR(10),
    registration VARCHAR(15),
    departure_airport VARCHAR(4) NOT NULL,
    arrival_airport VARCHAR(4) NOT NULL,
    departure_time TIMESTAMP WITH TIME ZONE NOT NULL,
    arrival_time TIMESTAMP WITH TIME ZONE,
    route_text TEXT,
    fixm_data JSONB NOT NULL,  -- Complete FIXM as JSON
    filing_status VARCHAR(20) DEFAULT 'ACCEPTED',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_gufi ON flight_plans(gufi);
CREATE INDEX idx_flight_number_date ON flight_plans(flight_number, DATE(departure_time));
CREATE INDEX idx_route ON flight_plans(departure_airport, arrival_airport);
CREATE INDEX idx_departure_time ON flight_plans(departure_time);
```

---

## **5. PERFORMANCE REQUIREMENTS**

### **5.1 Response Time**
| Service | Operation | Target | Maximum |
|---------|-----------|--------|---------|
| Filing Service | Submit eFPL | <2 seconds | 5 seconds |
| Query Service | Query by GUFI | <1 second | 3 seconds |
| Query Service | Search flights | <3 seconds | 5 seconds |

### **5.2 Throughput**
- **Filing Service**: 50 submissions/minute peak
- **Query Service**: 200 queries/minute peak
- **Concurrent users**: 100 simultaneous connections
- **Daily volume**: 1,000 flight plans

### **5.3 Availability**
- **Uptime**: 99.5% (43.8 hours downtime/year maximum)
- **Planned maintenance**: Max 4 hours/month
- **Recovery time**: <15 minutes for service restart

---

## **6. SECURITY REQUIREMENTS**

### **6.1 Authentication & Authorization**
```python
# Certificate-based authentication
class SecurityRequirements:
    CERTIFICATE_TYPE = "X.509 v3"
    MIN_KEY_SIZE = 2048  # RSA
    SIGNATURE_ALGORITHM = "SHA-256"
    
    # Authorized entities
    AUTHORIZED_CLIENTS = [
        "Vietnam Airlines",
        "VietJet Air", 
        "Bamboo Airways",
        "Regional ANSPs"
    ]
```

### **6.2 Data Protection**
- **Transport encryption**: TLS 1.3 minimum
- **Data at rest**: PostgreSQL encryption
- **Audit logging**: All API calls logged
- **Data retention**: 7 years for flight plans
- **PII handling**: Minimal personal data storage

### **6.3 Input Validation**
```python
# Security validations
def validate_efpl_security(efpl_xml):
    # XML bomb protection
    if len(efpl_xml) > 2_000_000:  # 2MB limit
        raise SecurityError("eFPL too large")
    
    # XXE injection prevention
    parser = etree.XMLParser(resolve_entities=False)
    
    # SQL injection prevention
    # Use parameterized queries only
    
    # XSS prevention in error messages
    # Escape all user input in responses
```

---

## **7. ERROR HANDLING SPECIFICATION**

### **7.1 Error Codes**
```python
# Filing Service errors
FILING_ERRORS = {
    "FPF001": "Invalid FIXM format",
    "FPF002": "Missing mandatory fields", 
    "FPF003": "Invalid GUFI format",
    "FPF004": "Duplicate flight plan",
    "FPF005": "Invalid aircraft type",
    "FPF006": "Invalid airport codes",
    "FPF007": "Business rule violation",
    "FPF008": "Service temporarily unavailable"
}

# Query Service errors  
QUERY_ERRORS = {
    "FPQ001": "Invalid GUFI format",
    "FPQ002": "Flight not found",
    "FPQ003": "Invalid search parameters",
    "FPQ004": "Query timeout",
    "FPQ005": "Service temporarily unavailable"
}
```

### **7.2 Error Response Format**
```json
{
    "error": {
        "code": "FPF002",
        "message": "Missing mandatory field: aircraft.aircraftType", 
        "timestamp": "2025-03-15T08:00:00Z",
        "correlationId": "req-12345",
        "details": {
            "field": "aircraft.aircraftType",
            "expected": "ICAO aircraft type code",
            "received": null
        }
    }
}
```

---

## **8. TESTING REQUIREMENTS**

### **8.1 Unit Testing**
- **Coverage**: Minimum 90% code coverage
- **Framework**: pytest with fixtures
- **Scope**: All business logic, validation, and utilities

### **8.2 Integration Testing**
```python
# Test scenarios
class IntegrationTests:
    def test_filing_to_query_flow():
        # 1. Submit eFPL via Filing Service
        # 2. Verify storage in database
        # 3. Query same flight via Query Service
        # 4. Verify data consistency
        
    def test_fixm_validation():
        # Test with various FIXM samples
        # Test error cases and edge conditions
        
    def test_concurrent_operations():
        # Multiple simultaneous submissions
        # Query during filing operations
```

### **8.3 Performance Testing**
- **Load testing**: 200 concurrent users
- **Stress testing**: 500 requests/minute
- **Endurance testing**: 24-hour continuous operation

---

## **9. DEPLOYMENT REQUIREMENTS**

### **9.1 Environment Setup**
```yaml
# Production environment
OS: Ubuntu 22.04 LTS
Python: 3.11.x
Database: PostgreSQL 15.x
Cache: Redis 7.x  
Web Server: Uvicorn + Nginx
Monitoring: Prometheus + Grafana
```

### **9.2 Configuration Management**
```python
# Environment-specific configs
class Settings(BaseSettings):
    DATABASE_URL: str
    REDIS_URL: str
    SWIM_REGISTRY_URL: str
    LOG_LEVEL: str = "INFO"
    CERTIFICATE_PATH: str
    PRIVATE_KEY_PATH: str
    
    class Config:
        env_file = ".env"
```

---

## **10. ACCEPTANCE CRITERIA**

### **10.1 Functional Acceptance**
- [ ] Filing Service accepts valid FIXM eFPL
- [ ] Filing Service rejects invalid eFPL with correct error codes
- [ ] Query Service returns flight data by GUFI
- [ ] Query Service handles "not found" cases properly
- [ ] Both services integrate with SWIM registry
- [ ] Authentication via X.509 certificates working
- [ ] All mandatory fields validated correctly

### **10.2 Performance Acceptance**
- [ ] Filing response time <2 seconds (95th percentile)
- [ ] Query response time <1 second (95th percentile)  
- [ ] System handles 50 concurrent users
- [ ] 99.5% uptime over 30-day period
- [ ] Database queries optimized (<100ms average)

### **10.3 Security Acceptance**
- [ ] TLS 1.3 enforced for all connections
- [ ] Certificate validation working correctly
- [ ] Input validation prevents injection attacks
- [ ] Audit logs capture all operations
- [ ] Error messages don't expose sensitive data

---

## **11. DELIVERY TIMELINE**

### **Phase 1: Foundation (Weeks 1-4)**
- Project setup and environment configuration
- Database schema implementation
- Basic FIXM processing capabilities
- Authentication framework

### **Phase 2: Core Services (Weeks 5-8)**
- Filing Service implementation
- Query Service implementation  
- Validation and error handling
- Unit testing

### **Phase 3: Integration (Weeks 9-12)**
- SWIM registry integration
- Security implementation
- Integration testing
- Performance optimization

### **Phase 4: Deployment (Weeks 13-16)**
- Production deployment
- User acceptance testing
- Documentation completion
- Go-live preparation

---

**TEAM RESPONSIBILITIES:**
- **Lead Developer**: Architecture, SWIM integration, security
- **Backend Developer**: Services implementation, database design
- **QA Engineer**: Testing strategy, test automation
- **DevOps Engineer**: Deployment, monitoring, performance tuning

**SUCCESS METRICS:**
- Services pass all acceptance criteria
- Performance requirements met
- Security audit completed successfully
- Integration with EUROCONTROL SWIM registry achieved
- Ready for airline onboarding

---
## **GLOSSARY - FF-ICE TERMS EXPLAINED**

### **ANSP (Air Navigation Service Provider)**
```
Đơn giản: Tổ chức quản lý không lưu
Ví dụ: VATM ở Việt Nam, AEROTHAI ở Thái Lan
Nhiệm vụ: Điều khiển máy bay, cấp phép bay, quản lý không phận
```

### **aASP vs eASP**
```
aASP (traditional ANSP): 
- Hệ thống cũ, dùng tin nhắn text đơn giản
- Ví dụ: VATM hiện tại với AMHS

eASP (enhanced ANSP):
- Hệ thống mới, dùng dữ liệu phong phú
- Ví dụ: VATM sau khi có FF-ICE + SWIM
- Có thể chia sẻ thông tin chi tiết hơn với airlines
```

### **Regional ANSPs**
```
Các ANSP láng giềng cần phối hợp:
- Thailand: AEROTHAI  
- Singapore: CAAS
- Cambodia: SSCA
- Malaysia: CAAM

Tại sao quan trọng: Máy bay bay qua nhiều nước
```

### **eFPL (extended Flight Plan)**
```
Trước: Flight plan đơn giản
"VN123 từ HAN đi SGN lúc 8:00"

Sau: eFPL chi tiết  
"VN123, A321, từ HAN đi SGN, 8:00 xuất phát, 
đường bay qua 50 điểm, thời gian tại từng điểm,
độ cao, tốc độ, nhiên liệu, thời tiết..."
```

### **eFPL Format**
```
XML format, rất dài và chi tiết:
<FlightPlan>
  <gufi>VN123-20250315-001</gufi>
  <aircraft>A321</aircraft>
  <route>
    <point1>HAN at 08:00</point1>
    <point2>REPOX at 08:15</point2>
    <point3>SGN at 10:00</point3>
  </route>
</FlightPlan>
```

### **Generate GUFI (Globally Unique Flight Identifier)**
```
GUFI = ID duy nhất cho mỗi chuyến bay trên toàn thế giới

Format: ABC123-20250315-001
- ABC123: Flight number
- 20250315: Date (15 March 2025)  
- 001: Sequence number (nếu có nhiều chuyến cùng ngày)

Tại sao cần: Phân biệt VN123 hôm nay vs VN123 ngày mai
```

### **FIXM Format**
```
FIXM = Flight Information Exchange Model
= Chuẩn định dạng dữ liệu bay của ICAO

Giống như:
- JSON cho web development
- XML cho flight data  

Đảm bảo: Mọi nước hiểu cùng 1 format dữ liệu
```

### **X.509 Certificate-based Authentication**
```
Thay vì username/password, dùng certificate

Giống như:
- Chứng minh thư cá nhân
- Passport khi đi máy bay

Hoạt động:
1. VATM cấp certificate cho Vietnam Airlines
2. Vietnam Airlines dùng certificate để chứng minh danh tính
3. VATM verify certificate trước khi cho truy cập dịch vụ

Lợi ích: An toàn hơn, khó fake hơn
```

### **PII Handling (Personal Identifiable Information)**
```
PII = Thông tin cá nhân

Trong flight plan:
✅ Thông tin máy bay: OK để lưu trữ
✅ Đường bay: OK để chia sẻ  
❌ Danh sách hành khách: KHÔNG được lưu
❌ Passport numbers: KHÔNG được chia sẻ

Quy tắc: Chỉ lưu thông tin cần thiết cho điều hành bay
```

## **TÓM TẮT NHANH:**

```
ANSP = Công ty quản lý không lưu
aASP = ANSP dùng công nghệ cũ  
eASP = ANSP dùng công nghệ mới
Regional ANSPs = Các ANSP láng giềng
eFPL = Kế hoạch bay siêu chi tiết
GUFI = ID duy nhất cho chuyến bay
FIXM = Chuẩn format dữ liệu bay
X.509 = Chứng chỉ bảo mật thay password
PII = Thông tin cá nhân cần bảo vệ
```

**Analogy đơn giản:**
- **ANSP** = Traffic control center cho máy bay
- **eFPL** = GPS navigation với đầy đủ thông tin thay vì bản đồ giấy
- **GUFI** = Biển số xe duy nhất cho mỗi chuyến bay  
- **FIXM** = Ngôn ngữ chung mà tất cả nước hiểu
- **X.509** = Passport thay vì danh thiếp

---
## **MỤC 6.1: AUTHENTICATION & AUTHORIZATION**

### **GIẢI THÍCH ĐƠN GIẢN:**

#### **Authentication = "Bạn là ai?"**
```
Giống như: Security guard kiểm tra CMND ở cửa tòa nhà
Trong FF-ICE: Kiểm tra certificate để biết ai đang gọi API
```

#### **Authorization = "Bạn được phép làm gì?"**
```
Giống như: Sau khi vào tòa nhà, bạn chỉ được vào tầng 5, không được vào tầng 10
Trong FF-ICE: Vietnam Airlines được query flight, nhưng không được query của VietJet
```

---

## **CHI TIẾT TECHNICAL:**

### **Certificate-based Authentication:**

#### **Thay vì Username/Password:**
```
❌ Traditional login:
POST /api/login
{
  "username": "vietnam_airlines",  
  "password": "secret123"
}

✅ Certificate-based:
GET /swim/query/flight/VN123-001
Headers:
  X-Client-Certificate: [VietnamAirlines.crt]
  Authorization: Certificate-based
```

#### **X.509 Certificate Structure:**
```
Certificate Contents:
Subject: CN=Vietnam Airlines, O=Vietnam Airlines JSC, C=VN
Issuer: CN=VATM Certificate Authority, O=VATM, C=VN  
Valid From: 2025-01-01
Valid Until: 2027-01-01
Public Key: RSA 2048-bit
Digital Signature: SHA-256
```

### **How It Works:**

#### **Step 1: Certificate Installation**
```python
# Vietnam Airlines installs certificate in their system
certificate_path = "/etc/ssl/certs/vietnam-airlines.crt"
private_key_path = "/etc/ssl/private/vietnam-airlines.key"

# When calling VATM API
requests.get(
    "https://swim.vatm.vn/query/flight/VN123-001",
    cert=(certificate_path, private_key_path),  # Client certificate
    verify="/etc/ssl/certs/vatm-ca.crt"        # Verify server
)
```

#### **Step 2: VATM Validates Certificate**
```python
def validate_client_certificate(request):
    # 1. Extract certificate from HTTPS connection
    client_cert = request.environ['SSL_CLIENT_CERT']
    
    # 2. Verify certificate chain
    if not verify_certificate_chain(client_cert):
        return HTTPError(401, "Invalid certificate")
    
    # 3. Check if certificate is revoked
    if is_certificate_revoked(client_cert):
        return HTTPError(401, "Certificate revoked")
        
    # 4. Extract client identity
    client_id = extract_subject_cn(client_cert)  # "Vietnam Airlines"
    
    return client_id
```

#### **Step 3: Authorization Check**
```python
def authorize_request(client_id, operation, resource):
    # Check permissions in database
    permissions = get_client_permissions(client_id)
    
    if operation == "query_flight":
        if "flight_query" in permissions:
            return True
            
    if operation == "file_flight":
        if "flight_filing" in permissions:
            # Additional check: can only file own flights
            if resource.airline == client_id:
                return True
                
    return False
```

---

## **AUTHORIZATION MATRIX:**

### **Client Permissions:**
```python
AUTHORIZED_CLIENTS = {
    "Vietnam Airlines": {
        "permissions": ["flight_query", "flight_filing"],
        "scope": "own_flights",  # Can only access own flights
        "rate_limit": 100,       # 100 requests/minute
        "valid_until": "2027-01-01"
    },
    
    "VietJet Air": {
        "permissions": ["flight_query", "flight_filing"], 
        "scope": "own_flights",
        "rate_limit": 100,
        "valid_until": "2027-01-01"
    },
    
    "Thailand AEROTHAI": {
        "permissions": ["flight_query"],  # No filing, only query
        "scope": "regional_flights",      # Can query flights in region
        "rate_limit": 200,
        "valid_until": "2026-12-31"
    },
    
    "VATM Operations": {
        "permissions": ["flight_query", "flight_filing", "admin"],
        "scope": "all_flights",          # Can access everything
        "rate_limit": 1000,
        "valid_until": "2030-01-01"
    }
}
```

### **Permission Examples:**
```python
# Vietnam Airlines tries to query VietJet flight
client_id = "Vietnam Airlines"
requested_flight = "VJ123-20250315-001"  # VietJet flight

# Authorization check
if client_permissions[client_id]["scope"] == "own_flights":
    if not requested_flight.startswith("VN"):  # Not Vietnam Airlines flight
        return HTTPError(403, "Access denied")

# Thailand AEROTHAI queries Vietnam flight
client_id = "Thailand AEROTHAI"  
requested_flight = "VN123-20250315-001"

# Check if flight is regional (crosses Thailand airspace)
flight_route = get_flight_route(requested_flight)
if "VTBS" in flight_route or "VT" in flight_route:  # Thailand airports/waypoints
    return allow_access()  # Regional flight, allowed
else:
    return HTTPError(403, "Not authorized for this route")
```

---

## **IMPLEMENTATION CODE:**

### **FastAPI Authentication Decorator:**
```python
from fastapi import Depends, HTTPException, Request
from fastapi.security import HTTPBearer
import ssl

security = HTTPBearer()

def get_current_client(request: Request):
    # Extract client certificate from HTTPS connection
    try:
        client_cert = request.scope["transport"]["ssl_object"].getpeercert()
    except:
        raise HTTPException(401, "Client certificate required")
    
    # Validate certificate
    client_id = validate_certificate(client_cert)
    if not client_id:
        raise HTTPException(401, "Invalid certificate")
        
    return client_id

def require_permission(required_permission: str):
    def decorator(client_id: str = Depends(get_current_client)):
        permissions = get_client_permissions(client_id)
        if required_permission not in permissions:
            raise HTTPException(403, f"Permission '{required_permission}' required")
        return client_id
    return decorator

# Usage in API endpoints
@app.get("/swim/query/flight/{gufi}")
async def query_flight(
    gufi: str, 
    client_id: str = Depends(require_permission("flight_query"))
):
    # Additional authorization check
    if not can_access_flight(client_id, gufi):
        raise HTTPException(403, "Access denied to this flight")
        
    return get_flight_data(gufi)
```

### **Certificate Validation Function:**
```python
import ssl, socket
from cryptography import x509
from cryptography.hazmat.backends import default_backend

def validate_certificate(cert_der):
    try:
        # Parse certificate
        cert = x509.load_der_x509_certificate(cert_der, default_backend())
        
        # Check expiration
        if cert.not_valid_after < datetime.now():
            return None
            
        # Check issuer (must be signed by VATM CA)
        issuer = cert.issuer.get_attributes_for_oid(x509.OID_COMMON_NAME)[0].value
        if issuer != "VATM Certificate Authority":
            return None
            
        # Extract client identity
        subject = cert.subject.get_attributes_for_oid(x509.OID_COMMON_NAME)[0].value
        
        # Check revocation list
        if is_revoked(cert.serial_number):
            return None
            
        return subject  # Return client ID
        
    except Exception as e:
        logger.error(f"Certificate validation error: {e}")
        return None
```

---

## **SECURITY BENEFITS:**

### **Why Certificate-based > Username/Password:**
```
✅ Cannot be easily stolen (needs private key)
✅ Cannot be brute-forced  
✅ Automatic expiration
✅ Can be revoked instantly
✅ Non-repudiation (can't deny making request)
✅ Industry standard for aviation

❌ Username/Password problems:
- Can be intercepted
- Can be brute-forced
- Often reused across systems
- Hard to revoke access
- No automatic expiration
```

### **Certificate Lifecycle:**
```
1. Certificate Request:
   Vietnam Airlines → VATM: "We need certificate"
   
2. Certificate Issuance:
   VATM → Vietnam Airlines: "Here's your certificate"
   
3. Daily Operations:
   Vietnam Airlines uses certificate to call SWIM APIs
   
4. Certificate Renewal (before expiration):
   Vietnam Airlines → VATM: "Renew our certificate"
   
5. Emergency Revocation (if compromised):
   VATM → Certificate Revocation List: "Block certificate #12345"
```

**TÓM TẮT:**
- **Authentication**: Dùng certificate thay username/password
- **Authorization**: Kiểm tra quyền hạn dựa trên client identity  
- **Security**: Mỗi airline chỉ được access flight của mình
- **Regional**: ANSPs láng giềng được query flights qua airspace của họ
- **Admin**: VATM có quyền full access để quản lý hệ thống

---

**BẠN PHÁT HIỆN RA VẤN ĐỀ QUAN TRỌNG!** Đây là gap trong specification. Để tôi làm rõ:

## **DATABASE OWNERSHIP - VẤN ĐỀ CHƯA ĐƯỢC GIẢI QUYẾT:**

### **Hiện tại trong spec:**
```
❓ Filing Service cần database để lưu eFPL
❓ Query Service cần database để search flights  
❓ Nhưng KHÔNG nói rõ ai làm database
❓ Database chạy ở đâu trong SWIM architecture
```

### **3 SCENARIOS POSSIBLE:**

---

## **SCENARIO 1: SHARED DATABASE (Recommended)**

### **Architecture:**
```mermaid
graph TB
    subgraph "SWIM Infrastructure (Machine 4)"
        DB[PostgreSQL Database<br/>Shared Flight Plans DB<br/>Port 5432]
        BACKUP[Database Backup<br/>& Replication]
    end
    
    subgraph "VATM Services (Machine 5)"
        FS[Filing Service<br/>Team A develops<br/>Port 8101]
        QS[Query Service<br/>Team B develops<br/>Port 8100]
    end
    
    FS -.->|Write eFPL| DB
    QS -.->|Read eFPL| DB
    DB --> BACKUP
```

### **Team Responsibilities:**
```
Database Team (Machine 4):
✅ PostgreSQL installation & configuration  
✅ Database schema creation & maintenance
✅ Backup & recovery procedures
✅ Performance tuning & monitoring
✅ Security & access control

Filing Service Team (Machine 5):
✅ Filing Service API development
✅ FIXM validation logic
✅ Database integration (connect to shared DB)
❌ NOT responsible for database management

Query Service Team (Machine 5):  
✅ Query Service API development
✅ Search algorithms & optimization
✅ Database integration (connect to shared DB)
❌ NOT responsible for database management
```

### **Implementation:**
```python
# Both teams use same database connection
DATABASE_CONFIG = {
    "host": "machine4.vatm.local",
    "port": 5432,
    "database": "swim_flight_plans",
    "username": "swim_app_user",
    "password": "secure_password"
}

# Filing Service (Team A)
class FilingService:
    def __init__(self):
        self.db = connect_to_database(DATABASE_CONFIG)
    
    def submit_efpl(self, efpl):
        # Insert into shared flight_plans table
        self.db.execute("""
            INSERT INTO flight_plans (gufi, fixm_data, ...) 
            VALUES (%s, %s, ...)
        """, (gufi, fixm_data))

# Query Service (Team B)  
class QueryService:
    def __init__(self):
        self.db = connect_to_database(DATABASE_CONFIG)  # Same DB!
    
    def query_flight(self, gufi):
        # Read from same flight_plans table
        return self.db.execute("""
            SELECT * FROM flight_plans WHERE gufi = %s
        """, (gufi,))
```

---

## **SCENARIO 2: SEPARATE DATABASES (Not Recommended)**

### **Architecture:**
```mermaid
graph TB
    subgraph "Filing Service Infrastructure"
        FS[Filing Service<br/>Port 8101]
        FS_DB[Filing PostgreSQL<br/>Port 5434]
        FS --> FS_DB
    end
    
    subgraph "Query Service Infrastructure"  
        QS[Query Service<br/>Port 8100]
        QS_DB[Query PostgreSQL<br/>Port 5435]
        QS --> QS_DB
    end
    
    FS_DB -.->|Data Sync| QS_DB
```

### **Problems với approach này:**
```
❌ Data synchronization complexity
❌ Potential data inconsistency  
❌ Double storage costs
❌ More failure points
❌ Sync delays between services
```

---

## **SCENARIO 3: SERVICE-OWNED DATABASES (Complex)**

### **Microservices approach:**
```
Filing Service owns:
- flight_plans table (for filing operations)
- filing_status, validation_logs

Query Service owns:
- flight_search_index (optimized for queries)
- query_cache, search_history

Integration:
- Filing Service publishes events
- Query Service subscribes to events
- Eventual consistency model
```

---

## **RECOMMENDATION CHO VATM:**

### **Go with SCENARIO 1 - Shared Database:**

#### **Team Structure:**
```
Team 1: Database & Infrastructure (2 people)
├── Database Administrator
├── Infrastructure Engineer  
└── Responsibilities:
    - Machine 4 setup (PostgreSQL + monitoring)
    - Database schema management
    - Backup & recovery
    - Performance optimization

Team 2: Filing Service Development (3 people)
├── Senior Developer (Team Lead)
├── Backend Developer
├── QA Engineer
└── Responsibilities:
    - Filing Service API (Machine 5)
    - FIXM validation
    - Business logic
    - Unit & integration testing

Team 3: Query Service Development (3 people)  
├── Senior Developer (Team Lead)
├── Backend Developer
├── QA Engineer
└── Responsibilities:
    - Query Service API (Machine 5)
    - Search optimization
    - Caching strategies
    - Performance testing
```

### **Database Schema Ownership:**
```sql
-- Shared schema managed by Database Team
CREATE TABLE flight_plans (
    id SERIAL PRIMARY KEY,
    gufi VARCHAR(20) UNIQUE NOT NULL,
    -- ... all fields as specified
    
    -- Metadata for service coordination
    created_by_service VARCHAR(20) DEFAULT 'filing',
    last_updated_by VARCHAR(20),
    version INTEGER DEFAULT 1
);

-- Service-specific tables if needed
CREATE TABLE filing_audit_log (
    id SERIAL PRIMARY KEY,
    gufi VARCHAR(20) REFERENCES flight_plans(gufi),
    operation VARCHAR(20),
    timestamp TIMESTAMP DEFAULT NOW()
);

CREATE TABLE query_performance_log (
    id SERIAL PRIMARY KEY, 
    query_type VARCHAR(50),
    execution_time_ms INTEGER,
    timestamp TIMESTAMP DEFAULT NOW()
);
```

### **Development Coordination:**
```python
# Database Team provides shared utilities
# /shared/database/models.py
class FlightPlan(BaseModel):
    gufi: str
    flight_number: str  
    # ... all shared fields
    
class DatabaseConnection:
    @staticmethod
    def get_connection():
        return connect_to_shared_db()

# Filing Service Team uses shared models
from shared.database.models import FlightPlan, DatabaseConnection

class FilingService:
    def submit_efpl(self, efpl_data):
        db = DatabaseConnection.get_connection()
        flight_plan = FlightPlan(**efpl_data)
        # Insert logic

# Query Service Team uses same shared models  
from shared.database.models import FlightPlan, DatabaseConnection

class QueryService:
    def query_flight(self, gufi):
        db = DatabaseConnection.get_connection()  
        # Query logic
```

## **REVISED TEAM ASSIGNMENTS:**

### **Machine 4 (Database Team):**
```bash
# Services running on Machine 4
sudo systemctl status postgresql      # Main database
sudo systemctl status postgresql-replica  # Backup replica
sudo systemctl status prometheus      # Monitoring
sudo systemctl status grafana        # Dashboard

# Database Team responsibilities
- PostgreSQL installation & tuning
- Schema creation & migration scripts
- Backup automation (daily, weekly)
- Performance monitoring & alerts
- Security (database-level encryption)
```

### **Machine 5 (Application Teams):**
```bash
# Services running on Machine 5  
sudo systemctl status filing-service   # Team A develops
sudo systemctl status query-service   # Team B develops
sudo systemctl status swim-adapter    # Shared component

# Connection to Machine 4 database
DATABASE_URL=postgresql://machine4:5432/swim_flight_plans
```

### **Coordination Meetings:**
```
Weekly Database Schema Review:
- Database Team presents schema changes
- Filing Team reviews impact on validation
- Query Team reviews impact on search performance
- All teams approve before implementation

Daily Standups:
- Each team reports progress independently
- Database Team reports any infrastructure issues
- Integration issues discussed immediately
```

**KẾT LUẬN:**
- ✅ **Database chạy trên Machine 4** (SWIM Infrastructure)
- ✅ **Database Team riêng** quản lý schema + infrastructure  
- ✅ **Filing & Query Teams** develop services độc lập
- ✅ **Shared database** cho data consistency
- ✅ **Clear ownership**: Database Team owns data, Service Teams own business logic

**Bạn phát hiện đúng gap này! Cần bổ sung Database Team vào project plan.**


---

**CHÍNH XẮC 100%!** Bạn hiểu đúng rồi!

## **SWIM = Infrastructure Provider**

### **SWIM cung cấp:**
```
✅ Truyền thông: HTTP/HTTPS, message routing
✅ Kết nối: Service registry, authentication  
✅ Database: PostgreSQL để lưu flight data
✅ Monitoring: Prometheus, Grafana
✅ Security: PKI certificates, encryption
```

### **2 dịch vụ FF-ICE = Tenant/User của SWIM:**
```
Filing Service:
- Sử dụng SWIM database để lưu eFPL
- Sử dụng SWIM network để nhận requests
- Sử dụng SWIM security để authenticate

Query Service:  
- Sử dụng SWIM database để đọc eFPL
- Sử dụng SWIM network để trả response
- Sử dụng SWIM security để authorize
```

## **ANALOGY ĐỠN GIẢN:**

### **SWIM giống như:**
```
🏢 Tòa nhà văn phòng cung cấp:
- Điện, nước, internet (infrastructure)
- Hệ thống bảo vệ (security)
- Thang máy, hành lang (networking)  
- Kho chung để lưu đồ (database)
```

### **FF-ICE services giống như:**
```
🏪 2 cửa hàng trong tòa nhà:
- Filing Service = Cửa hàng nhận hàng (nhận eFPL)
- Query Service = Cửa hàng tra cứu (tìm eFPL)

Cả 2 đều:
✅ Dùng chung điện nước của tòa nhà
✅ Dùng chung kho để lưu hàng  
✅ Dùng chung hệ thống bảo vệ
✅ Khách hàng đi thang máy chung để tới
```

## **TECHNICAL ARCHITECTURE:**

```mermaid
graph TB
    subgraph "SWIM Infrastructure Platform"
        subgraph "Shared Resources"
            DB[PostgreSQL Database<br/>📊 Chung cho tất cả services]
            NET[Networking Layer<br/>🌐 HTTP/HTTPS routing]
            SEC[Security Layer<br/>🔐 Authentication & PKI]
            MON[Monitoring<br/>📈 Logs & metrics]
        end
    end
    
    subgraph "FF-ICE Services (Tenants)"
        FILING[Filing Service<br/>📝 Team A develops]
        QUERY[Query Service<br/>🔍 Team B develops]
    end
    
    FILING -.->|Uses| DB
    FILING -.->|Uses| NET
    FILING -.->|Uses| SEC
    
    QUERY -.->|Uses| DB
    QUERY -.->|Uses| NET  
    QUERY -.->|Uses| SEC
    
    DB --> MON
    NET --> MON
    SEC --> MON
```

## **CODE EXAMPLE:**

### **SWIM cung cấp shared database:**
```python
# SWIM Infrastructure team setup này:
DATABASE_CONFIG = {
    "host": "swim-infrastructure.vatm.vn",
    "port": 5432,
    "database": "swim_shared_db",
    "schema": "flight_data"
}
```

### **Filing Service sử dụng SWIM database:**
```python  
# Team A chỉ cần connect và dùng
from swim_infrastructure import get_database_connection

class FilingService:
    def __init__(self):
        self.db = get_database_connection()  # SWIM's database
        
    def submit_flight_plan(self, efpl):
        # Lưu vào SWIM's shared database
        self.db.execute("""
            INSERT INTO flight_plans (gufi, data) 
            VALUES (%s, %s)
        """, (efpl.gufi, efpl.data))
```

### **Query Service sử dụng cùng SWIM database:**
```python
# Team B cũng connect cùng database  
from swim_infrastructure import get_database_connection

class QueryService:
    def __init__(self):
        self.db = get_database_connection()  # Same SWIM database
        
    def get_flight_plan(self, gufi):
        # Đọc từ SWIM's shared database
        return self.db.execute("""
            SELECT * FROM flight_plans WHERE gufi = %s
        """, (gufi,))
```

## **BENEFITS:**

### **Cho Development Teams:**
```
✅ Không cần setup database
✅ Không cần lo về networking
✅ Không cần implement security
✅ Focus vào business logic only
✅ Shared data automatically
```

### **Cho VATM:**
```
✅ Consistent architecture
✅ Centralized monitoring
✅ Easier maintenance  
✅ Better security
✅ Cost effective
```

## **TEAM RESPONSIBILITIES:**

### **SWIM Infrastructure Team:**
```
Provide & maintain:
- Database server + schema
- Network routing + load balancing
- Security + certificates  
- Monitoring + alerting
- Backup + disaster recovery
```

### **FF-ICE Service Teams:**
```
Develop & maintain:
- Business logic only
- API endpoints
- Data validation
- Service-specific features

Use SWIM's:
- Database (connect và dùng)
- Network (register service)
- Security (use certificates)
```

**TÓM LẠI:**
- ✅ **SWIM = Platform** (infrastructure + database)
- ✅ **FF-ICE Services = Applications** chạy trên platform
- ✅ **2 dịch vụ dùng chung database của SWIM**
- ✅ **Giống như apps chạy trên cloud platform**

**Bạn hiểu hoàn toàn chính xác!** 🎯


---

## **PHÂN TÍCH YÊU CẦU CỦA 2 TEAM**

### **🚨 NHỮNG VẤN ĐỀ NGHIÊM TRỌNG:**

---

## **1. HIỂU SAI SCOPE CƠ BẢN**

### **❌ Team đang làm FULL FF-ICE suite thay vì 2 mandatory services:**
```
Team proposal: 8+ services phức tạp
✅ Filed Flight Plan
✅ Flight Plan Update         ← KHÔNG bắt buộc 
✅ Flight Cancellation        ← KHÔNG bắt buộc
✅ Submission Response
✅ Filing Status
✅ Flight Data Request        ← Service khác hoàn toàn
✅ Flight Data Response

Thực tế cần: 2 mandatory services
✅ Filing Service (submit + response)  
✅ Query Service (search + retrieve)
```

### **💰 Impact:** Budget tăng 4-5 lần, timeline tăng 3-4 lần

---

## **2. GOLD-PLATING - OVER-ENGINEERING**

### **❌ GUI phức tạp không cần thiết:**
```
Team muốn build:
- "Trực quan hóa quỹ đạo 4D trên bản đồ"
- "Hiển thị FIR boundaries"  
- "Interactive trajectory editing"
- "Real-time collaboration interface"

Thực tế FF-ICE cần:
- REST API endpoints (không cần GUI fancy)
- FIXM XML processing
- Database CRUD operations
- Basic status responses
```

### **💰 Impact:** +$500K development cost, +6 tháng timeline

---

## **3. CONFUSION VỀ MESSAGE TYPES**

### **❌ Mixing FF-ICE với other aviation messages:**
```
Team mention:
✅ CTOT (Calculated Take-Off Time)    ← Đây là ATFM, không phải FF-ICE
✅ NOTAM (Notice to Airmen)           ← Đây là AIS, không phải FF-ICE  
✅ "Đàm phán trajectory"              ← Đây là FF-ICE/R2, không phải R1
✅ "Geographic constraints display"   ← Advanced feature, không mandatory
```

### **Reality check:**
```
FF-ICE/R1 chỉ cần:
- Submit eFPL → Store in database
- Query eFPL → Return from database
- Basic validation + error responses
```

---

## **4. TECHNICAL OVER-COMPLEXITY**

### **❌ Real-time collaboration features:**
```
Team proposal:
- "Real-time status updates"
- "Interactive trajectory negotiation"  
- "Multi-party collaborative editing"
- "Live map visualization"

FF-ICE/R1 reality:
- Simple request/response model
- Store eFPL in database
- Return stored data on query
- No real-time features required
```

---

## **NHỮNG GÌ THỰC SỰ CẦN LÀM:**

### **✅ FILING SERVICE - SIMPLE:**
```python
@app.post("/filing/submit")
async def file_flight_plan(efpl: str):
    # 1. Parse FIXM XML
    # 2. Validate mandatory fields  
    # 3. Generate GUFI
    # 4. Store in database
    # 5. Return status (ACCEPTED/REJECTED)
    pass

# That's it! No GUI, no maps, no real-time updates
```

### **✅ QUERY SERVICE - SIMPLE:**
```python
@app.get("/query/flight/{gufi}")
async def get_flight_plan(gufi: str):
    # 1. Search database by GUFI
    # 2. Return FIXM XML if found
    # 3. Return 404 if not found
    pass

@app.get("/query/search")
async def search_flights(flight_number: str, date: str):
    # 1. Search database by criteria
    # 2. Return list of matching flights
    pass

# That's it! No fancy visualization needed
```

---

## **REVISED REQUIREMENTS:**

### **Filing Service - Core Only:**
```
✅ MUST HAVE:
- Accept FIXM eFPL via HTTPS POST
- Validate XML against FIXM schema  
- Generate unique GUFI
- Store in SWIM database
- Return filing status (ACCEPTED/REJECTED/PENDING)
- Handle authentication via certificates

❌ REMOVE:
- Map visualization
- Trajectory editing interface
- Real-time collaboration
- Update/Cancel services (optional)
- CTOT/NOTAM integration
```

### **Query Service - Core Only:**
```
✅ MUST HAVE:
- Query by GUFI via HTTPS GET
- Search by flight number + date
- Return FIXM XML format
- Handle "not found" cases
- Support basic filtering

❌ REMOVE:  
- Interactive maps
- 4D trajectory visualization
- Real-time status updates
- Advanced filtering UI
- Flight data request service
```

---

## **SIMPLIFIED PROJECT SCOPE:**

### **What teams should build:**
```
Team A: Filing Service
├── Python FastAPI application
├── FIXM XML validation  
├── Database integration (SWIM's PostgreSQL)
├── Authentication handling
└── Unit tests

Team B: Query Service  
├── Python FastAPI application
├── Database search functionality
├── FIXM XML generation
├── Response formatting
└── Unit tests

Both teams: NO GUI development needed
```

### **Timeline reduction:**
```
Original team proposal: 12-18 months
Simplified scope: 4-6 months
Cost reduction: $1.5M → $400K
```

---

## **KEY MESSAGES CHO TEAMS:**

### **1. Focus on Standards Compliance:**
```
"Build exactly what ICAO Doc 9965 specifies for mandatory services.
No more, no less."
```

### **2. API-First Approach:**
```
"FF-ICE is about machine-to-machine communication.
Airlines will use their own software to call your APIs.
No fancy GUI needed."
```

### **3. SWIM Integration Priority:**
```
"Your services are plugins to SWIM infrastructure.
Focus on integrating with existing SWIM database and security.
Don't reinvent infrastructure."
```

### **4. Start Minimal, Iterate:**
```
"Build basic Filing + Query first.
Optional features can be added in Phase 2 if business needs exist."
```

---

## **RECOMMENDED ACTION:**

### **Immediate steps:**
1. **Stop current development** on GUI/visualization features
2. **Refocus on core APIs** only  
3. **Simplify technical architecture** to basic REST services
4. **Remove optional FF-ICE services** from current scope
5. **Align with SWIM infrastructure** plan

### **New timeline:**
```
Month 1-2: Core Filing Service API
Month 3-4: Core Query Service API  
Month 5-6: Integration testing + deployment

Total: 6 months instead of 18 months
```

**BOTTOM LINE:** Teams đang over-engineer một solution đơn giản. Cần refocus vào 2 mandatory services với minimal features để meet ICAO compliance, không phải build aviation management suite.

---

## **CHỨC NĂNG FILING SERVICE (Dịch vụ Nộp Kế hoạch Bay)**

### **Accept eFPL submissions**
```
= Nhận kế hoạch bay mở rộng từ hãng hàng không
Ví dụ: Vietnam Airlines gửi kế hoạch bay VN123 từ HAN đi SGN
```

### **Validate eFPL format** 
```
= Kiểm tra định dạng kế hoạch bay có đúng chuẩn không
Ví dụ: Có đủ thông tin máy bay, sân bay xuất phát, sân bay đến không?
```

### **Generate GUFI**
```
= Tạo mã định danh duy nhất cho chuyến bay
Ví dụ: VN123-20250315-001 (VN123 ngày 15/3/2025, chuyến số 1)
```

### **Store flight plans**
```
= Lưu kế hoạch bay vào cơ sở dữ liệu
Ví dụ: Lưu thông tin VN123 để sau có thể tra cứu
```

### **Return filing status**
```
= Trả lời kết quả nộp kế hoạch bay
Ví dụ: "CHẤP NHẬN" hoặc "TỪ CHỐI" hoặc "ĐANG XỬ LÝ"
```

---

## **CHỨC NĂNG QUERY SERVICE (Dịch vụ Tra cứu Kế hoạch Bay)**

### **Query by GUFI**
```
= Tìm chuyến bay bằng mã định danh duy nhất
Ví dụ: Nhập "VN123-20250315-001" → Hiện thông tin chuyến VN123
```

### **Query by flight number**
```
= Tìm chuyến bay bằng số hiệu
Ví dụ: Nhập "VN123" + "15/3/2025" → Hiện chuyến VN123 ngày đó
```

### **Query by route**
```
= Tìm chuyến bay theo tuyến đường
Ví dụ: Tìm tất cả chuyến bay từ HAN đi SGN hôm nay
```

### **Query by time range**
```
= Tìm chuyến bay trong khoảng thời gian
Ví dụ: Tất cả chuyến bay từ 8:00-10:00 sáng nay
```

### **Handle not found**
```
= Xử lý khi không tìm thấy chuyến bay
Ví dụ: Nhập "VN999" → Trả lời "Không tìm thấy chuyến bay này"
```

### **Return FIXM format**
```
= Trả thông tin chuyến bay theo định dạng chuẩn quốc tế
Ví dụ: Thông tin VN123 theo format mà tất cả nước hiểu
```

### **Support filtering**
```
= Hỗ trợ lọc kết quả theo điều kiện
Ví dụ: Chỉ hiện chuyến của Vietnam Airlines, hoặc chỉ máy bay A321
```

---

## **TÓM TẮT BẰNG VÍ DỤ THỰC TẾ:**

### **Filing Service giống như:**
```
🏪 Quầy tiếp nhận hồ sơ
- Hãng hàng không mang kế hoạch bay đến nộp
- Nhân viên kiểm tra hồ sơ có đầy đủ không
- Đóng dấu số thứ tự duy nhất
- Cất hồ sơ vào tủ
- Trả phiếu báo "Đã nhận" hoặc "Thiếu giấy tờ"
```

### **Query Service giống như:**
```
🔍 Quầy tra cứu thông tin
- Khách hàng đưa số thứ tự → Tìm hồ sơ tương ứng
- Hoặc nói tên chuyến bay → Tìm trong sổ ghi chép
- Hoặc hỏi "chuyến nào từ Hà Nội đi TP.HCM?" → Liệt kê tất cả
- Nếu không tìm thấy → Nói "Không có thông tin"
- Trả thông tin theo mẫu chuẩn
```

**NGẮN GỌN:** Filing = Nộp + Lưu, Query = Tìm + Trả

---
**CHÍNH XÁC!** Bạn hiểu đúng flow. Để tôi vẽ rõ:

## **USERS CALL FILING SERVICE:**

### **Primary Users:**
```
✅ Airlines (Hãng hàng không):
- Vietnam Airlines
- VietJet Air  
- Bamboo Airways
- Jetstar Pacific
- International airlines (Thai Airways, Singapore Airlines, etc.)

✅ Ground Handling Companies:
- SASCO, Nasco (đại diện airlines)

✅ Flight Planning Service Providers:
- Jeppesen, LIDO, NavBlue (thay mặt airlines)
```

## **AIRLINE INFRASTRUCTURE CẦN CÓ:**

### **Tại Vietnam Airlines Office:**

```mermaid
graph TB
    subgraph "Vietnam Airlines Operations Center"
        DISPATCHER[Flight Dispatcher<br/>Nhân viên lập kế hoạch bay]
        VNA_SYSTEM[VNA Flight Planning System<br/>Sabre/Amadeus/Internal]
        SWIM_CLIENT[SWIM Client Software<br/>🔑 KEY COMPONENT]
        CERT[VNA Certificate<br/>X.509 for authentication]
    end
    
    subgraph "Network"
        INTERNET[Internet Connection<br/>VPN or Dedicated Line]
    end
    
    subgraph "VATM SWIM Infrastructure"
        SWIM_GW[SWIM Gateway]
        FILING_SVC[Filing Service]
        DB[(Flight Plans Database)]
    end
    
    DISPATCHER --> VNA_SYSTEM
    VNA_SYSTEM --> SWIM_CLIENT
    SWIM_CLIENT --> CERT
    SWIM_CLIENT --> INTERNET
    INTERNET --> SWIM_GW
    SWIM_GW --> FILING_SVC
    FILING_SVC --> DB
    
    classDef vna fill:#ffebee,stroke:#c62828,stroke-width:2px
    classDef vatm fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    
    class DISPATCHER,VNA_SYSTEM,SWIM_CLIENT,CERT vna
    class SWIM_GW,FILING_SVC,DB vatm
```

### **Vietnam Airlines CẦN CÓ:**

#### **1. SWIM Client Software (BẮT BUỘC):**
```python
# VNA cần install SWIM client library
from vatm_swim_client import VATMSWIMClient

client = VATMSWIMClient(
    base_url="https://swim.vatm.vn",
    certificate_path="/etc/ssl/vna-certificate.p12",
    certificate_password="vna_secret_key"
)

# VNA dispatcher tạo eFPL
efpl_data = create_flight_plan(
    flight_number="VN123",
    departure="VVNB", 
    arrival="VVTS",
    departure_time="08:00Z"
)

# Gửi đến VATM Filing Service
response = client.file_flight_plan(efpl_data)
print(f"Status: {response.status}")  # ACCEPTED/REJECTED
```

#### **2. Certificate Authentication:**
```
VNA cần có certificate từ VATM:
- File: vietnam-airlines.crt (public certificate)
- File: vietnam-airlines.key (private key)  
- Issued by: VATM Certificate Authority
- Valid for: 2 years
- Used for: Authentication với SWIM services
```

#### **3. Integration với Existing Systems:**
```
VNA Current System:
┌─────────────────────────┐
│   Sabre Flight Planning │
│   (Existing system)     │
└─────────────────────────┘
            ↓
┌─────────────────────────┐
│   SWIM Client Adapter   │  ← NEW COMPONENT
│   (Convert to FIXM)     │
└─────────────────────────┘
            ↓
┌─────────────────────────┐
│   VATM Filing Service   │
└─────────────────────────┘
```

## **WORKFLOW THỰC TẾ:**

### **Buổi sáng tại Vietnam Airlines:**
```
07:00 - Dispatcher Nguyễn Văn A:
1. Mở Sabre system
2. Tạo flight plan VN123 HAN-SGN
3. Click "Submit to VATM"
4. Sabre → SWIM Client → VATM Filing Service
5. Nhận response: "ACCEPTED - GUFI: VN123-20250315-001"
6. VN123 ready để cất cánh!
```

### **Technical Flow:**
```
VNA Sabre System:
POST http://localhost:8080/internal/submit-to-vatm
{
  "flight": "VN123",
  "route": "HAN-SGN",
  "time": "08:00Z"
}
    ↓
VNA SWIM Client:
- Convert to FIXM format
- Add authentication certificate
- Call VATM API:

POST https://swim.vatm.vn/filing/submit
Content-Type: application/xml
Authorization: Certificate vietnam-airlines.crt
{FIXM XML data}
    ↓
VATM Filing Service:
- Validate certificate
- Process FIXM
- Store in database  
- Return status

HTTP 200 OK
{
  "status": "ACCEPTED",
  "gufi": "VN123-20250315-001"
}
```

## **OTHER AIRLINES SETUP:**

### **VietJet Air:**
```
Similar setup:
- VietJet SWIM Client Software
- VietJet Certificate from VATM
- Integration với VietJet planning system
- Different branding, same technical approach
```

### **International Airlines:**
```
Thai Airways (in Bangkok):
- Thai SWIM Client  
- Connect to Vietnam SWIM network
- Cross-border flight filing
- Use existing AEROTHAI certificates + VATM mutual recognition
```

## **SWIM CLIENT DISTRIBUTION:**

### **VATM Strategy:**
```
Option 1: Free SDK Distribution
- VATM provides SWIM Client SDK
- Airlines develop integration themselves
- Technical support included

Option 2: Commercial Partners
- Partner với Sabre, Amadeus, SITA
- They build SWIM integration
- Airlines buy từ existing vendors

Option 3: SaaS Web Portal
- VATM hosts web-based filing portal
- Airlines access via browser
- No local software installation needed
```

### **Recommended for Vietnam market:**
```
Tier 1 Airlines (VNA, VietJet): Custom SDK integration
Tier 2 Airlines (Bamboo, Jetstar): Web portal access
International Airlines: Partner integration via SITA/ARINC
```

## **CERTIFICATE MANAGEMENT:**

### **VATM Certificate Authority:**
```
VATM issues certificates to:
✅ Vietnam Airlines  
✅ VietJet Air
✅ Bamboo Airways
✅ Thai Airways (for cross-border)
✅ Singapore Airlines (for cross-border)

Certificate contains:
- Airline identity
- Authorized services (filing, query)
- Validity period
- Digital signature
```

**KẾT LUẬN:**
- ✅ **Airlines = Primary users** of Filing Service
- ✅ **Airlines cần SWIM Client Software** at their operations center
- ✅ **Certificate authentication** required  
- ✅ **Integration với existing flight planning systems**
- ✅ **VATM provides SDK hoặc web portal** for access

**Mỗi airline cần invest in SWIM client infrastructure để connect với VATM!**


---
**CHÍNH XẮC!** Bạn phát hiện ra điều quan trọng. Để tôi trả lời từng phần:

## **1. FF-ICE R1 CÓ PHỤ THUỘC SWIM KHÔNG?**

### **Câu trả lời: KHÔNG hoàn toàn phụ thuộc!**

```
ICAO DOC 9965 nói rõ:
✅ FF-ICE có thể implement trên SWIM infrastructure
✅ NHƯNG cũng có thể implement standalone
✅ SWIM chỉ là 1 trong nhiều transport options
```

### **Transport Options cho FF-ICE:**
```
Option 1: SWIM-based (Recommended)
- Service discovery via SWIM registry
- Message routing via SWIM gateways
- Security via SWIM certificates

Option 2: Direct HTTP/HTTPS (Simple)
- Airlines call FF-ICE services directly
- No SWIM infrastructure needed
- Basic authentication (API keys, certificates)

Option 3: AMHS Gateway (Legacy)
- FF-ICE messages via X.400
- Integration với existing AMHS
- For airlines không có modern systems
```

## **2. SWIM CÓ PHẢI LÀ CHUẨN BẮT BUỘC KHÔNG?**

### **Câu trả lời: KHÔNG bắt buộc, nhưng RECOMMENDED!**

#### **ICAO Standards:**
```
✅ FF-ICE message format (FIXM): MANDATORY
✅ Service interfaces (REST/SOAP): MANDATORY  
✅ Security requirements: MANDATORY
❌ SWIM infrastructure: RECOMMENDED but OPTIONAL
```

#### **Implementation flexibility:**
```
Thailand AEROTHAI: Dùng SWIM infrastructure
Singapore CAAS: Dùng SWIM-compliant services  
Some countries: Direct FF-ICE services without full SWIM
```

## **3. TEAM CÓ THỂ LÀM STANDALONE KHÔNG?**

### **HOÀN TOÀN ĐƯỢC!** Đây là approach rất practical:

```mermaid
graph TB
    subgraph "Standalone FF-ICE Implementation"
        subgraph "Airlines"
            VNA[Vietnam Airlines<br/>HTTP Client]
            VJ[VietJet<br/>HTTP Client] 
        end
        
        subgraph "VATM FF-ICE Services (Standalone)"
            LB[Nginx Load Balancer<br/>Port 443]
            FS[Filing Service<br/>Port 8001]
            QS[Query Service<br/>Port 8002]
            AUTH[Simple Auth Service<br/>API Keys]
            DB[(PostgreSQL<br/>Flight Plans)]
        end
        
        VNA -->|HTTPS POST| LB
        VJ -->|HTTPS POST| LB
        LB --> FS
        LB --> QS
        FS --> AUTH
        QS --> AUTH
        FS --> DB
        QS --> DB
    end
    
    classDef simple fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    class FS,QS,AUTH,DB simple
```

### **Standalone Implementation:**

#### **Filing Service - Simple Version:**
```python
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import HTTPBearer
import uuid

app = FastAPI(title="VATM FF-ICE Filing Service")
security = HTTPBearer()

# Simple authentication
API_KEYS = {
    "vna_api_key_123": "Vietnam Airlines",
    "vietjet_api_key_456": "VietJet Air"
}

def authenticate(token: str = Depends(security)):
    if token.credentials not in API_KEYS:
        raise HTTPException(401, "Invalid API key")
    return API_KEYS[token.credentials]

@app.post("/ffice/filing/submit")
async def file_flight_plan(
    efpl_xml: str,
    airline: str = Depends(authenticate)
):
    # 1. Parse FIXM XML
    flight_plan = parse_fixm(efpl_xml)
    
    # 2. Generate GUFI  
    gufi = f"{flight_plan.flight_number}-{date.today().strftime('%Y%m%d')}-001"
    
    # 3. Store in database
    save_flight_plan(gufi, efpl_xml, airline)
    
    # 4. Return status
    return {
        "status": "ACCEPTED",
        "gufi": gufi,
        "timestamp": datetime.now().isoformat()
    }
```

#### **Query Service - Simple Version:**
```python
@app.get("/ffice/query/flight/{gufi}")
async def query_flight(
    gufi: str,
    airline: str = Depends(authenticate)
):
    # 1. Search database
    flight_plan = get_flight_by_gufi(gufi)
    
    if not flight_plan:
        raise HTTPException(404, "Flight not found")
    
    # 2. Check permissions (airline can only see own flights)
    if not can_access_flight(airline, flight_plan):
        raise HTTPException(403, "Access denied")
    
    # 3. Return FIXM
    return {
        "gufi": gufi,
        "fixm_data": flight_plan.fixm_xml,
        "status": flight_plan.status
    }
```

## **4. STANDALONE VS SWIM COMPARISON:**

### **Standalone Approach:**
```
✅ Pros:
- Simple to implement (2-3 months)
- Lower cost ($200K vs $800K)
- No SWIM complexity
- Full control over infrastructure
- Easy testing and debugging

❌ Cons:
- No service discovery
- Manual configuration for each airline
- Limited interoperability với other countries
- No standardized security framework
- Harder to scale internationally
```

### **SWIM Approach:**
```
✅ Pros:
- Industry standard approach
- Automatic service discovery
- Better security framework
- Easy integration với other ANSPs
- Future-proof architecture

❌ Cons:
- Complex to implement (6-8 months)
- Higher cost ($800K+)
- Learning curve for SWIM concepts
- Dependencies on SWIM infrastructure
```

## **5. HYBRID APPROACH - BEST OF BOTH:**

### **Start Standalone, Migrate to SWIM Later:**
```
Phase 1 (3 months): Standalone FF-ICE
- Direct HTTP APIs
- Simple authentication
- Basic functionality
- Immediate value to airlines

Phase 2 (6 months later): Add SWIM compatibility
- Keep existing APIs running
- Add SWIM service registration  
- Gradual migration of airlines
- International interoperability
```

### **Implementation Strategy:**
```python
# Design APIs to be SWIM-compatible từ đầu
@app.post("/ffice/filing/submit")  # Standalone endpoint
@app.post("/swim/filing/submit")   # SWIM-compliant endpoint (same logic)
async def file_flight_plan_swim(efpl_xml: str):
    # Same implementation, different authentication
    return file_flight_plan_logic(efpl_xml)
```

## **6. TESTING WITHOUT SWIM:**

### **Team có thể test ngay bây giờ:**
```python
# 1. Setup simple test environment
pip install fastapi uvicorn lxml pytest

# 2. Create mock airline client
import requests

def test_filing_service():
    efpl_xml = create_test_fixm_message()
    response = requests.post(
        "http://localhost:8000/ffice/filing/submit",
        headers={"Authorization": "Bearer vna_api_key_123"},
        data=efpl_xml
    )
    assert response.status_code == 200
    assert response.json()["status"] == "ACCEPTED"

# 3. Run tests immediately
pytest test_ffice.py
```

### **Mock Data Setup:**
```python
# Create test eFPL messages
def create_test_efpl():
    return """
    <FlightPlan xmlns="http://www.fixm.aero/flight/4.2">
        <gufi>VN123-20250315-001</gufi>
        <flight>
            <flightNumber>VN123</flightNumber>
            <aircraft>
                <aircraftType>A321</aircraftType>
            </aircraft>
            <departure>
                <aerodrome>VVNB</aerodrome>
                <scheduledTimeOfDeparture>2025-03-15T08:00:00Z</scheduledTimeOfDeparture>
            </departure>
            <arrival>
                <aerodrome>VVTS</aerodrome>
            </arrival>
        </flight>
    </FlightPlan>
    """
```

## **RECOMMENDATION:**

### **Start Standalone Approach:**
```
Week 1-4: Build basic Filing + Query services
Week 5-8: Add authentication and database
Week 9-12: Testing and documentation
Week 13-16: Airline integration and go-live

Total: 4 months, $300K budget
```

### **Benefits:**
- ✅ **Immediate results** and learning
- ✅ **Prove concept** to management
- ✅ **Start airline onboarding** early
- ✅ **Add SWIM later** when needed for international connectivity

**KẾT LUẬN:** 
- ❌ FF-ICE KHÔNG bắt buộc phải có SWIM
- ✅ Team có thể làm standalone version
- ✅ SWIM compatibility có thể add later
- ✅ Start simple, evolve to complex

**Hãy bắt đầu với standalone approach để có results nhanh!**


---

**CHÍNH XÁC 100%!** Bạn phát hiện ra missing piece quan trọng! 

## **VẤN ĐỀ: THIẾU FF-ICE CLIENT**

### **Hiện tại team có:**
```
✅ Filing Service (backend)
✅ Query Service (backend)  
❌ Client Software để test services
❌ Demo cho airlines
❌ Integration testing tools
```

### **Không có client thì:**
```
❌ Không test được services
❌ Airlines không biết cách integrate
❌ Không demo được cho management
❌ Không có reference implementation cho airlines
```

## **CẦN THÊM: FF-ICE CLIENT DEVELOPMENT**

### **Client Components cần thiết:**

```mermaid
graph TB
    subgraph "FF-ICE Client Suite"
        subgraph "Test/Demo Client"
            WEB_CLIENT[Web-based Test Client<br/>Next.js Dashboard]
            CLI_CLIENT[Command Line Client<br/>Python CLI tool]
        end
        
        subgraph "SDK for Airlines"
            PYTHON_SDK[Python SDK<br/>For airline developers]
            JAVA_SDK[Java SDK<br/>For enterprise systems]
            JS_SDK[JavaScript SDK<br/>For web applications]
        end
        
        subgraph "Integration Examples"
            VNA_DEMO[Vietnam Airlines Demo<br/>Integration example]
            POSTMAN[Postman Collection<br/>API testing]
        end
    end
    
    subgraph "VATM FF-ICE Services"
        FILING[Filing Service]
        QUERY[Query Service]
    end
    
    WEB_CLIENT --> FILING
    CLI_CLIENT --> FILING
    PYTHON_SDK --> FILING
    WEB_CLIENT --> QUERY
    CLI_CLIENT --> QUERY
    PYTHON_SDK --> QUERY
```

## **1. WEB-BASED TEST CLIENT (Priority 1)**

### **Next.js Testing Dashboard:**
```javascript
// FF-ICE Test Client Dashboard
import { useState } from 'react';

export default function FFICETestDashboard() {
  const [eflpXml, setEflpXml] = useState('');
  const [filingResult, setFilingResult] = useState(null);

  const submitFlightPlan = async () => {
    const response = await fetch('/api/ffice/filing/submit', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/xml',
        'Authorization': 'Bearer test_api_key'
      },
      body: eflpXml
    });
    
    const result = await response.json();
    setFilingResult(result);
  };

  return (
    <div className="ffice-test-dashboard">
      <h1>FF-ICE Test Client</h1>
      
      {/* Filing Test Section */}
      <div className="filing-section">
        <h2>Test Filing Service</h2>
        <textarea
          placeholder="Paste FIXM eFPL XML here..."
          value={eflpXml}
          onChange={(e) => setEflpXml(e.target.value)}
          rows={10}
          cols={80}
        />
        <br/>
        <button onClick={submitFlightPlan}>
          Submit Flight Plan
        </button>
        
        {filingResult && (
          <div className="result">
            <h3>Filing Result:</h3>
            <pre>{JSON.stringify(filingResult, null, 2)}</pre>
          </div>
        )}
      </div>
      
      {/* Query Test Section */}
      <div className="query-section">
        <h2>Test Query Service</h2>
        <QueryTestForm />
      </div>
    </div>
  );
}
```

### **Features của Web Test Client:**
```
✅ FIXM eFPL editor với syntax highlighting
✅ One-click filing test
✅ Query interface với multiple search options
✅ Response viewer với formatting
✅ Error display và debugging info
✅ Sample eFPL templates
✅ API documentation integrated
```

## **2. PYTHON SDK (Priority 1)**

### **Airlines Integration Library:**
```python
# vatm_ffice_client.py
import requests
from typing import Optional, Dict, Any
from dataclasses import dataclass

@dataclass
class FlightPlan:
    gufi: str
    flight_number: str
    departure_airport: str
    arrival_airport: str
    departure_time: str
    fixm_data: str

class VATMFFICEClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url.rstrip('/')
        self.api_key = api_key
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/xml'
        })
    
    def file_flight_plan(self, efpl_xml: str) -> Dict[str, Any]:
        """
        Submit eFPL to VATM Filing Service
        
        Args:
            efpl_xml: FIXM-formatted flight plan XML
            
        Returns:
            Dict with status, gufi, and timestamp
        """
        response = self.session.post(
            f'{self.base_url}/ffice/filing/submit',
            data=efpl_xml
        )
        response.raise_for_status()
        return response.json()
    
    def query_flight_by_gufi(self, gufi: str) -> Optional[FlightPlan]:
        """
        Query flight by GUFI
        
        Args:
            gufi: Globally Unique Flight Identifier
            
        Returns:
            FlightPlan object or None if not found
        """
        response = self.session.get(
            f'{self.base_url}/ffice/query/flight/{gufi}'
        )
        
        if response.status_code == 404:
            return None
            
        response.raise_for_status()
        data = response.json()
        
        return FlightPlan(
            gufi=data['gufi'],
            flight_number=data['flight_number'],
            departure_airport=data['departure_airport'],
            arrival_airport=data['arrival_airport'],
            departure_time=data['departure_time'],
            fixm_data=data['fixm_data']
        )
    
    def search_flights(self, 
                      flight_number: Optional[str] = None,
                      date: Optional[str] = None,
                      departure_airport: Optional[str] = None,
                      arrival_airport: Optional[str] = None) -> List[FlightPlan]:
        """
        Search flights by various criteria
        """
        params = {}
        if flight_number:
            params['flight_number'] = flight_number
        if date:
            params['date'] = date
        if departure_airport:
            params['departure'] = departure_airport
        if arrival_airport:
            params['arrival'] = arrival_airport
            
        response = self.session.get(
            f'{self.base_url}/ffice/query/search',
            params=params
        )
        response.raise_for_status()
        
        flights = []
        for item in response.json():
            flights.append(FlightPlan(**item))
        return flights

# Usage example for airlines
if __name__ == "__main__":
    # Vietnam Airlines usage
    client = VATMFFICEClient(
        base_url="https://ffice.vatm.vn",
        api_key="vna_production_key"
    )
    
    # Test filing
    with open("vn123_efpl.xml", "r") as f:
        efpl_xml = f.read()
    
    result = client.file_flight_plan(efpl_xml)
    print(f"Filing result: {result}")
    
    # Test query
    flight = client.query_flight_by_gufi(result['gufi'])
    print(f"Retrieved flight: {flight}")
```

## **3. COMMAND LINE CLIENT (Priority 2)**

### **CLI Tool for Testing:**
```python
# ffice_cli.py
import click
import json
from vatm_ffice_client import VATMFFICEClient

@click.group()
@click.option('--base-url', default='http://localhost:8000', help='FF-ICE service URL')
@click.option('--api-key', default='test_api_key', help='API key for authentication')
@click.pass_context
def cli(ctx, base_url, api_key):
    """VATM FF-ICE Command Line Client"""
    ctx.ensure_object(dict)
    ctx.obj['client'] = VATMFFICEClient(base_url, api_key)

@cli.command()
@click.argument('efpl_file', type=click.File('r'))
def file(ctx, efpl_file):
    """Submit eFPL from file"""
    client = ctx.obj['client']
    efpl_xml = efpl_file.read()
    
    try:
        result = client.file_flight_plan(efpl_xml)
        click.echo(f"✅ Filing successful!")
        click.echo(f"GUFI: {result['gufi']}")
        click.echo(f"Status: {result['status']}")
    except Exception as e:
        click.echo(f"❌ Filing failed: {e}")

@cli.command()
@click.argument('gufi')
def query(ctx, gufi):
    """Query flight by GUFI"""
    client = ctx.obj['client']
    
    flight = client.query_flight_by_gufi(gufi)
    if flight:
        click.echo(f"✅ Flight found:")
        click.echo(f"Flight: {flight.flight_number}")
        click.echo(f"Route: {flight.departure_airport} → {flight.arrival_airport}")
        click.echo(f"Time: {flight.departure_time}")
    else:
        click.echo(f"❌ Flight not found: {gufi}")

@cli.command()
@click.option('--flight-number', help='Flight number')
@click.option('--date', help='Date (YYYY-MM-DD)')
def search(ctx, flight_number, date):
    """Search flights"""
    client = ctx.obj['client']
    
    flights = client.search_flights(
        flight_number=flight_number,
        date=date
    )
    
    if flights:
        click.echo(f"✅ Found {len(flights)} flights:")
        for flight in flights:
            click.echo(f"  {flight.flight_number}: {flight.departure_airport}→{flight.arrival_airport}")
    else:
        click.echo("❌ No flights found")

if __name__ == '__main__':
    cli()

# Usage:
# python ffice_cli.py file sample_efpl.xml
# python ffice_cli.py query VN123-20250315-001
# python ffice_cli.py search --flight-number VN123 --date 2025-03-15
```

## **4. SAMPLE DATA GENERATOR**

### **Test Data Creation:**
```python
# test_data_generator.py
from datetime import datetime, timedelta
import random

def generate_sample_efpl(flight_number: str, 
                        departure: str, 
                        arrival: str,
                        departure_time: datetime) -> str:
    """Generate sample FIXM eFPL for testing"""
    
    gufi = f"{flight_number}-{departure_time.strftime('%Y%m%d')}-001"
    aircraft_types = ['A321', 'B737', 'A320', 'B787']
    
    efpl_xml = f"""<?xml version="1.0" encoding="UTF-8"?>
<FlightPlan xmlns="http://www.fixm.aero/flight/4.2"
             xmlns:fb="http://www.fixm.aero/base/4.2">
    <gufi>{gufi}</gufi>
    <flight>
        <flightNumber>{flight_number}</flightNumber>
        <aircraft>
            <aircraftType>{random.choice(aircraft_types)}</aircraftType>
            <registration>VN-A{random.randint(100,999)}</registration>
        </aircraft>
        <departure>
            <aerodrome>{departure}</aerodrome>
            <scheduledTimeOfDeparture>{departure_time.isoformat()}Z</scheduledTimeOfDeparture>
        </departure>
        <arrival>
            <aerodrome>{arrival}</aerodrome>
            <estimatedTimeOfArrival>{(departure_time + timedelta(hours=2)).isoformat()}Z</estimatedTimeOfArrival>
        </arrival>
        <routeTrajectory>
            <element>
                <elementStartPoint>
                    <position>
                        <lat>21.221944</lat>
                        <lon>105.807222</lon>
                    </position>
                </elementStartPoint>
            </element>
        </routeTrajectory>
    </flight>
</FlightPlan>"""
    
    return efpl_xml

# Generate test data
sample_flights = [
    ("VN123", "VVNB", "VVTS", datetime(2025, 3, 15, 8, 0)),
    ("VJ456", "VVTS", "VVNB", datetime(2025, 3, 15, 10, 0)),
    ("QH789", "VVNB", "VVDN", datetime(2025, 3, 15, 14, 0))
]

for flight_num, dep, arr, time in sample_flights:
    efpl = generate_sample_efpl(flight_num, dep, arr, time)
    with open(f"{flight_num.lower()}_sample.xml", "w") as f:
        f.write(efpl)
    print(f"Generated: {flight_num.lower()}_sample.xml")
```

## **REVISED PROJECT STRUCTURE:**

### **Complete FF-ICE Implementation:**
```
vatm-ffice-project/
├── services/
│   ├── filing-service/          # Team A
│   └── query-service/           # Team B  
├── client-tools/                # NEW - Team C
│   ├── web-client/             # Next.js test dashboard
│   ├── python-sdk/             # Airlines integration library
│   ├── cli-client/             # Command line testing
│   └── sample-data/            # Test eFPL examples
├── documentation/
└── deployment/
```

### **Team C Responsibilities:**
```
Client Development Team (2 người):
✅ Web-based testing dashboard  
✅ Python SDK for airlines
✅ CLI testing tools
✅ Sample data generation
✅ Integration examples
✅ Client documentation
```

## **BENEFITS OF CLIENT TOOLS:**

### **For Development:**
```
✅ Immediate testing của Filing/Query services
✅ Debug và troubleshoot issues  
✅ Performance testing với realistic data
✅ Integration testing workflows
```

### **For Airlines:**
```
✅ Reference implementation examples
✅ Easy integration với existing systems
✅ Reduced development time
✅ Standardized approach
```

### **For VATM:**
```
✅ Professional demos cho management
✅ Airline onboarding support
✅ Reduced support requests
✅ Faster adoption
```

**RECOMMENDATION:**
- ✅ **Add Team C: Client Development**
- ✅ **Build client tools song song với services**  
- ✅ **Start với web test client và Python SDK**
- ✅ **Use clients để test services immediately**

**Không có client tools = Không test được services effectively!**


---

# **TỔNG HỢP CÔNG VIỆC TEAM FF-ICE R1**
## **Standalone Implementation Strategy**

---

## **OVERVIEW: 4 MAIN COMPONENTS**

```mermaid
graph TB
    subgraph "FF-ICE R1 Complete Solution"
        subgraph "Backend Infrastructure"
            DB[Database Layer<br/>PostgreSQL + Schema]
            FS[Filing Service<br/>Python FastAPI]
            QS[Query Service<br/>Python FastAPI]
        end
        
        subgraph "Client Tools"
            WEB[Web Test Client<br/>Next.js Dashboard]
            SDK[Python SDK<br/>Airlines Integration]
            CLI[CLI Tools<br/>Testing & Demo]
        end
        
        subgraph "Future SWIM Integration"
            SWIM[SWIM Compatibility Layer<br/>Easy migration path]
        end
    end
    
    DB --> FS
    DB --> QS
    WEB --> FS
    WEB --> QS
    SDK --> FS
    SDK --> QS
    CLI --> FS
    CLI --> QS
    
    SWIM -.->|Future Config Change| FS
    SWIM -.->|Future Config Change| QS
```

---

## **TEAM STRUCTURE & ASSIGNMENTS**

### **Team A: Database Infrastructure (2 people)**
```
Database Administrator + Backend Engineer
Timeline: 2-3 weeks
Budget: $50K
```

### **Team B: Filing Service (2-3 people)**
```
Senior Developer + Backend Developer + QA
Timeline: 6-8 weeks  
Budget: $150K
```

### **Team C: Query Service (2-3 people)**
```
Senior Developer + Backend Developer + QA
Timeline: 4-6 weeks
Budget: $120K
```

### **Team D: Client Tools (2 people)**
```
Frontend Developer + SDK Developer
Timeline: 6-8 weeks
Budget: $100K
```

---

## **1. DATABASE LAYER (Team A)**

### **1.1 Core Database Setup:**
```sql
-- Flight Plans Database Schema
CREATE DATABASE ffice_flight_plans;

CREATE TABLE flight_plans (
    id SERIAL PRIMARY KEY,
    gufi VARCHAR(20) UNIQUE NOT NULL,
    flight_number VARCHAR(10) NOT NULL,
    airline VARCHAR(50) NOT NULL,
    aircraft_type VARCHAR(10),
    registration VARCHAR(15),
    departure_airport VARCHAR(4) NOT NULL,
    arrival_airport VARCHAR(4) NOT NULL,
    departure_time TIMESTAMP WITH TIME ZONE NOT NULL,
    arrival_time TIMESTAMP WITH TIME ZONE,
    route_text TEXT,
    fixm_data JSONB NOT NULL,
    filing_status VARCHAR(20) DEFAULT 'ACCEPTED',
    filed_by VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Performance indexes
CREATE INDEX idx_gufi ON flight_plans(gufi);
CREATE INDEX idx_flight_number_date ON flight_plans(flight_number, DATE(departure_time));
CREATE INDEX idx_airline ON flight_plans(airline);
CREATE INDEX idx_route ON flight_plans(departure_airport, arrival_airport);
CREATE INDEX idx_departure_time ON flight_plans(departure_time);
CREATE INDEX idx_status ON flight_plans(filing_status);

-- Authentication table
CREATE TABLE api_clients (
    id SERIAL PRIMARY KEY,
    client_id VARCHAR(50) UNIQUE NOT NULL,
    client_name VARCHAR(100) NOT NULL,
    api_key VARCHAR(255) NOT NULL,
    permissions TEXT[] DEFAULT ARRAY['flight_query', 'flight_filing'],
    active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### **1.2 Database Utilities:**
```python
# database/connection.py
import psycopg2
from psycopg2.extras import RealDictCursor
import os

class DatabaseConnection:
    def __init__(self):
        self.connection_string = os.getenv(
            'DATABASE_URL', 
            'postgresql://ffice_user:password@localhost:5432/ffice_flight_plans'
        )
    
    def get_connection(self):
        return psycopg2.connect(
            self.connection_string,
            cursor_factory=RealDictCursor
        )
    
    def execute_query(self, query, params=None):
        with self.get_connection() as conn:
            with conn.cursor() as cursor:
                cursor.execute(query, params)
                return cursor.fetchall()
```

### **1.3 SWIM-Ready Architecture:**
```python
# Future SWIM migration: chỉ cần thay đổi connection config
class DatabaseConfig:
    # Standalone mode
    STANDALONE_DB = "postgresql://localhost:5432/ffice_flight_plans"
    
    # Future SWIM mode  
    SWIM_DB = "postgresql://swim-infrastructure:5432/swim_shared_db"
    
    @classmethod
    def get_database_url(cls):
        if os.getenv('SWIM_MODE') == 'true':
            return cls.SWIM_DB
        return cls.STANDALONE_DB
```

---

## **2. FILING SERVICE (Team B)**

### **2.1 Core Implementation:**
```python
# filing_service/main.py
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import HTTPBearer
import xml.etree.ElementTree as ET
from datetime import datetime
import uuid

app = FastAPI(title="VATM FF-ICE Filing Service")
security = HTTPBearer()

class FilingService:
    def __init__(self):
        self.db = DatabaseConnection()
    
    def validate_efpl(self, efpl_xml: str) -> dict:
        """Validate FIXM eFPL format"""
        try:
            root = ET.fromstring(efpl_xml)
            
            # Extract mandatory fields
            gufi = root.find('.//{http://www.fixm.aero/flight/4.2}gufi')
            flight_number = root.find('.//{http://www.fixm.aero/flight/4.2}flightNumber')
            departure = root.find('.//{http://www.fixm.aero/flight/4.2}aerodrome')
            
            if not all([gufi, flight_number, departure]):
                raise ValueError("Missing mandatory fields")
                
            return {
                'gufi': gufi.text,
                'flight_number': flight_number.text,
                'departure_airport': departure.text,
                'valid': True
            }
        except Exception as e:
            return {'valid': False, 'error': str(e)}
    
    def generate_gufi(self, flight_number: str, date: str = None) -> str:
        """Generate GUFI if not provided"""
        if not date:
            date = datetime.now().strftime('%Y%m%d')
        sequence = str(uuid.uuid4().int)[:3].zfill(3)
        return f"{flight_number}-{date}-{sequence}"
    
    def store_flight_plan(self, efpl_data: dict, airline: str) -> str:
        """Store eFPL in database"""
        query = """
        INSERT INTO flight_plans (
            gufi, flight_number, airline, fixm_data, filed_by
        ) VALUES (%s, %s, %s, %s, %s)
        RETURNING gufi
        """
        
        result = self.db.execute_query(query, [
            efpl_data['gufi'],
            efpl_data['flight_number'], 
            airline,
            efpl_data['fixm_xml'],
            airline
        ])
        
        return result[0]['gufi']

@app.post("/ffice/filing/submit")
async def file_flight_plan(
    efpl_xml: str,
    airline: str = Depends(authenticate_client)
):
    """Submit eFPL for filing"""
    filing_service = FilingService()
    
    # Validate eFPL
    validation = filing_service.validate_efpl(efpl_xml)
    if not validation['valid']:
        return {
            "status": "REJECTED",
            "error_code": "FPF001",
            "error_message": validation['error'],
            "timestamp": datetime.now().isoformat()
        }
    
    # Generate GUFI if needed
    if not validation.get('gufi'):
        validation['gufi'] = filing_service.generate_gufi(
            validation['flight_number']
        )
    
    # Store in database
    try:
        gufi = filing_service.store_flight_plan({
            'gufi': validation['gufi'],
            'flight_number': validation['flight_number'],
            'fixm_xml': efpl_xml
        }, airline)
        
        return {
            "status": "ACCEPTED", 
            "gufi": gufi,
            "timestamp": datetime.now().isoformat()
        }
        
    except Exception as e:
        return {
            "status": "REJECTED",
            "error_code": "FPF007", 
            "error_message": str(e),
            "timestamp": datetime.now().isoformat()
        }
```

### **2.2 SWIM Migration Path:**
```python
# Future SWIM integration: chỉ cần add endpoint
@app.post("/swim/filing/submit")  # SWIM-compatible endpoint
async def file_flight_plan_swim(
    efpl_xml: str,
    client_cert: dict = Depends(validate_swim_certificate)  # Future SWIM auth
):
    # Same logic, different authentication
    return await file_flight_plan(efpl_xml, client_cert['client_id'])
```

---

## **3. QUERY SERVICE (Team C)**

### **3.1 Core Implementation:**
```python
# query_service/main.py
from fastapi import FastAPI, HTTPException, Query
from typing import Optional, List
from datetime import datetime

app = FastAPI(title="VATM FF-ICE Query Service")

class QueryService:
    def __init__(self):
        self.db = DatabaseConnection()
    
    def get_flight_by_gufi(self, gufi: str, airline: str = None) -> dict:
        """Query flight by GUFI"""
        query = """
        SELECT * FROM flight_plans WHERE gufi = %s
        """
        params = [gufi]
        
        # Add airline filter for security
        if airline:
            query += " AND airline = %s"
            params.append(airline)
            
        result = self.db.execute_query(query, params)
        return result[0] if result else None
    
    def search_flights(self, 
                      flight_number: str = None,
                      date: str = None,
                      departure: str = None,
                      arrival: str = None,
                      airline: str = None) -> List[dict]:
        """Search flights by criteria"""
        
        query = "SELECT * FROM flight_plans WHERE 1=1"
        params = []
        
        if flight_number:
            query += " AND flight_number = %s"
            params.append(flight_number)
        
        if date:
            query += " AND DATE(departure_time) = %s"
            params.append(date)
            
        if departure:
            query += " AND departure_airport = %s"
            params.append(departure)
            
        if arrival:
            query += " AND arrival_airport = %s"
            params.append(arrival)
            
        if airline:
            query += " AND airline = %s"
            params.append(airline)
            
        query += " ORDER BY departure_time"
        
        return self.db.execute_query(query, params)

@app.get("/ffice/query/flight/{gufi}")
async def query_flight_by_gufi(
    gufi: str,
    airline: str = Depends(authenticate_client)
):
    """Query flight by GUFI"""
    query_service = QueryService()
    
    flight = query_service.get_flight_by_gufi(gufi, airline)
    
    if not flight:
        raise HTTPException(
            status_code=404,
            detail={
                "error_code": "FPQ002",
                "error_message": "Flight not found",
                "gufi": gufi
            }
        )
    
    return {
        "gufi": flight['gufi'],
        "flight_number": flight['flight_number'],
        "status": flight['filing_status'],
        "fixm_data": flight['fixm_data'],
        "timestamp": datetime.now().isoformat()
    }

@app.get("/ffice/query/search")
async def search_flights(
    flight_number: Optional[str] = Query(None),
    date: Optional[str] = Query(None),
    departure: Optional[str] = Query(None),
    arrival: Optional[str] = Query(None),
    airline: str = Depends(authenticate_client)
):
    """Search flights"""
    query_service = QueryService()
    
    flights = query_service.search_flights(
        flight_number=flight_number,
        date=date,
        departure=departure,
        arrival=arrival,
        airline=airline
    )
    
    return {
        "total": len(flights),
        "flights": flights,
        "timestamp": datetime.now().isoformat()
    }
```

---

## **4. CLIENT TOOLS (Team D)**

### **4.1 Python SDK:**
```python
# client_tools/python_sdk/vatm_ffice_client.py
import requests
from typing import Optional, List, Dict

class VATMFFICEClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/xml'
        })
    
    def file_flight_plan(self, efpl_xml: str) -> Dict:
        """File eFPL with VATM"""
        response = self.session.post(
            f'{self.base_url}/ffice/filing/submit',
            data=efpl_xml
        )
        response.raise_for_status()
        return response.json()
    
    def query_flight(self, gufi: str) -> Optional[Dict]:
        """Query flight by GUFI"""
        response = self.session.get(
            f'{self.base_url}/ffice/query/flight/{gufi}'
        )
        
        if response.status_code == 404:
            return None
            
        response.raise_for_status()
        return response.json()
    
    def search_flights(self, **criteria) -> List[Dict]:
        """Search flights"""
        response = self.session.get(
            f'{self.base_url}/ffice/query/search',
            params=criteria
        )
        response.raise_for_status()
        return response.json()['flights']

# Future SWIM compatibility
class SWIMFFICEClient(VATMFFICEClient):
    def __init__(self, base_url: str, certificate_path: str):
        # SWIM authentication với certificates
        super().__init__(base_url, None)
        self.session.cert = certificate_path
        # Override endpoints for SWIM
        self.filing_endpoint = '/swim/filing/submit'
        self.query_endpoint = '/swim/query'
```

### **4.2 Web Test Client:**
```javascript
// client_tools/web_client/components/FFICETestDashboard.jsx
import { useState } from 'react';

export default function FFICETestDashboard() {
  const [apiEndpoint, setApiEndpoint] = useState('http://localhost:8000');
  const [apiKey, setApiKey] = useState('test_api_key');
  
  // Filing test section
  const [eflpXml, setEflpXml] = useState('');
  const [filingResult, setFilingResult] = useState(null);
  
  // Query test section  
  const [queryGufi, setQueryGufi] = useState('');
  const [queryResult, setQueryResult] = useState(null);

  const testFiling = async () => {
    try {
      const response = await fetch(`${apiEndpoint}/ffice/filing/submit`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/xml'
        },
        body: eflpXml
      });
      
      const result = await response.json();
      setFilingResult(result);
    } catch (error) {
      setFilingResult({ error: error.message });
    }
  };

  const testQuery = async () => {
    try {
      const response = await fetch(
        `${apiEndpoint}/ffice/query/flight/${queryGufi}`,
        {
          headers: { 'Authorization': `Bearer ${apiKey}` }
        }
      );
      
      const result = await response.json();
      setQueryResult(result);
    } catch (error) {
      setQueryResult({ error: error.message });
    }
  };

  return (
    <div className="ffice-dashboard">
      <h1>FF-ICE R1 Test Dashboard</h1>
      
      {/* Configuration */}
      <div className="config-section">
        <input 
          placeholder="API Endpoint" 
          value={apiEndpoint}
          onChange={(e) => setApiEndpoint(e.target.value)}
        />
        <input 
          placeholder="API Key" 
          value={apiKey}
          onChange={(e) => setApiKey(e.target.value)}
        />
      </div>

      {/* Filing Test */}
      <div className="filing-section">
        <h2>Filing Service Test</h2>
        <textarea
          rows={10}
          cols={80}
          value={eflpXml}
          onChange={(e) => setEflpXml(e.target.value)}
          placeholder="Paste FIXM eFPL XML here..."
        />
        <button onClick={testFiling}>Submit Flight Plan</button>
        
        {filingResult && (
          <pre>{JSON.stringify(filingResult, null, 2)}</pre>
        )}
      </div>

      {/* Query Test */}
      <div className="query-section">
        <h2>Query Service Test</h2>
        <input
          value={queryGufi}
          onChange={(e) => setQueryGufi(e.target.value)}
          placeholder="Enter GUFI to query..."
        />
        <button onClick={testQuery}>Query Flight</button>
        
        {queryResult && (
          <pre>{JSON.stringify(queryResult, null, 2)}</pre>
        )}
      </div>
    </div>
  );
}
```

---

## **5. DEPLOYMENT & MIGRATION STRATEGY**

### **5.1 Standalone Deployment:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  database:
    image: postgres:15
    environment:
      POSTGRES_DB: ffice_flight_plans
      POSTGRES_USER: ffice_user
      POSTGRES_PASSWORD: secure_password
    ports:
      - "5432:5432"
      
  filing-service:
    build: ./filing_service
    ports:
      - "8001:8000"
    environment:
      DATABASE_URL: postgresql://ffice_user:secure_password@database:5432/ffice_flight_plans
      
  query-service:
    build: ./query_service  
    ports:
      - "8002:8000"
    environment:
      DATABASE_URL: postgresql://ffice_user:secure_password@database:5432/ffice_flight_plans
      
  web-client:
    build: ./client_tools/web_client
    ports:
      - "3000:3000"
```

### **5.2 SWIM Migration (Future):**
```bash
# Migration script
#!/bin/bash
echo "Migrating to SWIM mode..."

# 1. Update environment variables
export SWIM_MODE=true
export DATABASE_URL=postgresql://swim-infrastructure:5432/swim_shared_db
export AUTHENTICATION_MODE=swim_certificates

# 2. Deploy SWIM-compatible endpoints
docker-compose -f docker-compose.swim.yml up -d

# 3. Register services with SWIM registry
python scripts/register_with_swim.py

echo "Migration to SWIM completed!"
```

---

## **TIMELINE & BUDGET SUMMARY**

### **Development Timeline:**
```
Week 1-3:   Database setup (Team A)
Week 2-9:   Filing Service (Team B) 
Week 4-9:   Query Service (Team C)
Week 3-10:  Client Tools (Team D)
Week 11-12: Integration testing
Week 13-14: Deployment & documentation

Total: 14 weeks (3.5 months)
```

### **Budget Breakdown:**
```
Team A (Database):     $50K
Team B (Filing):       $150K  
Team C (Query):        $120K
Team D (Client):       $100K
Infrastructure:        $30K
Testing & Deployment:  $50K

Total: $500K
```

### **SWIM Migration Cost (Future):**
```
Configuration changes: $20K
SWIM integration:      $50K  
Testing & validation:  $30K

Migration Total: $100K (20% of original project)
```

**KEY SUCCESS FACTORS:**
- ✅ **Standalone first**: Immediate value, faster deployment
- ✅ **SWIM-ready architecture**: Easy future migration
- ✅ **Client tools**: Professional airline onboarding
- ✅ **Complete solution**: Database + Services + Clients
- ✅ **Cost-effective**: $500K vs $2M+ SWIM-first approach

**DELIVERABLES:**
1. Working FF-ICE Filing & Query services
2. Complete client SDK for airlines
3. Web-based testing dashboard
4. Sample data và documentation
5. Clear migration path to SWIM
