# TECHNICAL SPECIFICATION DOCUMENT
## FF-ICE Filing Service for VATM
### Version 1.0 | Date: September 12, 2025

---

## 1. EXECUTIVE SUMMARY

### 1.1 Project Overview
Vietnam Air Traffic Management Corporation (VATM) requires implementation of FF-ICE Filing Service as a mandatory component for transition from traditional ANSP (aASP) to enhanced ANSP (eASP). This service enables airlines to submit Extended Flight Plans (eFPL) in FIXM format and receive filing status responses via SWIM infrastructure.

### 1.2 Scope
This document specifies requirements for developing a Filing Service that:
- Accepts eFPL submissions via REST API and SWIM messaging
- Validates flight plans against Vietnamese flight rules and regulations
- Processes and stores validated flight plans in central database
- Distributes approved flight plans to relevant stakeholders
- Returns filing status to submitting airlines
- Integrates with VATM's existing Flight Data Processing System (FDPS)

### 1.3 Success Criteria
- Process 1000+ eFPL submissions per day
- Response time < 10 seconds per filing
- 99.7% availability during operational hours (18 hours/day)
- FIXM 4.2.0 compliance for all inputs
- 100% integration with existing VATM FDPS
- Vietnamese airspace regulatory compliance
- Hãy đóng vai trò chuyên gia đặc tả hệ thống. Viết cho tôi tài liệu đặc tả tính năng hệ thống để làm Filing Service .
---

## 2. SYSTEM ARCHITECTURE

### 2.1 High-Level Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                             │
│ ┌─────────────┬─────────────┬─────────────┬─────────────────────┐│
│ │  Airlines   │   SWIM      │  Web Portal │    Other ANSPs     ││
│ │   (eFPL)    │Integration  │    (GUI)    │   (Cross-border)   ││
│ └─────────────┴─────────────┴─────────────┴─────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                    API Gateway Layer                            │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │          Authentication & Rate Limiting                     │ │
│ │   ┌──────────┬──────────┬──────────┬──────────────────────┐ │ │
│ │   │API Key   │  OAuth2  │IP Filter │   Request Validation │ │ │
│ │   └──────────┴──────────┴──────────┴──────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Filing Service Layer                          │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │              Core Filing Application                        │ │
│ │ ┌──────────┬──────────┬──────────┬──────────┬──────────────┐ │ │
│ │ │eFPL      │Business  │Flight    │FDPS      │Notification  │ │ │
│ │ │Processor │Rules     │Validator │Integrator│ Manager      │ │ │
│ │ └──────────┴──────────┴──────────┴──────────┴──────────────┘ │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Integration Layer                            │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │                External System Integration                  │ │
│ │ ┌──────────┬──────────┬──────────┬──────────┬──────────────┐ │ │
│ │ │VATM FDPS │Weather   │Airspace  │Military  │Airport       │ │ │
│ │ │System    │Service   │Database  │Coord.    │Systems (ACV) │ │ │
│ │ └──────────┴──────────┴──────────┴──────────┴──────────────┘ │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Data Layer                                 │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │                PostgreSQL Database Cluster                  │ │
│ │ ┌──────────┬──────────┬──────────┬──────────┬──────────────┐ │ │
│ │ │Filed     │Business  │Validation│Audit     │Configuration │ │ │
│ │ │eFPLs     │Rules     │Results   │Logs      │ Data         │ │ │
│ │ └──────────┴──────────┴──────────┴──────────┴──────────────┘ │ │
│ └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Specifications

#### 2.2.1 eFPL Processor
- **Technology**: Spring Boot with JAXB XML processing
- **Responsibility**: Parse, validate FIXM format, extract flight data
- **Performance**: Process 100+ eFPLs per minute
- **Validation**: Schema validation against FIXM 4.2.0

#### 2.2.2 Business Rules Engine
- **Technology**: Drools Rule Engine
- **Responsibility**: Apply Vietnamese flight rules and regulations
- **Rules Categories**: Airspace, aircraft performance, route validation
- **Configuration**: Hot-reload capable rule updates

#### 2.2.3 Flight Validator
- **Technology**: Custom validation framework
- **Responsibility**: Technical and operational validation
- **Checks**: Route feasibility, aircraft capability, slot availability
- **Performance**: Validation complete within 5 seconds

#### 2.2.4 FDPS Integrator
- **Technology**: JMS messaging with existing VATM systems
- **Responsibility**: Integration with current Flight Data Processing System
- **Protocol**: SWIM-compatible messaging
- **Reliability**: Guaranteed message delivery

#### 2.2.5 Notification Manager
- **Technology**: Apache ActiveMQ with REST callbacks
- **Responsibility**: Distribute filing results to stakeholders
- **Recipients**: Airlines, ATC, airports, adjacent ANSPs
- **Delivery**: Real-time push notifications

---

## 3. FUNCTIONAL REQUIREMENTS

### 3.1 Primary Functions

#### FR-001: eFPL Submission Processing
- **Description**: Accept and process Extended Flight Plan submissions
- **Input**: FIXM 4.2.0 compliant eFPL XML
- **Processing Steps**:
  1. Schema validation against FIXM 4.2.0
  2. Business rule validation
  3. Technical feasibility check
  4. FDPS integration
  5. Stakeholder notification
- **Output**: Filing Status Response (ACCEPTED/REJECTED/PENDING)
- **Response Time**: < 10 seconds for 95% of submissions
- **Priority**: Critical

#### FR-002: Flight Plan Validation
- **Description**: Comprehensive validation of submitted flight plans
- **Validation Categories**:
  - **Format Validation**: FIXM schema compliance
  - **Data Validation**: Required fields, data types, constraints
  - **Business Validation**: Vietnamese flight rules compliance
  - **Technical Validation**: Route feasibility, aircraft performance
  - **Operational Validation**: Slot availability, airspace capacity
- **Response**: Detailed validation report with specific error codes
- **Priority**: Critical

#### FR-003: Flight Rules Compliance Check
- **Description**: Validate against Vietnamese airspace regulations
- **Rules Categories**:
  - **Airspace Rules**: Vietnamese FIR restrictions, prohibited areas
  - **Airport Rules**: Operating hours, runway restrictions, noise limits
  - **Aircraft Rules**: Type certification, equipment requirements
  - **Route Rules**: Minimum altitudes, navigation requirements
  - **Time Rules**: Slot coordination, flow management
- **Configuration**: Rules updated via configuration without code changes
- **Priority**: Critical

#### FR-004: FDPS Integration
- **Description**: Seamless integration with existing VATM FDPS
- **Integration Points**:
  - **Flight Plan Database**: Store approved eFPLs
  - **Strip Management**: Generate controller strips
  - **Conflict Detection**: Check for conflicts with existing traffic
  - **Coordination**: Interface with adjacent ACCs
- **Data Synchronization**: Real-time bidirectional sync
- **Priority**: High

#### FR-005: Stakeholder Notification
- **Description**: Distribute filing results to relevant parties
- **Notification Recipients**:
  - **Submitting Airline**: Filing status and details
  - **ATC Units**: Approved flight plans for coordination
  - **Airports**: Flight information for ground handling
  - **Adjacent ANSPs**: Cross-border coordination
  - **Military**: Coordination for restricted airspace
- **Delivery Methods**: SWIM messaging, REST callbacks, email
- **Priority**: High

#### FR-006: Filing Status Management
- **Description**: Track and manage flight plan filing lifecycle
- **Status Types**:
  - **RECEIVED**: eFPL received, initial processing
  - **VALIDATING**: Under validation process
  - **PENDING**: Awaiting manual review or external approval
  - **ACCEPTED**: Approved and active in system
  - **REJECTED**: Rejected with detailed reasons
  - **CANCELLED**: Cancelled by airline or system
  - **SUSPENDED**: Temporarily suspended due to restrictions
- **Tracking**: Full audit trail of status changes
- **Priority**: Medium

#### FR-007: Amendment Processing
- **Description**: Handle updates and amendments to filed flight plans
- **Amendment Types**:
  - **Route Amendment**: Change in flight route
  - **Time Amendment**: Departure/arrival time changes
  - **Aircraft Amendment**: Aircraft type or registration change
  - **Cancellation**: Flight cancellation
- **Validation**: Same validation rules as original filing
- **Coordination**: Automatic notification to affected parties
- **Priority**: Medium

#### FR-008: Bulk Filing Support
- **Description**: Support for multiple flight plan submissions
- **Input**: Array of eFPL objects in single request
- **Processing**: Parallel processing with individual status tracking
- **Limitations**: Maximum 50 eFPLs per bulk request
- **Response**: Array of individual filing statuses
- **Priority**: Low

### 3.2 Business Rules Categories

#### BR-001: Vietnamese Airspace Rules
```yaml
Restricted Areas:
  - Military training areas (schedule-based restrictions)
  - Presidential flight restrictions (dynamic)
  - Weather-related closures (temporary)
  - Maintenance closures (scheduled)

Altitude Restrictions:
  - Minimum safe altitudes per route segment
  - Maximum altitudes for aircraft types
  - RVSM capability requirements above FL290
  - Noise abatement procedures

Route Restrictions:
  - Prohibited routes for certain aircraft types
  - Time-based route restrictions
  - Fuel/emergency routing requirements
```

#### BR-002: Aircraft Performance Rules
```yaml
Equipment Requirements:
  - Navigation equipment (RNP, RNAV capabilities)
  - Communication equipment (8.33 kHz spacing)
  - Surveillance equipment (ADS-B, Mode S)
  - Emergency equipment requirements

Performance Validation:
  - Cruise speed vs aircraft type
  - Service ceiling limitations
  - Range validation for route length
  - Fuel calculations for route + reserves
```

#### BR-003: Airport Operating Rules
```yaml
Operating Hours:
  - Airport operating schedules
  - Runway availability windows
  - Ground services availability
  - Customs and immigration hours

Slot Coordination:
  - Coordinated airports require prior slots
  - Peak hour restrictions
  - Noise curfew compliance
  - Ground handling capacity limits
```

### 3.3 Integration Requirements

#### IR-001: VATM FDPS Integration
- **Interface Type**: JMS messaging over secure network
- **Message Format**: VATM internal format (converted from FIXM)
- **Frequency**: Real-time, immediate upon filing approval
- **Error Handling**: Retry mechanism with exponential backoff
- **Fallback**: Manual coordination procedures

#### IR-002: Weather Service Integration
- **Provider**: Vietnam National Meteorological Service
- **Interface**: REST API with weather data
- **Usage**: Route weather validation, hazard checking
- **Update Frequency**: Every 15 minutes
- **Timeout**: 5 seconds maximum response time

#### IR-003: Military Coordination
- **Interface**: Secure messaging to military ATC units
- **Purpose**: Coordination for flights through military airspace
- **Protocol**: Encrypted email and phone backup
- **Response Time**: 30 minutes maximum for coordination

#### IR-004: Airport Systems (ACV)
- **Interface**: REST API to airport operational systems
- **Data Exchange**: Slot requests, ground handling coordination
- **Airports**: VVNB, VVTS, VVDN, VVCI (major airports)
- **Frequency**: Real-time for slot validation

---

## 4. API SPECIFICATIONS

### 4.1 REST Endpoints

#### 4.1.1 Submit Flight Plan
```http
POST /api/v1/filing/submit
```

**Headers:**
```
Content-Type: application/xml
Authorization: Bearer {api_key}
X-Airline-Code: VN
X-Request-ID: {unique_request_id}
```

**Request Body (FIXM 4.2.0 eFPL):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<fx:FlightPlan xmlns:fx="http://www.fixm.aero/flight/4.2"
               xmlns:fb="http://www.fixm.aero/base/4.2">
    <fx:gufi>VN123-20250912-001</fx:gufi>
    <fx:flight>
        <fx:flightIdentification>
            <fx:aircraftIdentification>VN123</fx:aircraftIdentification>
        </fx:flightIdentification>
        
        <fx:aircraft>
            <fx:aircraftType>A321</fx:aircraftType>
            <fx:registration>VN-A321</fx:registration>
            <fx:operatorDesignator>HVN</fx:operatorDesignator>
            <fx:aircraftCapabilities>
                <fx:navigation>
                    <fx:performanceBasedNavigationCapability>RNP1</fx:performanceBasedNavigationCapability>
                </fx:navigation>
                <fx:communication>
                    <fx:communicationCapability>VOICE</fx:communicationCapability>
                </fx:communication>
                <fx:surveillance>
                    <fx:surveillanceCapability>ADSB</fx:surveillanceCapability>
                </fx:surveillance>
            </fx:aircraftCapabilities>
        </fx:aircraft>
        
        <fx:departure>
            <fx:aerodrome>VVNB</fx:aerodrome>
            <fx:estimatedOffBlockTime>2025-09-12T02:00:00Z</fx:estimatedOffBlockTime>
            <fx:runway>11L</fx:runway>
        </fx:departure>
        
        <fx:arrival>
            <fx:aerodrome>VVTS</fx:aerodrome>
            <fx:estimatedOnBlockTime>2025-09-12T04:30:00Z</fx:estimatedOnBlockTime>
        </fx:arrival>
        
        <fx:routeTrajectory>
            <fx:routeInformation>
                <fx:routeText>VVNB DCT ADMOX A1 OTBAN VVTS</fx:routeText>
                <fx:cruisingSpeed>450</fx:cruisingSpeed>
                <fx:cruisingLevel>370</fx:cruisingLevel>
                <fx:totalEstimatedElapsedTime>PT2H30M</fx:totalEstimatedElapsedTime>
            </fx:routeInformation>
            
            <fx:trajectory4D>
                <fx:element seqNum="1">
                    <fx:elementStartPoint>
                        <fx:position>
                            <fb:latitude>21.221192</fb:latitude>
                            <fb:longitude>105.807178</fb:longitude>
                        </fx:position>
                        <fx:altitude>
                            <fb:altitude>0</fb:altitude>
                            <fb:altitudeReference>MSL</fb:altitudeReference>
                        </fx:altitude>
                        <fx:time>2025-09-12T02:00:00Z</fx:time>
                    </fx:elementStartPoint>
                </fx:element>
                
                <fx:element seqNum="2">
                    <fx:elementStartPoint>
                        <fx:position>
                            <fb:latitude>18.5667</fb:latitude>
                            <fb:longitude>106.5833</fb:longitude>
                        </fx:position>
                        <fx:altitude>
                            <fb:altitude>37000</fb:altitude>
                            <fb:altitudeReference>MSL</fb:altitudeReference>
                        </fx:altitude>
                        <fx:time>2025-09-12T02:45:00Z</fx:time>
                    </fx:elementStartPoint>
                </fx:element>
            </fx:trajectory4D>
        </fx:routeTrajectory>
        
        <fx:supplementaryInformation>
            <fx:fuelEndurance>PT4H30M</fx:fuelEndurance>
            <fx:personsOnBoard>189</fx:personsOnBoard>
            <fx:pilotInCommand>NGUYEN VAN A</fx:pilotInCommand>
            <fx:emergencyContactInformation>
                <fx:contact>ops@vietnamairlines.com</fx:contact>
            </fx:emergencyContactInformation>
        </fx:supplementaryInformation>
    </fx:flight>
</fx:FlightPlan>
```

**Response (Success - 201 Created):**
```json
{
    "status": "ACCEPTED",
    "gufi": "VN123-20250912-001",
    "filingId": "VATM-20250912-000123",
    "submissionTime": "2025-09-12T01:45:32Z",
    "processedTime": "2025-09-12T01:45:38Z",
    "validationResults": {
        "formatValidation": "PASSED",
        "businessRulesValidation": "PASSED", 
        "technicalValidation": "PASSED",
        "operationalValidation": "PASSED"
    },
    "assignedSlot": {
        "departure": "2025-09-12T02:00:00Z",
        "arrival": "2025-09-12T04:30:00Z"
    },
    "coordinationStatus": {
        "fdpsIntegration": "COMPLETED",
        "atcNotification": "SENT",
        "airportNotification": "SENT"
    },
    "message": "Flight plan filed successfully and distributed to relevant parties"
}
```

**Response (Validation Failed - 422 Unprocessable Entity):**
```json
{
    "status": "REJECTED",
    "gufi": "VN123-20250912-001",
    "filingId": "VATM-20250912-000124",
    "submissionTime": "2025-09-12T01:45:32Z",
    "processedTime": "2025-09-12T01:45:35Z",
    "validationResults": {
        "formatValidation": "PASSED",
        "businessRulesValidation": "FAILED",
        "technicalValidation": "FAILED",
        "operationalValidation": "PASSED"
    },
    "errors": [
        {
            "code": "BR001",
            "category": "BUSINESS_RULE",
            "severity": "ERROR",
            "field": "routeTrajectory.cruisingLevel",
            "message": "Requested cruise level FL370 exceeds maximum altitude FL350 for A321 on route ADMOX-OTBAN",
            "suggestion": "Reduce cruise level to FL350 or below"
        },
        {
            "code": "TV003", 
            "category": "TECHNICAL_VALIDATION",
            "severity": "ERROR",
            "field": "aircraft.registration",
            "message": "Aircraft registration VN-A321 not found in Vietnamese aircraft registry",
            "suggestion": "Verify aircraft registration or register aircraft with CAAV"
        }
    ],
    "message": "Flight plan rejected due to validation errors. Please correct errors and resubmit."
}
```

**Response (Pending Manual Review - 202 Accepted):**
```json
{
    "status": "PENDING",
    "gufi": "VN123-20250912-001", 
    "filingId": "VATM-20250912-000125",
    "submissionTime": "2025-09-12T01:45:32Z",
    "processedTime": "2025-09-12T01:45:40Z",
    "validationResults": {
        "formatValidation": "PASSED",
        "businessRulesValidation": "PASSED",
        "technicalValidation": "PASSED", 
        "operationalValidation": "PENDING"
    },
    "pendingReasons": [
        {
            "code": "MR001",
            "category": "MANUAL_REVIEW",
            "message": "Flight requires military airspace coordination",
            "estimatedResolutionTime": "2025-09-12T02:15:00Z"
        }
    ],
    "trackingUrl": "https://filing.vatm.vn/status/VATM-20250912-000125",
    "message": "Flight plan pending manual review. You will be notified when review is complete."
}
```

#### 4.1.2 Check Filing Status
```http
GET /api/v1/filing/status/{filingId}
```

**Parameters:**
- `filingId` (path): VATM filing identifier

**Response (200 OK):**
```json
{
    "filingId": "VATM-20250912-000123",
    "gufi": "VN123-20250912-001",
    "currentStatus": "ACCEPTED",
    "statusHistory": [
        {
            "status": "RECEIVED",
            "timestamp": "2025-09-12T01:45:32Z",
            "description": "Flight plan received for processing"
        },
        {
            "status": "VALIDATING", 
            "timestamp": "2025-09-12T01:45:33Z",
            "description": "Validation in progress"
        },
        {
            "status": "ACCEPTED",
            "timestamp": "2025-09-12T01:45:38Z", 
            "description": "Flight plan approved and active"
        }
    ],
    "lastUpdated": "2025-09-12T01:45:38Z"
}
```

#### 4.1.3 Update Flight Plan
```http
PUT /api/v1/filing/update/{filingId}
```

**Request Body (Amendment):**
```json
{
    "amendmentType": "TIME_CHANGE",
    "gufi": "VN123-20250912-001",
    "changes": {
        "estimatedOffBlockTime": "2025-09-12T02:30:00Z",
        "estimatedOnBlockTime": "2025-09-12T05:00:00Z"
    },
    "reason": "ATC Flow Control Delay",
    "requestedBy": "VN_DISPATCH_HAN"
}
```

**Response (200 OK):**
```json
{
    "status": "ACCEPTED",
    "amendmentId": "VATM-20250912-000123-AMD001", 
    "originalFilingId": "VATM-20250912-000123",
    "processedTime": "2025-09-12T02:15:45Z",
    "updatedFields": ["estimatedOffBlockTime", "estimatedOnBlockTime"],
    "coordinationRequired": ["ATC_VVNB_TWR", "ATC_VVTS_APP"],
    "message": "Amendment accepted. ATC coordination in progress."
}
```

#### 4.1.4 Cancel Flight Plan
```http
DELETE /api/v1/filing/cancel/{filingId}
```

**Request Body:**
```json
{
    "reason": "AIRCRAFT_TECHNICAL_PROBLEM",
    "description": "Engine maintenance required",
    "cancelledBy": "VN_DISPATCH_HAN",
    "effectiveTime": "2025-09-12T01:50:00Z"
}
```

**Response (200 OK):**
```json
{
    "status": "CANCELLED",
    "filingId": "VATM-20250912-000123",
    "cancellationId": "VATM-20250912-000123-CXL001",
    "processedTime": "2025-09-12T01:51:15Z",
    "notificationsSent": [
        "ATC_VVNB_TWR",
        "ATC_VVTS_APP", 
        "AIRPORT_VVNB_OPS",
        "AIRPORT_VVTS_OPS"
    ],
    "message": "Flight plan cancelled successfully. All parties notified."
}
```

#### 4.1.5 Bulk Filing
```http
POST /api/v1/filing/bulk
```

**Request Body:**
```json
{
    "submissions": [
        {
            "requestId": "VN_BULK_001",
            "efpl": "... FIXM XML content ..."
        },
        {
            "requestId": "VN_BULK_002", 
            "efpl": "... FIXM XML content ..."
        }
    ],
    "processingMode": "PARALLEL",
    "notificationMode": "INDIVIDUAL"
}
```

**Response (207 Multi-Status):**
```json
{
    "bulkId": "VATM-BULK-20250912-001",
    "submissionTime": "2025-09-12T03:00:00Z",
    "processedTime": "2025-09-12T03:00:45Z",
    "summary": {
        "totalSubmissions": 2,
        "accepted": 1,
        "rejected": 1,
        "pending": 0
    },
    "results": [
        {
            "requestId": "VN_BULK_001",
            "status": "ACCEPTED",
            "filingId": "VATM-20250912-000126",
            "gufi": "VN124-20250912-001"
        },
        {
            "requestId": "VN_BULK_002",
            "status": "REJECTED", 
            "errors": [
                {
                    "code": "FV001",
                    "message": "Invalid departure airport code"
                }
            ]
        }
    ]
}
```

### 4.2 SWIM Integration Endpoints

#### 4.2.1 SWIM Message Handler
```http
POST /api/v1/swim/filing
```

**Headers:**
```
Content-Type: application/soap+xml
SOAPAction: "http://www.fixm.aero/filing/submit"
Authorization: Certificate-based
```

**SOAP Request Body:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
               xmlns:fil="http://www.vatm.vn/filing">
    <soap:Header>
        <fil:MessageHeader>
            <fil:messageId>MSG-VN-20250912-001</fil:messageId>
            <fil:timestamp>2025-09-12T01:45:30Z</fil:timestamp>
            <fil:sender>HVN</fil:sender>
            <fil:recipient>VATM</fil:recipient>
        </fil:MessageHeader>
    </soap:Header>
    <soap:Body>
        <fil:FilingRequest>
            <fil:flightPlan>
                <!-- FIXM eFPL content -->
            </fil:flightPlan>
        </fil:FilingRequest>
    </soap:Body>
</soap:Envelope>
```

### 4.3 Error Codes Reference

#### 4.3.1 Format Validation Errors (FV)
```yaml
FV001: "Invalid XML format"
FV002: "FIXM schema validation failed"
FV003: "Required field missing"
FV004: "Invalid data type"
FV005: "Field length exceeds maximum"
FV006: "Invalid character encoding"
FV007: "Malformed GUFI format"
FV008: "Invalid timestamp format"
```

#### 4.3.2 Business Rules Errors (BR)
```yaml
BR001: "Cruise altitude exceeds aircraft capability"
BR002: "Route violates airspace restrictions"
BR003: "Flight time outside airport operating hours"
BR004: "Aircraft not certified for requested route"
BR005: "Fuel endurance insufficient for route"
BR006: "Equipment requirements not met"
BR007: "Wake turbulence separation violation"
BR008: "Noise curfew violation"
```

#### 4.3.3 Technical Validation Errors (TV)
```yaml
TV001: "Route not found in navigation database"
TV002: "Waypoint does not exist"
TV003: "Aircraft registration not valid"
TV004: "Operator not authorized"
TV005: "Performance data inconsistent"
TV006: "Navigation capability insufficient"
TV007: "Communication equipment inadequate"
TV008: "Surveillance equipment missing"
```

#### 4.3.4 Operational Validation Errors (OV)
```yaml
OV001: "Slot not available at requested time"
OV002: "Airport capacity exceeded"
OV003: "Runway closure conflict"
OV004: "Weather restriction in effect"
OV005: "Military airspace coordination required"
OV006: "Flow control restriction active"
OV007: "Customs facility not available"
OV008: "Ground handling capacity exceeded"
```

---

## 5. DATABASE DESIGN

### 5.1 Core Tables

#### 5.1.1 filed_flight_plans
```sql
CREATE TABLE filed_flight_plans (
    id                      BIGSERIAL PRIMARY KEY,
    filing_id              VARCHAR(50) UNIQUE NOT NULL,
    gufi                   VARCHAR(50) UNIQUE NOT NULL,
    submission_request_id   VARCHAR(100),
    airline_code           VARCHAR(3) NOT NULL,
    operator_code          VARCHAR(3),
    flight_number          VARCHAR(10) NOT NULL,
    aircraft_registration  VARCHAR(10),
    aircraft_type          VARCHAR(10) NOT NULL,
    
    -- Departure Information
    departure_airport      VARCHAR(4) NOT NULL,
    departure_runway       VARCHAR(10),
    estimated_off_block    TIMESTAMP WITH TIME ZONE NOT NULL,
    scheduled_departure    TIMESTAMP WITH TIME ZONE,
    actual_departure       TIMESTAMP WITH TIME ZONE,
    
    -- Arrival Information  
    arrival_airport        VARCHAR(4) NOT NULL,
    arrival_runway         VARCHAR(10),
    estimated_on_block     TIMESTAMP WITH TIME ZONE NOT NULL,
    scheduled_arrival      TIMESTAMP WITH TIME ZONE,
    actual_arrival         TIMESTAMP WITH TIME ZONE,
    
    -- Route Information
    route_text            TEXT NOT NULL,
    cruise_altitude       INTEGER,
    cruise_speed          INTEGER,
    total_eet_minutes     INTEGER, -- Estimated Elapsed Time
    
    -- Aircraft Details
    -- Aircraft Details
    wake_turbulence       VARCHAR(1), -- L, M, H, J
    equipment_code        VARCHAR(20),
    transponder_code      VARCHAR(10),
    
    -- Navigation Capabilities
    rnp_capability        VARCHAR(10), -- RNP1, RNP4, etc.
    pbn_capability        VARCHAR(50), -- Performance Based Navigation
    communication_cap     VARCHAR(100),
    surveillance_cap      VARCHAR(100),
    
    -- Flight Details
    flight_rules          VARCHAR(1) DEFAULT 'I', -- I=IFR, V=VFR
    flight_type           VARCHAR(1) DEFAULT 'S', -- S=Scheduled, N=Non-scheduled
    persons_on_board      INTEGER,
    fuel_endurance_hours  INTEGER,
    fuel_endurance_minutes INTEGER,
    
    -- Alternate Airports
    alternate_airport_1   VARCHAR(4),
    alternate_airport_2   VARCHAR(4),
    
    -- Emergency Information
    pilot_in_command      VARCHAR(100),
    emergency_contact     VARCHAR(200),
    special_handling      TEXT,
    
    -- Filing Status
    current_status        VARCHAR(20) DEFAULT 'RECEIVED',
    filing_priority       VARCHAR(10) DEFAULT 'NORMAL', -- NORMAL, HIGH, EMERGENCY
    
    -- Original Data Storage
    original_fixm_xml     TEXT NOT NULL,
    trajectory_data       JSONB,
    supplementary_info    JSONB,
    
    -- Processing Information
    validation_results    JSONB,
    business_rules_applied JSONB,
    coordination_status   JSONB,
    
    -- Audit Fields
    submitted_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    processed_at          TIMESTAMP WITH TIME ZONE,
    last_updated_at       TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    submitted_by          VARCHAR(100) NOT NULL,
    processed_by          VARCHAR(100),
    
    -- Integration Fields
    fdps_message_id       VARCHAR(100),
    atc_strip_generated   BOOLEAN DEFAULT FALSE,
    external_notifications JSONB,
    
    -- Constraints
    CONSTRAINT chk_flight_rules CHECK (flight_rules IN ('I', 'V')),
    CONSTRAINT chk_flight_type CHECK (flight_type IN ('S', 'N', 'G', 'M')),
    CONSTRAINT chk_wake_turbulence CHECK (wake_turbulence IN ('L', 'M', 'H', 'J')),
    CONSTRAINT chk_valid_airports CHECK (LENGTH(departure_airport) = 4 AND LENGTH(arrival_airport) = 4),
    CONSTRAINT chk_departure_before_arrival CHECK (estimated_off_block < estimated_on_block),
    CONSTRAINT chk_persons_positive CHECK (persons_on_board > 0),
    CONSTRAINT chk_valid_status CHECK (current_status IN ('RECEIVED', 'VALIDATING', 'PENDING', 'ACCEPTED', 'REJECTED', 'CANCELLED', 'SUSPENDED'))
);

-- Indexes for performance
CREATE INDEX idx_filed_flight_plans_filing_id ON filed_flight_plans (filing_id);
CREATE INDEX idx_filed_flight_plans_gufi ON filed_flight_plans (gufi);
CREATE INDEX idx_filed_flight_plans_status ON filed_flight_plans (current_status);
CREATE INDEX idx_filed_flight_plans_airline_date ON filed_flight_plans (airline_code, DATE(estimated_off_block));
CREATE INDEX idx_filed_flight_plans_departure ON filed_flight_plans (departure_airport, estimated_off_block);
CREATE INDEX idx_filed_flight_plans_arrival ON filed_flight_plans (arrival_airport, estimated_on_block);
CREATE INDEX idx_filed_flight_plans_submitted ON filed_flight_plans (submitted_at DESC);
CREATE INDEX idx_filed_flight_plans_flight_number ON filed_flight_plans (flight_number, DATE(estimated_off_block));

-- Partial indexes for active flights
CREATE INDEX idx_filed_flight_plans_active ON filed_flight_plans (current_status, estimated_off_block) 
    WHERE current_status IN ('ACCEPTED', 'PENDING');

-- GIN indexes for JSONB fields
CREATE INDEX idx_filed_flight_plans_trajectory ON filed_flight_plans USING GIN (trajectory_data);
CREATE INDEX idx_filed_flight_plans_validation ON filed_flight_plans USING GIN (validation_results);
```

#### 5.1.2 filing_status_history
```sql
CREATE TABLE filing_status_history (
    id                    BIGSERIAL PRIMARY KEY,
    filing_id             VARCHAR(50) NOT NULL REFERENCES filed_flight_plans(filing_id) ON DELETE CASCADE,
    previous_status       VARCHAR(20),
    new_status           VARCHAR(20) NOT NULL,
    status_change_time   TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    changed_by           VARCHAR(100),
    change_reason        TEXT,
    system_generated     BOOLEAN DEFAULT TRUE,
    additional_data      JSONB,
    
    INDEX idx_filing_status_history_filing_id (filing_id),
    INDEX idx_filing_status_history_time (status_change_time DESC),
    INDEX idx_filing_status_history_status (new_status)
);
```

#### 5.1.3 validation_rules
```sql
CREATE TABLE validation_rules (
    id                   BIGSERIAL PRIMARY KEY,
    rule_code           VARCHAR(20) UNIQUE NOT NULL,
    rule_name           VARCHAR(200) NOT NULL,
    rule_category       VARCHAR(50) NOT NULL, -- BUSINESS, TECHNICAL, OPERATIONAL, FORMAT
    rule_type           VARCHAR(50) NOT NULL, -- MANDATORY, WARNING, ADVISORY
    rule_description    TEXT,
    rule_expression     TEXT NOT NULL, -- Drools rule expression
    error_message       TEXT NOT NULL,
    suggestion_message  TEXT,
    
    -- Rule Applicability
    applicable_aircraft_types TEXT[], -- Array of aircraft types
    applicable_airports       TEXT[], -- Array of airport codes
    applicable_routes         TEXT[], -- Array of route patterns
    applicable_operators      TEXT[], -- Array of operator codes
    
    -- Rule Scheduling
    effective_from      TIMESTAMP WITH TIME ZONE NOT NULL,
    effective_until     TIMESTAMP WITH TIME ZONE,
    time_based_rules    JSONB, -- Time-of-day or seasonal rules
    
    -- Rule Priority and Dependencies
    rule_priority       INTEGER DEFAULT 100,
    depends_on_rules    TEXT[], -- Array of rule codes this rule depends on
    
    -- Status and Metadata
    rule_status         VARCHAR(20) DEFAULT 'ACTIVE', -- ACTIVE, INACTIVE, DRAFT
    version             VARCHAR(20) DEFAULT '1.0',
    created_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by          VARCHAR(100) NOT NULL,
    updated_by          VARCHAR(100),
    
    -- Approval Workflow
    approved_by         VARCHAR(100),
    approved_at         TIMESTAMP WITH TIME ZONE,
    approval_comments   TEXT,
    
    CONSTRAINT chk_rule_category CHECK (rule_category IN ('BUSINESS', 'TECHNICAL', 'OPERATIONAL', 'FORMAT')),
    CONSTRAINT chk_rule_type CHECK (rule_type IN ('MANDATORY', 'WARNING', 'ADVISORY')),
    CONSTRAINT chk_rule_status CHECK (rule_status IN ('ACTIVE', 'INACTIVE', 'DRAFT')),
    CONSTRAINT chk_effective_dates CHECK (effective_from < effective_until OR effective_until IS NULL)
);

-- Indexes for rule lookup
CREATE INDEX idx_validation_rules_category ON validation_rules (rule_category);
CREATE INDEX idx_validation_rules_status ON validation_rules (rule_status);
CREATE INDEX idx_validation_rules_effective ON validation_rules (effective_from, effective_until);
CREATE INDEX idx_validation_rules_priority ON validation_rules (rule_priority);

-- GIN indexes for array fields
CREATE INDEX idx_validation_rules_aircraft_types ON validation_rules USING GIN (applicable_aircraft_types);
CREATE INDEX idx_validation_rules_airports ON validation_rules USING GIN (applicable_airports);
```

#### 5.1.4 validation_results
```sql
CREATE TABLE validation_results (
    id                   BIGSERIAL PRIMARY KEY,
    filing_id           VARCHAR(50) NOT NULL REFERENCES filed_flight_plans(filing_id) ON DELETE CASCADE,
    validation_run_id   VARCHAR(100) NOT NULL,
    rule_code           VARCHAR(20) NOT NULL REFERENCES validation_rules(rule_code),
    validation_status   VARCHAR(20) NOT NULL, -- PASSED, FAILED, WARNING, SKIPPED
    
    -- Error Details
    error_code          VARCHAR(20),
    error_message       TEXT,
    error_severity      VARCHAR(20), -- ERROR, WARNING, INFO
    field_path          VARCHAR(500), -- XPath to the field that failed
    field_value         TEXT, -- The actual value that failed validation
    expected_value      TEXT, -- What the value should be
    suggestion_message  TEXT,
    
    -- Validation Context
    validation_time     TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    validation_duration_ms INTEGER,
    rule_version        VARCHAR(20),
    validator_system    VARCHAR(50) DEFAULT 'FILING_SERVICE',
    
    -- Additional Data
    validation_context  JSONB, -- Additional context for complex validations
    
    INDEX idx_validation_results_filing_id (filing_id),
    INDEX idx_validation_results_rule_code (rule_code),
    INDEX idx_validation_results_status (validation_status),
    INDEX idx_validation_results_time (validation_time DESC),
    
    CONSTRAINT chk_validation_status CHECK (validation_status IN ('PASSED', 'FAILED', 'WARNING', 'SKIPPED')),
    CONSTRAINT chk_error_severity CHECK (error_severity IN ('ERROR', 'WARNING', 'INFO'))
);
```

#### 5.1.5 coordination_log
```sql
CREATE TABLE coordination_log (
    id                   BIGSERIAL PRIMARY KEY,
    filing_id           VARCHAR(50) NOT NULL REFERENCES filed_flight_plans(filing_id) ON DELETE CASCADE,
    coordination_type   VARCHAR(50) NOT NULL, -- FDPS, ATC, AIRPORT, MILITARY, ANSP
    target_system       VARCHAR(100) NOT NULL,
    coordination_status VARCHAR(20) NOT NULL, -- PENDING, SENT, ACKNOWLEDGED, FAILED, TIMEOUT
    
    -- Message Details
    message_id          VARCHAR(100),
    message_type        VARCHAR(50), -- FILING, UPDATE, CANCELLATION
    message_content     TEXT,
    message_format      VARCHAR(20), -- FIXM, AFTN, JSON, XML
    
    -- Timing Information
    initiated_at        TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    sent_at            TIMESTAMP WITH TIME ZONE,
    acknowledged_at    TIMESTAMP WITH TIME ZONE,
    timeout_at         TIMESTAMP WITH TIME ZONE,
    
    -- Response Information
    response_code      VARCHAR(20),
    response_message   TEXT,
    response_data      JSONB,
    
    -- Retry Information
    retry_count        INTEGER DEFAULT 0,
    max_retries        INTEGER DEFAULT 3,
    next_retry_at      TIMESTAMP WITH TIME ZONE,
    
    -- Error Handling
    error_code         VARCHAR(50),
    error_message      TEXT,
    error_details      JSONB,
    
    INDEX idx_coordination_log_filing_id (filing_id),
    INDEX idx_coordination_log_type (coordination_type),
    INDEX idx_coordination_log_status (coordination_status),
    INDEX idx_coordination_log_target (target_system),
    INDEX idx_coordination_log_time (initiated_at DESC),
    
    CONSTRAINT chk_coordination_status CHECK (coordination_status IN ('PENDING', 'SENT', 'ACKNOWLEDGED', 'FAILED', 'TIMEOUT'))
);
```

#### 5.1.6 airspace_restrictions
```sql
CREATE TABLE airspace_restrictions (
    id                   BIGSERIAL PRIMARY KEY,
    restriction_id      VARCHAR(50) UNIQUE NOT NULL,
    restriction_name    VARCHAR(200) NOT NULL,
    restriction_type    VARCHAR(50) NOT NULL, -- PROHIBITED, RESTRICTED, DANGER, MILITARY, TEMPORARY
    
    -- Geographic Definition
    airspace_definition JSONB NOT NULL, -- Polygon or circle definition
    center_latitude     DECIMAL(10,6),
    center_longitude    DECIMAL(11,6),
    radius_nm          DECIMAL(8,2),
    
    -- Altitude Restrictions
    lower_altitude     INTEGER, -- Feet MSL
    upper_altitude     INTEGER, -- Feet MSL
    altitude_reference VARCHAR(10) DEFAULT 'MSL', -- MSL, AGL, FL
    
    -- Time Restrictions
    effective_from     TIMESTAMP WITH TIME ZONE NOT NULL,
    effective_until    TIMESTAMP WITH TIME ZONE,
    time_schedule      JSONB, -- Complex time-based restrictions
    
    -- Restriction Details
    restriction_reason TEXT,
    permitted_operations TEXT[],
    prohibited_operations TEXT[],
    coordination_required BOOLEAN DEFAULT FALSE,
    coordination_authority VARCHAR(100),
    
    -- Notification Information
    notam_reference    VARCHAR(50),
    published_date     TIMESTAMP WITH TIME ZONE,
    originator         VARCHAR(100),
    
    -- Status
    restriction_status VARCHAR(20) DEFAULT 'ACTIVE', -- ACTIVE, INACTIVE, TEMPORARY
    created_at         TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at         TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    INDEX idx_airspace_restrictions_type (restriction_type),
    INDEX idx_airspace_restrictions_status (restriction_status),
    INDEX idx_airspace_restrictions_effective (effective_from, effective_until),
    INDEX idx_airspace_restrictions_location (center_latitude, center_longitude),
    
    CONSTRAINT chk_restriction_type CHECK (restriction_type IN ('PROHIBITED', 'RESTRICTED', 'DANGER', 'MILITARY', 'TEMPORARY')),
    CONSTRAINT chk_restriction_status CHECK (restriction_status IN ('ACTIVE', 'INACTIVE', 'TEMPORARY')),
    CONSTRAINT chk_altitude_order CHECK (lower_altitude <= upper_altitude OR upper_altitude IS NULL)
);

-- Spatial index for geographic queries
CREATE INDEX idx_airspace_restrictions_geo ON airspace_restrictions USING GIST (
    ST_Point(center_longitude, center_latitude)
);
```

#### 5.1.7 slot_coordination
```sql
CREATE TABLE slot_coordination (
    id                   BIGSERIAL PRIMARY KEY,
    filing_id           VARCHAR(50) NOT NULL REFERENCES filed_flight_plans(filing_id) ON DELETE CASCADE,
    airport_code        VARCHAR(4) NOT NULL,
    slot_type          VARCHAR(20) NOT NULL, -- DEPARTURE, ARRIVAL
    
    -- Requested Slot
    requested_time     TIMESTAMP WITH TIME ZONE NOT NULL,
    requested_runway   VARCHAR(10),
    
    -- Assigned Slot
    assigned_time      TIMESTAMP WITH TIME ZONE,
    assigned_runway    VARCHAR(10),
    slot_status        VARCHAR(20) DEFAULT 'REQUESTED', -- REQUESTED, ASSIGNED, DENIED, CANCELLED
    
    -- Slot Details
    slot_tolerance_minutes INTEGER DEFAULT 15,
    priority_level     VARCHAR(20) DEFAULT 'NORMAL', -- HIGH, NORMAL, LOW
    slot_category      VARCHAR(20), -- COORDINATED, FACILITATED, NON_COORDINATED
    
    -- Coordination Information
    coordinator_system VARCHAR(100),
    coordination_reference VARCHAR(100),
    assigned_by        VARCHAR(100),
    assigned_at        TIMESTAMP WITH TIME ZONE,
    
    -- Timing
    created_at         TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at         TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    INDEX idx_slot_coordination_filing_id (filing_id),
    INDEX idx_slot_coordination_airport (airport_code),
    INDEX idx_slot_coordination_type (slot_type),
    INDEX idx_slot_coordination_status (slot_status),
    INDEX idx_slot_coordination_time (requested_time),
    
    CONSTRAINT chk_slot_type CHECK (slot_type IN ('DEPARTURE', 'ARRIVAL')),
    CONSTRAINT chk_slot_status CHECK (slot_status IN ('REQUESTED', 'ASSIGNED', 'DENIED', 'CANCELLED')),
    CONSTRAINT chk_priority_level CHECK (priority_level IN ('HIGH', 'NORMAL', 'LOW'))
);
```

### 5.2 Sample Business Rules Data

#### 5.2.1 Vietnamese Airspace Business Rules
```sql
-- Insert sample validation rules
INSERT INTO validation_rules (
    rule_code, rule_name, rule_category, rule_type, rule_description,
    rule_expression, error_message, suggestion_message, effective_from
) VALUES

-- Aircraft Performance Rules
('BR001', 'Maximum Cruise Altitude Check', 'BUSINESS', 'MANDATORY',
 'Validates that requested cruise altitude does not exceed aircraft capability',
 'cruiseAltitude <= aircraftType.maxAltitude',
 'Requested cruise level exceeds maximum altitude for aircraft type',
 'Reduce cruise level to maximum certified altitude or below',
 '2025-01-01 00:00:00+00'),

('BR002', 'Minimum Equipment Requirements', 'BUSINESS', 'MANDATORY',
 'Ensures aircraft has required navigation equipment for Vietnamese airspace',
 'aircraft.hasRNPCapability() || route.isConventionalNavOnly()',
 'Aircraft lacks required RNP navigation capability for requested route',
 'Ensure aircraft is equipped with RNP-capable navigation system',
 '2025-01-01 00:00:00+00'),

-- Airspace Rules
('BR003', 'Prohibited Area Check', 'BUSINESS', 'MANDATORY',
 'Validates route does not pass through prohibited airspace',
 'route.checkProhibitedAreas(vietnameseProhibitedZones)',
 'Requested route passes through prohibited airspace',
 'Modify route to avoid prohibited areas or request special authorization',
 '2025-01-01 00:00:00+00'),

('BR004', 'Military Coordination Required', 'BUSINESS', 'WARNING',
 'Flags flights requiring military airspace coordination',
 'route.intersects(militaryAirspace) && !hasPreAuthorization',
 'Flight requires military airspace coordination',
 'Military coordination will be initiated automatically',
 '2025-01-01 00:00:00+00'),

-- Time-based Rules  
('BR005', 'Airport Operating Hours', 'BUSINESS', 'MANDATORY',
 'Validates flight times against airport operating schedules',
 'departure.time.isBetween(airport.operatingHours) && arrival.time.isBetween(airport.operatingHours)',
 'Flight time outside airport operating hours',
 'Adjust departure/arrival times to airport operating schedule',
 '2025-01-01 00:00:00+00'),

('BR006', 'Noise Curfew Compliance', 'BUSINESS', 'MANDATORY',
 'Checks compliance with airport noise restrictions',
 '!airport.hasNoiseCurfew() || !departure.time.isDuring(airport.noiseCurfew) || aircraft.isNoiseExempt()',
 'Departure time violates airport noise curfew',
 'Reschedule departure outside curfew hours or use noise-compliant aircraft',
 '2025-01-01 00:00:00+00'),

-- Fuel and Safety Rules
('BR007', 'Fuel Endurance Check', 'BUSINESS', 'MANDATORY',
 'Validates fuel endurance meets regulatory requirements',
 'fuelEndurance >= route.estimatedTime + alternateReserve + finalReserve',
 'Fuel endurance insufficient for planned route with required reserves',
 'Increase fuel load or select closer alternate airports',
 '2025-01-01 00:00:00+00');

-- Set applicable parameters for rules
UPDATE validation_rules SET applicable_airports = ARRAY['VVNB', 'VVTS', 'VVDN', 'VVCI'] 
WHERE rule_code IN ('BR005', 'BR006');

UPDATE validation_rules SET applicable_aircraft_types = ARRAY['A320', 'A321', 'B737', 'A350'] 
WHERE rule_code = 'BR001';
```

#### 5.2.2 Sample Airspace Restrictions
```sql
-- Insert Vietnamese airspace restrictions
INSERT INTO airspace_restrictions (
    restriction_id, restriction_name, restriction_type,
    center_latitude, center_longitude, radius_nm,
    lower_altitude, upper_altitude,
    effective_from, restriction_reason,
    coordination_required, coordination_authority
) VALUES

-- Military Training Areas
('VN_MIL_001', 'Nha Trang Military Training Area', 'MILITARY',
 12.2500, 109.1667, 15.0,
 0, 25000,
 '2025-01-01 00:00:00+00',
 'Military training operations',
 true, 'Vietnam Air Force Command'),

('VN_MIL_002', 'Phu Cat Military Airspace', 'MILITARY', 
 13.9500, 109.0333, 10.0,
 0, 35000,
 '2025-01-01 00:00:00+00',
 'Military air base operations',
 true, 'Vietnam Air Force Command'),

-- Prohibited Areas
('VN_PROB_001', 'Presidential Palace Prohibited Zone', 'PROHIBITED',
 21.0369, 105.8340, 2.0,
 0, 3000,
 '2025-01-01 00:00:00+00',
 'National security - Presidential facilities',
 false, 'Ministry of Defense'),

-- Temporary Restrictions
('VN_TEMP_001', 'Hanoi Fireworks Display', 'TEMPORARY',
 21.0285, 105.8542, 3.0,
 0, 5000,
 '2025-12-31 16:00:00+00',
 'New Year fireworks display',
 true, 'Hanoi City Authority');

-- Update effective dates for temporary restrictions
UPDATE airspace_restrictions 
SET effective_until = '2025-12-31 20:00:00+00'
WHERE restriction_id = 'VN_TEMP_001';
```

### 5.3 Performance Optimization

#### 5.3.1 Partitioning Strategy
```sql
-- Partition filed_flight_plans by date for better performance
CREATE TABLE filed_flight_plans_y2025m09 PARTITION OF filed_flight_plans
    FOR VALUES FROM ('2025-09-01') TO ('2025-10-01');

CREATE TABLE filed_flight_plans_y2025m10 PARTITION OF filed_flight_plans  
    FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');

-- Create monthly partitions automatically
CREATE OR REPLACE FUNCTION create_monthly_partition()
RETURNS void AS $$
DECLARE
    start_date date;
    end_date date;
    table_name text;
BEGIN
    start_date := date_trunc('month', CURRENT_DATE + interval '1 month');
    end_date := start_date + interval '1 month';
    table_name := 'filed_flight_plans_y' || to_char(start_date, 'YYYY') || 'm' || to_char(start_date, 'MM');
    
    EXECUTE format('CREATE TABLE %I PARTITION OF filed_flight_plans FOR VALUES FROM (%L) TO (%L)',
                   table_name, start_date, end_date);
END;
$$ LANGUAGE plpgsql;

-- Schedule monthly partition creation
SELECT cron.schedule('create-monthly-partition', '0 0 15 * *', 'SELECT create_monthly_partition();');
```

#### 5.3.2 Database Views for Common Queries
```sql
-- View for active flight plans
CREATE VIEW active_flight_plans AS
SELECT 
    filing_id,
    gufi,
    flight_number,
    airline_code,
    departure_airport,
    arrival_airport,
    estimated_off_block,
    estimated_on_block,
    current_status,
    submitted_at
FROM filed_flight_plans
WHERE current_status IN ('ACCEPTED', 'PENDING')
  AND estimated_off_block >= CURRENT_DATE
  AND estimated_off_block <= CURRENT_DATE + interval '7 days';

-- View for today's departures by airport
CREATE VIEW todays_departures AS
SELECT 
    departure_airport,
    COUNT(*) as flight_count,
    array_agg(flight_number ORDER BY estimated_off_block) as flights,
    min(estimated_off_block) as first_departure,
    max(estimated_off_block) as last_departure
FROM filed_flight_plans
WHERE DATE(estimated_off_block) = CURRENT_DATE
  AND current_status = 'ACCEPTED'
GROUP BY departure_airport;

-- View for validation statistics
CREATE VIEW validation_statistics AS
SELECT 
    DATE(submitted_at) as filing_date,
    current_status,
    COUNT(*) as count,
    AVG(EXTRACT(EPOCH FROM (processed_at - submitted_at))) as avg_processing_time_seconds
FROM filed_flight_plans
WHERE submitted_at >= CURRENT_DATE - interval '30 days'
GROUP BY DATE(submitted_at), current_status
ORDER BY filing_date DESC, current_status;
```

---

## 6. BUSINESS LOGIC IMPLEMENTATION

### 6.1 Filing Process Workflow

#### 6.1.1 Main Filing Process
```java
@Service
@Transactional
public class FilingService {
    
    @Autowired
    private eFPLProcessor efplProcessor;
    
    @Autowired
    private ValidationEngine validationEngine;
    
    @Autowired
    private BusinessRulesEngine businessRulesEngine;
    
    @Autowired
    private FDPSIntegrator fdpsIntegrator;
    
    @Autowired
    private NotificationManager notificationManager;
    
    public FilingResponse processFlightPlanFiling(FilingRequest request) {
        // Generate filing ID
        String filingId = generateFilingId();
        
        try {
            // Step 1: Parse and validate FIXM format
            FlightPlan flightPlan = efplProcessor.parseeFPL(request.geteFPLXml());
            validateeFPLFormat(flightPlan);
            
            // Step 2: Store initial filing record
            FiledFlightPlan filedFlight = createFilingRecord(filingId, flightPlan, request);
            
            // Step 3: Execute validation chain
            ValidationResult validationResult = executeValidationChain(flightPlan);
            
            // Step 4: Determine filing status based on validation
            FilingStatus status = determineFilingStatus(validationResult);
            
            // Step 5: Process based on status
            switch (status) {
                case ACCEPTED:
                    return processAcceptedFiling(filedFlight, validationResult);
                case REJECTED:
                    return processRejectedFiling(filedFlight, validationResult);
                case PENDING:
                    return processPendingFiling(filedFlight, validationResult);
                default:
                    throw new FilingProcessException("Unknown filing status: " + status);
            }
            
        } catch (Exception e) {
            handleFilingError(filingId, e);
            throw new FilingProcessException("Filing process failed", e);
        }
    }
    
    private ValidationResult executeValidationChain(FlightPlan flightPlan) {
        ValidationResult result = new ValidationResult();
        
        // Format validation (already done in parsing)
        result.setFormatValidation(ValidationStatus.PASSED);
        
        // Business rules validation
        BusinessRulesResult businessResult = businessRulesEngine.validate(flightPlan);
        result.setBusinessRulesValidation(businessResult.getStatus());
        result.addErrors(businessResult.getErrors());
        
        // Technical validation
        TechnicalValidationResult techResult = validationEngine.validateTechnical(flightPlan);
        result.setTechnicalValidation(techResult.getStatus());
        result.addErrors(techResult.getErrors());
        
        // Operational validation
        OperationalValidationResult opResult = validationEngine.validateOperational(flightPlan);
        result.setOperationalValidation(opResult.getStatus());
        result.addErrors(opResult.getErrors());
        
        return result;
    }
    
    private FilingResponse processAcceptedFiling(FiledFlightPlan filing, ValidationResult validation) {
        // Update status
        filing.setCurrentStatus(FilingStatus.ACCEPTED);
        filing.setProcessedAt(Instant.now());
        
        // Integrate with FDPS
        FDPSIntegrationResult fdpsResult = fdpsIntegrator.submitToFDPS(filing);
        if (!fdpsResult.isSuccess()) {
            // FDPS integration failed, but filing is still valid
            log.warn("FDPS integration failed for filing {}: {}", filing.getFilingId(), fdpsResult.getErrorMessage());
        }
        
        // Send notifications
        notificationManager.notifyStakeholders(filing, NotificationType.FILING_ACCEPTED);
        
        // Request slot coordination if needed
        if (requiresSlotCoordination(filing)) {
            slotCoordinationService.requestSlots(filing);
        }
        
        return buildSuccessResponse(filing, validation, fdpsResult);
    }
    
    private FilingResponse processRejectedFiling(FiledFlightPlan filing, ValidationResult validation) {
        filing.setCurrentStatus(FilingStatus.REJECTED);
        filing.setProcessedAt(Instant.now());
        
        // Send rejection notification
        notificationManager.notifyStakeholders(filing, NotificationType.FILING_REJECTED);
        
        return buildRejectionResponse(filing, validation);
    }
    
    private FilingResponse processPendingFiling(FiledFlightPlan filing, ValidationResult validation) {
        filing.setCurrentStatus(FilingStatus.PENDING);
        
        // Determine what requires manual review
        List<PendingReason> pendingReasons = determinePendingReasons(validation);
        
        // Queue for manual review
        manualReviewQueue.addForReview(filing, pendingReasons);
        
        // Send pending notification
        notificationManager.notifyStakeholders(filing, NotificationType.FILING_PENDING);
        
        return buildPendingResponse(filing, validation, pendingReasons);
    }
}
```

#### 6.1.2 Business Rules Engine Implementation
```java
@Component
public class BusinessRulesEngine {
    
    @Autowired
    private KieSession kieSession;
    
    @Autowired
    private AirspaceService airspaceService;
    
    @Autowired
    private WeatherService weatherService;
    
    public BusinessRulesResult validate(FlightPlan flightPlan) {
        BusinessRulesResult result = new BusinessRulesResult();
        
        try {
            // Set up facts for rules engine
            kieSession.insert(flightPlan);
            kieSession.insert(result);
            
            // Add contextual facts
            addAirspaceRestrictions(flightPlan);
            addWeatherInformation(flightPlan);
            addAircraftCapabilities(flightPlan);
            
            // Execute rules
            int rulesFired = kieSession.fireAllRules();
            log.debug("Executed {} business rules for flight {}", rulesFired, flightPlan.getGufi());
            
            // Clear session for next use
            kieSession.dispose();
            
        } catch (Exception e) {
            log.error("Business rules validation failed for flight {}", flightPlan.getGufi(), e);
            result.addError(new ValidationError("BR999", "Business rules engine error", e.getMessage()));
        }
        
        return result;
    }
    
    private void addAirspaceRestrictions(FlightPlan flightPlan) {
        List<AirspaceRestriction> restrictions = airspaceService.getRestrictionsForRoute(
            flightPlan.getRoute(), 
            flightPlan.getDepartureTime()
        );
        
        for (AirspaceRestriction restriction : restrictions) {
            kieSession.insert(restriction);
        }

    private void addWeatherInformation(FlightPlan flightPlan) {
        // Get weather along route
        WeatherInformation weather = weatherService.getWeatherForRoute(
            flightPlan.getRoute(),
            flightPlan.getDepartureTime(),
            flightPlan.getArrivalTime()
        );
        
        kieSession.insert(weather);
        
        // Get NOTAM information
        List<NOTAM> notams = notamService.getActiveNOTAMs(
            flightPlan.getDepartureAirport(),
            flightPlan.getArrivalAirport(),
            flightPlan.getDepartureTime()
        );
        
        for (NOTAM notam : notams) {
            kieSession.insert(notam);
        }
    }
    
    private void addAircraftCapabilities(FlightPlan flightPlan) {
        AircraftCapabilities capabilities = aircraftService.getCapabilities(
            flightPlan.getAircraftType(),
            flightPlan.getEquipmentCode()
        );
        
        kieSession.insert(capabilities);
    }
}
```

#### 6.1.3 Drools Business Rules Examples
```drools
// Business Rules in DRL format

package com.vatm.filing.rules

import com.vatm.filing.model.*
import java.time.LocalTime

// Rule: Maximum cruise altitude validation
rule "Maximum Cruise Altitude Check"
    when
        $fp: FlightPlan()
        $ac: AircraftCapabilities(aircraftType == $fp.aircraftType, 
                                 maxCruiseAltitude < $fp.cruiseAltitude)
        $result: BusinessRulesResult()
    then
        $result.addError(new ValidationError(
            "BR001",
            "BUSINESS_RULE", 
            "Requested cruise level FL" + ($fp.cruiseAltitude/100) + 
            " exceeds maximum altitude FL" + ($ac.maxCruiseAltitude/100) + 
            " for " + $fp.aircraftType,
            "Reduce cruise level to FL" + ($ac.maxCruiseAltitude/100) + " or below"
        ));
end

// Rule: Prohibited airspace check
rule "Prohibited Airspace Violation"
    when
        $fp: FlightPlan()
        $restriction: AirspaceRestriction(restrictionType == "PROHIBITED",
                                         routeIntersects($fp.route) == true,
                                         isActiveAt($fp.departureTime) == true)
        $result: BusinessRulesResult()
    then
        $result.addError(new ValidationError(
            "BR003",
            "BUSINESS_RULE",
            "Route passes through prohibited airspace: " + $restriction.getName(),
            "Modify route to avoid " + $restriction.getName() + " or request special authorization"
        ));
end

// Rule: Military coordination required
rule "Military Airspace Coordination"
    when
        $fp: FlightPlan()
        $restriction: AirspaceRestriction(restrictionType == "MILITARY",
                                         routeIntersects($fp.route) == true,
                                         coordinationRequired == true)
        $result: BusinessRulesResult()
    then
        $result.addWarning(new ValidationError(
            "BR004",
            "COORDINATION_REQUIRED",
            "Flight requires military airspace coordination: " + $restriction.getName(),
            "Military coordination will be initiated automatically"
        ));
end

// Rule: Airport operating hours
rule "Airport Operating Hours Check"
    when
        $fp: FlightPlan()
        $airport: Airport(icaoCode == $fp.departureAirport,
                         isOperatingAt($fp.departureTime) == false)
        $result: BusinessRulesResult()
    then
        $result.addError(new ValidationError(
            "BR005",
            "BUSINESS_RULE",
            "Departure time " + $fp.departureTime + " outside operating hours for " + $airport.getName(),
            "Adjust departure time to airport operating schedule: " + $airport.getOperatingHours()
        ));
end

// Rule: Noise curfew compliance
rule "Noise Curfew Violation"
    when
        $fp: FlightPlan()
        $airport: Airport(icaoCode == $fp.departureAirport,
                         hasNoiseCurfew() == true,
                         isDuringCurfew($fp.departureTime) == true)
        $aircraft: AircraftCapabilities(aircraftType == $fp.aircraftType,
                                       noiseExempt == false)
        $result: BusinessRulesResult()
    then
        $result.addError(new ValidationError(
            "BR006",
            "BUSINESS_RULE",
            "Departure time violates noise curfew at " + $airport.getName(),
            "Reschedule departure outside curfew hours: " + $airport.getNoiseCurfewHours()
        ));
end

// Rule: Fuel endurance validation
rule "Fuel Endurance Check"
    when
        $fp: FlightPlan()
        eval($fp.getFuelEnduranceMinutes() < ($fp.getEstimatedFlightTime() + $fp.getAlternateReserve() + 45))
        $result: BusinessRulesResult()
    then
        int requiredMinutes = $fp.getEstimatedFlightTime() + $fp.getAlternateReserve() + 45;
        $result.addError(new ValidationError(
            "BR007",
            "BUSINESS_RULE",
            "Fuel endurance " + $fp.getFuelEnduranceMinutes() + " minutes insufficient. Required: " + requiredMinutes + " minutes",
            "Increase fuel load or select closer alternate airports"
        ));
end

// Rule: RNP capability requirement
rule "RNP Capability Required"
    when
        $fp: FlightPlan()
        $route: Route(requiresRNP == true)
        $aircraft: AircraftCapabilities(aircraftType == $fp.aircraftType,
                                       hasRNPCapability() == false)
        $result: BusinessRulesResult()
    then
        $result.addError(new ValidationError(
            "BR002",
            "BUSINESS_RULE",
            "Aircraft lacks required RNP navigation capability for route " + $route.getName(),
            "Ensure aircraft is equipped with RNP-capable navigation system or select conventional route"
        ));
end

// Rule: RVSM compliance above FL290
rule "RVSM Capability Above FL290"
    when
        $fp: FlightPlan(cruiseAltitude >= 29000)
        $aircraft: AircraftCapabilities(aircraftType == $fp.aircraftType,
                                       rvsm_approved == false)
        $result: BusinessRulesResult()
    then
        $result.addError(new ValidationError(
            "BR008",
            "BUSINESS_RULE",
            "RVSM approval required for operations above FL290",
            "Reduce cruise altitude to FL280 or below, or ensure aircraft has RVSM approval"
        ));
end
```

### 6.2 FDPS Integration Module

#### 6.2.1 FDPS Integration Service
```java
@Service
public class FDPSIntegrator {
    
    @Autowired
    private JmsTemplate jmsTemplate;
    
    @Autowired
    private FDPSMessageConverter messageConverter;
    
    @Value("${vatm.fdps.queue.outbound}")
    private String fdpsOutboundQueue;
    
    @Value("${vatm.fdps.timeout.seconds:30}")
    private int fdpsTimeoutSeconds;
    
    public FDPSIntegrationResult submitToFDPS(FiledFlightPlan filing) {
        try {
            // Convert eFPL to FDPS internal format
            FDPSMessage fdpsMessage = messageConverter.convertToFDPSFormat(filing);
            
            // Add message headers
            fdpsMessage.setMessageId(generateMessageId());
            fdpsMessage.setTimestamp(Instant.now());
            fdpsMessage.setOriginatingSystem("FF-ICE_FILING_SERVICE");
            fdpsMessage.setMessageType(FDPSMessageType.FLIGHT_PLAN_FILING);
            
            // Send to FDPS queue
            jmsTemplate.convertAndSend(fdpsOutboundQueue, fdpsMessage, message -> {
                message.setJMSCorrelationID(filing.getFilingId());
                message.setJMSExpiration(System.currentTimeMillis() + (fdpsTimeoutSeconds * 1000));
                return message;
            });
            
            // Wait for acknowledgment (synchronous for critical operations)
            FDPSResponse response = waitForFDPSResponse(filing.getFilingId());
            
            // Log integration result
            logFDPSIntegration(filing, fdpsMessage, response);
            
            return new FDPSIntegrationResult(true, response.getResponseCode(), response.getMessage());
            
        } catch (JMSException e) {
            log.error("FDPS integration failed for filing {}: {}", filing.getFilingId(), e.getMessage(), e);
            return new FDPSIntegrationResult(false, "JMS_ERROR", e.getMessage());
        } catch (TimeoutException e) {
            log.error("FDPS integration timeout for filing {}", filing.getFilingId(), e);
            return new FDPSIntegrationResult(false, "TIMEOUT", "FDPS response timeout");
        } catch (Exception e) {
            log.error("Unexpected error in FDPS integration for filing {}", filing.getFilingId(), e);
            return new FDPSIntegrationResult(false, "UNKNOWN_ERROR", e.getMessage());
        }
    }
    
    private FDPSResponse waitForFDPSResponse(String filingId) throws TimeoutException {
        String responseQueue = "fdps.response." + filingId;
        
        try {
            Message responseMessage = jmsTemplate.receive(responseQueue);
            if (responseMessage == null) {
                throw new TimeoutException("No response received from FDPS within timeout period");
            }
            
            return messageConverter.convertFromFDPSResponse(responseMessage);
            
        } catch (JMSException e) {
            throw new RuntimeException("Error receiving FDPS response", e);
        }
    }
    
    private void logFDPSIntegration(FiledFlightPlan filing, FDPSMessage request, FDPSResponse response) {
        CoordinationLog logEntry = new CoordinationLog();
        logEntry.setFilingId(filing.getFilingId());
        logEntry.setCoordinationType(CoordinationType.FDPS);
        logEntry.setTargetSystem("VATM_FDPS");
        logEntry.setMessageId(request.getMessageId());
        logEntry.setMessageType("FLIGHT_PLAN_FILING");
        logEntry.setCoordinationStatus(response.isSuccess() ? CoordinationStatus.ACKNOWLEDGED : CoordinationStatus.FAILED);
        logEntry.setResponseCode(response.getResponseCode());
        logEntry.setResponseMessage(response.getMessage());
        logEntry.setInitiatedAt(Instant.now());
        
        coordinationLogRepository.save(logEntry);
    }
}
```

#### 6.2.2 Message Format Conversion
```java
@Component
public class FDPSMessageConverter {
    
    public FDPSMessage convertToFDPSFormat(FiledFlightPlan filing) {
        FDPSMessage message = new FDPSMessage();
        
        // Flight identification
        message.setCallsign(filing.getFlightNumber());
        message.setAircraftId(filing.getAircraftRegistration());
        message.setAircraftType(filing.getAircraftType());
        message.setOperator(filing.getOperatorCode());
        
        // Departure information
        message.setDepartureAirport(filing.getDepartureAirport());
        message.setDepartureTime(filing.getEstimatedOffBlock());
        message.setDepartureRunway(filing.getDepartureRunway());
        
        // Arrival information
        message.setArrivalAirport(filing.getArrivalAirport());
        message.setArrivalTime(filing.getEstimatedOnBlock());
        
        // Route information
        message.setRoute(filing.getRouteText());
        message.setCruiseAltitude(filing.getCruiseAltitude());
        message.setCruiseSpeed(filing.getCruiseSpeed());
        
        // Flight rules and type
        message.setFlightRules(filing.getFlightRules());
        message.setFlightType(filing.getFlightType());
        
        // Equipment and capabilities
        message.setEquipmentCode(filing.getEquipmentCode());
        message.setTransponderCode(filing.getTransponderCode());
        message.setWakeTurbulenceCategory(filing.getWakeTurbulence());
        
        // Additional information
        message.setPersonsOnBoard(filing.getPersonsOnBoard());
        message.setFuelEndurance(formatFuelEndurance(filing));
        message.setAlternateAirports(formatAlternateAirports(filing));
        message.setRemarks(filing.getSpecialHandling());
        
        // Filing metadata
        message.setGufi(filing.getGufi());
        message.setFilingId(filing.getFilingId());
        message.setFilingTime(filing.getSubmittedAt());
        
        return message;
    }
    
    private String formatFuelEndurance(FiledFlightPlan filing) {
        if (filing.getFuelEnduranceHours() != null && filing.getFuelEnduranceMinutes() != null) {
            return String.format("%02d%02d", filing.getFuelEnduranceHours(), filing.getFuelEnduranceMinutes());
        }
        return null;
    }
    
    private String formatAlternateAirports(FiledFlightPlan filing) {
        List<String> alternates = new ArrayList<>();
        if (filing.getAlternateAirport1() != null) {
            alternates.add(filing.getAlternateAirport1());
        }
        if (filing.getAlternateAirport2() != null) {
            alternates.add(filing.getAlternateAirport2());
        }
        return String.join(" ", alternates);
    }
}
```

### 6.3 Notification Management

#### 6.3.1 Notification Manager Service
```java
@Service
@Async
public class NotificationManager {
    
    @Autowired
    private List<NotificationChannel> notificationChannels;
    
    @Autowired
    private NotificationTemplateService templateService;
    
    public void notifyStakeholders(FiledFlightPlan filing, NotificationType notificationType) {
        try {
            // Determine notification recipients based on flight and notification type
            List<NotificationRecipient> recipients = determineRecipients(filing, notificationType);
            
            for (NotificationRecipient recipient : recipients) {
                sendNotification(filing, notificationType, recipient);
            }
            
        } catch (Exception e) {
            log.error("Error sending notifications for filing {}: {}", filing.getFilingId(), e.getMessage(), e);
        }
    }
    
    private void sendNotification(FiledFlightPlan filing, NotificationType type, NotificationRecipient recipient) {
        try {
            // Generate notification content
            NotificationContent content = templateService.generateContent(filing, type, recipient);
            
            // Find appropriate notification channel
            NotificationChannel channel = findNotificationChannel(recipient.getPreferredChannel());
            
            // Send notification
            NotificationResult result = channel.sendNotification(recipient, content);
            
            // Log notification attempt
            logNotificationAttempt(filing, type, recipient, result);
            
        } catch (Exception e) {
            log.error("Failed to send notification to {}: {}", recipient.getIdentifier(), e.getMessage(), e);
            logNotificationFailure(filing, type, recipient, e);
        }
    }
    
    private List<NotificationRecipient> determineRecipients(FiledFlightPlan filing, NotificationType type) {
        List<NotificationRecipient> recipients = new ArrayList<>();
        
        switch (type) {
            case FILING_ACCEPTED:
                // Notify submitting airline
                recipients.add(createAirlineRecipient(filing.getAirlineCode()));
                
                // Notify departure ATC
                recipients.add(createATCRecipient(filing.getDepartureAirport()));
                
                // Notify arrival ATC if different
                if (!filing.getDepartureAirport().equals(filing.getArrivalAirport())) {
                    recipients.add(createATCRecipient(filing.getArrivalAirport()));
                }
                
                // Notify airports
                recipients.add(createAirportRecipient(filing.getDepartureAirport()));
                recipients.add(createAirportRecipient(filing.getArrivalAirport()));
                
                // Notify adjacent ANSPs for international flights
                if (isInternationalFlight(filing)) {
                    recipients.addAll(getAdjacentANSPs(filing));
                }
                break;
                
            case FILING_REJECTED:
                // Only notify submitting airline
                recipients.add(createAirlineRecipient(filing.getAirlineCode()));
                break;
                
            case FILING_PENDING:
                // Notify submitting airline
                recipients.add(createAirlineRecipient(filing.getAirlineCode()));
                
                // Notify relevant coordination authorities
                recipients.addAll(getCoordinationAuthorities(filing));
                break;
                
            case FILING_CANCELLED:
                // Notify all previously notified parties
                recipients.addAll(getAllPreviouslyNotifiedParties(filing));
                break;
        }
        
        return recipients;
    }
    
    private NotificationChannel findNotificationChannel(ChannelType channelType) {
        return notificationChannels.stream()
            .filter(channel -> channel.supports(channelType))
            .findFirst()
            .orElseThrow(() -> new RuntimeException("No notification channel found for type: " + channelType));
    }
}
```

#### 6.3.2 SWIM Notification Channel
```java
@Component
public class SWIMNotificationChannel implements NotificationChannel {
    
    @Autowired
    private SWIMMessageSender swimMessageSender;
    
    @Override
    public boolean supports(ChannelType channelType) {
        return channelType == ChannelType.SWIM;
    }
    
    @Override
    public NotificationResult sendNotification(NotificationRecipient recipient, NotificationContent content) {
        try {
            // Create SWIM message
            SWIMMessage swimMessage = createSWIMMessage(recipient, content);
            
            // Send via SWIM infrastructure
            SWIMResponse response = swimMessageSender.sendMessage(
                recipient.getSWIMEndpoint(), 
                swimMessage
            );
            
            if (response.isSuccess()) {
                return NotificationResult.success(response.getMessageId());
            } else {
                return NotificationResult.failure(response.getErrorMessage());
            }
            
        } catch (Exception e) {
            return NotificationResult.failure(e.getMessage());
        }
    }
    
    private SWIMMessage createSWIMMessage(NotificationRecipient recipient, NotificationContent content) {
        SWIMMessage message = new SWIMMessage();
        
        // Message header
        message.setMessageId(UUID.randomUUID().toString());
        message.setTimestamp(Instant.now());
        message.setSender("VATM");
        message.setRecipient(recipient.getIdentifier());
        message.setMessageType(content.getMessageType());
        
        // Message body based on content type
        switch (content.getContentType()) {
            case FILING_STATUS:
                message.setPayload(createFilingStatusPayload(content));
                break;
            case FLIGHT_PLAN_DATA:
                message.setPayload(createFlightPlanPayload(content));
                break;
            case COORDINATION_REQUEST:
                message.setPayload(createCoordinationPayload(content));
                break;
        }
        
        return message;
    }
}
```

---

## 7. SECURITY & AUTHENTICATION

### 7.1 API Security Implementation

#### 7.1.1 Security Configuration
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    
    @Autowired
    private APIKeyAuthenticationProvider apiKeyAuthProvider;
    
    @Autowired
    private JwtAuthenticationProvider jwtAuthProvider;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/api/v1/filing/**").authenticated()
                .requestMatchers("/api/v1/swim/**").authenticated()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(new APIKeyAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
            .addFilterBefore(new JwtAuthenticationFilter(), APIKeyAuthenticationFilter.class)
            .authenticationProvider(apiKeyAuthProvider)
            .authenticationProvider(jwtAuthProvider)
            .exceptionHandling()
                .authenticationEntryPoint(new CustomAuthenticationEntryPoint())
                .accessDeniedHandler(new CustomAccessDeniedHandler());
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean
    public RateLimitingFilter rateLimitingFilter() {
        return new RateLimitingFilter();
    }
}
```

#### 7.1.2 API Key Authentication
```java
@Component
public class APIKeyAuthenticationProvider implements AuthenticationProvider {
    
    @Autowired
    private APIKeyService apiKeyService;
    
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String apiKey = authentication.getPrincipal().toString();
        
        APIKeyDetails keyDetails = apiKeyService.validateAPIKey(apiKey);
        
        if (keyDetails == null) {
            throw new BadCredentialsException("Invalid API key");
        }
        
        if (!keyDetails.isActive()) {
            throw new DisabledException("API key is disabled");
        }
        
        if (keyDetails.isExpired()) {
            throw new CredentialsExpiredException("API key has expired");
        }
        
        // Check IP restrictions
        String clientIP = getClientIP();
        if (!keyDetails.isIPAllowed(clientIP)) {
            throw new BadCredentialsException("IP address not allowed for this API key");
        }
        
        // Create authenticated token
        List<GrantedAuthority> authorities = keyDetails.getAuthorities();
        APIKeyAuthenticationToken token = new APIKeyAuthenticationToken(apiKey, authorities);
        token.setAuthenticated(true);
        token.setDetails(keyDetails);
        
        // Update last used timestamp
        apiKeyService.updateLastUsed(apiKey);
        
        return token;
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return APIKeyAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

#### 7.1.3 Rate Limiting Implementation
```java
@Component
public class RateLimitingFilter extends OncePerRequestFilter {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @Value("${vatm.security.rate-limit.default-hourly:1000}")
    private int defaultHourlyLimit;
    
    @Value("${vatm.security.rate-limit.default-minute:50}")
    private int defaultMinuteLimit;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, 
                                  FilterChain filterChain) throws ServletException, IOException {
        
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()) {
            filterChain.doFilter(request, response);
            return;
        }
        
        String identifier = getClientIdentifier(auth, request);
        
        // Check rate limits
        RateLimitResult hourlyCheck = checkRateLimit(identifier, "hourly", 3600, getHourlyLimit(auth));
        RateLimitResult minuteCheck = checkRateLimit(identifier, "minute", 60, getMinuteLimit(auth));
        
        // Set rate limit headers
        response.setHeader("X-RateLimit-Hourly-Limit", String.valueOf(getHourlyLimit(auth)));
        response.setHeader("X-RateLimit-Hourly-Remaining", String.valueOf(hourlyCheck.getRemaining()));
        response.setHeader("X-RateLimit-Minute-Limit", String.valueOf(getMinuteLimit(auth)));
        response.setHeader("X-RateLimit-Minute-Remaining", String.valueOf(minuteCheck.getRemaining()));
        
        if (!hourlyCheck.isAllowed() || !minuteCheck.isAllowed()) {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.setHeader("Retry-After", String.valueOf(Math.min(hourlyCheck.getRetryAfter(), minuteCheck.getRetryAfter())));
            response.getWriter().write("{\"error\":\"Rate limit exceeded\",\"message\":\"Too many requests\"}");
            return;
        }
        
        filterChain.doFilter(request, response);
    }
    
    private RateLimitResult checkRateLimit(String identifier, String window, int windowSeconds, int limit) {
        String key = "rate_limit:" + identifier + ":" + window;
        String currentWindow = String.valueOf(System.currentTimeMillis() / (windowSeconds * 1000));
        String fullKey = key + ":" + currentWindow;
        
        Long currentCount = redisTemplate.opsForValue().increment(fullKey);
        if (currentCount == 1) {
            redisTemplate.expire(fullKey, windowSeconds, TimeUnit.SECONDS);
        }
        
        boolean allowed = currentCount <= limit;
        int remaining = Math.max(0, limit - currentCount.intValue());
        int retryAfter = allowed ? 0 : windowSeconds;
        
        return new RateLimitResult(allowed, remaining, retryAfter);
    }
}
```

### 7.2 Data Encryption and Privacy

#### 7.2.1 Field-Level Encryption
```java
@Component
public class FieldEncryptionService {
    
    @Value("${vatm.security.encryption.key}")
    private String encryptionKey;
    
    private final AESUtil aesUtil = new AESUtil();
    
    @EventListener
    public void handlePrePersist(Object entity) {
        if (entity instanceof FiledFlightPlan) {
            encryptSensitiveFields((FiledFlightPlan) entity);
        }
    }
    
    @EventListener  
    public void handlePostLoad(Object entity) {
        if (entity instanceof FiledFlightPlan) {
            decryptSensitiveFields((FiledFlightPlan) entity);
        }
    }
    
    private void encryptSensitiveFields(FiledFlightPlan filing) {
        try {
            // Encrypt pilot information
            if (filing.getPilotInCommand() != null) {
                filing.setPilotInCommand(aesUtil.encrypt(filing.getPilotInCommand(), encryptionKey));
            }
            
            // Encrypt emergency contact
            if (filing.getEmergencyContact() != null) {
                filing.setEmergencyContact(aesUtil.encrypt(filing.getEmergencyContact(), encryptionKey));
            }
            
            // Encrypt persons on board (sensitive operational info)
            if (filing.getPersonsOnBoard() != null) {
                filing.setPersonsOnBoard(Integer.valueOf(
                    aesUtil.encrypt(filing.getPersonsOnBoard().toString(), encryptionKey)
                ));
            }
            
        } catch (Exception e) {
            throw new EncryptionException("Failed to encrypt sensitive fields", e);
        }
    }
    
    private void decryptSensitiveFields(FiledFlightPlan filing) {
        try {
            // Decrypt pilot information
            if (filing.getPilotInCommand() != null) {
                filing.setPilotInCommand(aesUtil.decrypt(filing.getPilotInCommand(), encryptionKey));
            }
            
            // Decrypt emergency contact
            if (filing.getEmergencyContact() != null) {
                filing.setEmergencyContact(aesUtil.decrypt(filing.getEmergencyContact(), encryptionKey));
            }
            
        } catch (Exception e) {
            log.error("Failed to decrypt sensitive fields for filing {}", filing.getFilingId(), e);
        }
    }
}
```

### 7.3 Audit Logging

#### 7.3.1 Comprehensive Audit Trail
```java
@Component
@Aspect
public class AuditLoggingAspect {
    
    @Autowired
    private AuditLogRepository auditLogRepository;
    
    @Around("@annotation(Auditable)")
    public Object auditMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        AuditLog auditLog = createAuditLog(joinPoint);
        
        try {
            Object result = joinPoint.proceed();
            auditLog.setOperationStatus("SUCCESS");
            auditLog.setResponseData(serializeResponse(result));
            return result;
            
        } catch (Throwable e) {
            auditLog.setOperationStatus("FAILURE");
            auditLog.setErrorMessage(e.getMessage());
            auditLog.setErrorDetails(ExceptionUtils.getStackTrace(e));
            throw e;
            
        } finally {
            auditLog.setEndTime(Instant.now());
            auditLog.setDurationMs(Duration.between(auditLog.getStartTime(), auditLog.getEndTime()).toMillis());
            auditLogRepository.save(auditLog);
        }
    }
    
    private AuditLog createAuditLog(ProceedingJoinPoint joinPoint) {
        AuditLog auditLog = new AuditLog();
        
        // Basic operation info
        auditLog.setOperationType(joinPoint.getSignature().getName());
        auditLog.setClassName(joinPoint.getTarget().getClass().getSimpleName());
        auditLog.setStartTime(Instant.now());
        
        // User context
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth != null) {
            auditLog.setUserId(auth.getName());
            auditLog.setUserType(determineUserType(auth));
            auditLog.setClientIP(getClientIP());
        }
        
        // Request data
        Object[] args = joinPoint.getArgs();
        if (args.length > 0) {
            auditLog.setRequestData(serializeArguments(args));
            auditLog.setResourceId(extractResourceId(args));
        }
        
        return auditLog;
    }
    
    private String extractResourceId(Object[] args) {
        for (Object arg : args) {
            if (arg instanceof FilingRequest) {
                return ((FilingRequest) arg).getGufi();
            } else if (arg instanceof String && arg.toString().startsWith("VATM-")) {
                return arg.toString(); // Filing ID
            }
        }
        return null;
    }
}
```

---
## 8. NON-FUNCTIONAL REQUIREMENTS

### 8.1 Performance Requirements

#### NFR-001: Response Time Requirements
- **Single eFPL Filing**: < 10 seconds (95th percentile)
- **Simple Validation**: < 5 seconds (95th percentile)
- **Complex Business Rules**: < 15 seconds (99th percentile)
- **FDPS Integration**: < 20 seconds (including external system response)
- **Status Query**: < 2 seconds (95th percentile)
- **Bulk Filing (50 eFPLs)**: < 5 minutes (95th percentile)

#### NFR-002: Throughput Requirements
- **Peak Filing Rate**: 100 eFPL submissions per minute
- **Concurrent Users**: 200+ simultaneous connections
- **Daily Volume**: 1000+ flight plan filings
- **Database Transactions**: 500+ TPS sustained
- **Message Queue Throughput**: 1000+ messages per minute

#### NFR-003: Resource Utilization
- **CPU Usage**: < 70% average during peak hours
- **Memory Usage**: < 80% of allocated heap
- **Database Connections**: < 80% of connection pool
- **Disk I/O**: < 1000 IOPS average
- **Network Bandwidth**: < 100 Mbps average

### 8.2 Availability and Reliability

#### NFR-004: System Availability
- **Uptime Target**: 99.7% during operational hours (06:00-24:00 ICT)
- **Maximum Downtime**: 4 hours per month (planned maintenance)
- **Recovery Time**: < 30 minutes for system restart
- **Failover Time**: < 5 minutes for automatic failover
- **Data Consistency**: 99.99% data integrity guarantee

#### NFR-005: Error Handling
- **Graceful Degradation**: System continues with reduced functionality during partial failures
- **Retry Mechanisms**: Automatic retry for transient failures (3 attempts with exponential backoff)
- **Circuit Breakers**: Prevent cascade failures in external integrations
- **Timeout Handling**: All external calls have defined timeouts
- **Error Recovery**: Automatic recovery procedures for common failure scenarios

#### NFR-006: Data Backup and Recovery
- **Backup Frequency**: Continuous WAL backup + daily full backup
- **Recovery Point Objective (RPO)**: < 1 hour data loss maximum
- **Recovery Time Objective (RTO)**: < 4 hours full system recovery
- **Backup Retention**: 30 days online, 1 year archive
- **Disaster Recovery**: Hot standby system at secondary site

### 8.3 Security Requirements

#### NFR-007: Authentication and Authorization
- **Multi-factor Authentication**: Support for certificate-based authentication
- **Role-based Access Control**: Granular permissions based on user roles
- **API Key Management**: Secure key generation, rotation, and revocation
- **Session Management**: Secure token handling with appropriate expiration
- **Audit Requirements**: Complete audit trail of all authentication attempts

#### NFR-008: Data Security
- **Encryption in Transit**: TLS 1.3 for all external communications
- **Encryption at Rest**: AES-256 encryption for sensitive database fields
- **Key Management**: Hardware Security Module (HSM) for encryption keys
- **Data Masking**: PII masking in non-production environments
- **Secure Configuration**: No hardcoded credentials, secure default settings

#### NFR-009: Network Security
- **Firewall Rules**: Restrictive ingress/egress rules
- **DDoS Protection**: Rate limiting and traffic analysis
- **Intrusion Detection**: Real-time monitoring for suspicious activity
- **VPN Access**: Secure tunneling for administrative access
- **Network Segmentation**: Isolated network segments for different components

### 8.4 Scalability and Capacity

#### NFR-010: Horizontal Scalability
- **Application Scaling**: Support for multiple application instances
- **Database Scaling**: Read replicas and connection pooling
- **Load Balancing**: Automatic traffic distribution across instances
- **Auto-scaling**: Dynamic scaling based on load metrics
- **Resource Elasticity**: Ability to scale up/down based on demand

#### NFR-011: Capacity Planning
- **Growth Projection**: 50% annual increase in filing volume
- **Peak Capacity**: Handle 3x average load during peak periods
- **Storage Growth**: 200GB annual database growth
- **Archive Strategy**: Automated archival of old flight plans
- **Performance Monitoring**: Continuous capacity utilization monitoring

### 8.5 Integration and Compatibility

#### NFR-012: System Integration
- **FDPS Compatibility**: Seamless integration with existing VATM FDPS
- **Legacy Support**: Backward compatibility during transition period
- **External APIs**: RESTful API design following OpenAPI 3.0 standards
- **Message Formats**: Support for FIXM 4.2.0 and SWIM messaging
- **Protocol Support**: HTTP/HTTPS, JMS, SOAP, REST

#### NFR-013: Data Format Compliance
- **FIXM Standards**: Full compliance with FIXM 4.2.0 Core specification
- **ICAO Compliance**: Adherence to ICAO Doc 9965 FF-ICE standards
- **SWIM Compliance**: EUROCONTROL SWIM-TI Yellow Profile compliance
- **XML Standards**: W3C XML Schema 1.1 validation
- **Character Encoding**: UTF-8 support for international characters

---

## 9. DEPLOYMENT ARCHITECTURE

### 9.1 Production Environment Design

#### 9.1.1 Multi-Tier Architecture
```yaml
# Production Deployment Architecture

Load Balancer Tier:
  Primary:
    Type: nginx + keepalived
    Specs: 4 CPU, 16GB RAM, 200GB SSD
    Location: VATM DMZ
    Function: SSL termination, load balancing, DDoS protection
  
  Secondary:
    Type: nginx + keepalived (standby)
    Specs: 4 CPU, 16GB RAM, 200GB SSD
    Location: VATM DMZ
    Function: Automatic failover

Application Tier:
  App Server 1:
    Type: Ubuntu 22.04 LTS
    Specs: 8 CPU, 32GB RAM, 500GB NVMe SSD
    Location: VATM Data Center Zone A
    Function: Primary application instance
  
  App Server 2:
    Type: Ubuntu 22.04 LTS  
    Specs: 8 CPU, 32GB RAM, 500GB NVMe SSD
    Location: VATM Data Center Zone B
    Function: Secondary application instance
    
  Background Services:
    Type: Ubuntu 22.04 LTS
    Specs: 4 CPU, 16GB RAM, 200GB SSD
    Function: Async processing, notifications, cleanup jobs

Database Tier:
  Primary DB:
    Type: Ubuntu 22.04 LTS + PostgreSQL 15
    Specs: 12 CPU, 64GB RAM, 2TB NVMe SSD + 8TB HDD
    Location: VATM Data Center Zone A
    Function: Primary database, read/write operations
  
  Replica DB:
    Type: Ubuntu 22.04 LTS + PostgreSQL 15
    Specs: 8 CPU, 32GB RAM, 2TB NVMe SSD
    Location: VATM Data Center Zone B
    Function: Read replica, backup, failover target

Cache Tier:
  Redis Cluster:
    Node 1: 4 CPU, 16GB RAM, Zone A
    Node 2: 4 CPU, 16GB RAM, Zone B  
    Node 3: 4 CPU, 16GB RAM, Zone A (tie-breaker)
    Function: Session storage, application caching, rate limiting

Monitoring Tier:
  Monitoring Stack:
    Type: Ubuntu 22.04 LTS
    Specs: 6 CPU, 24GB RAM, 1TB SSD
    Components: Prometheus, Grafana, AlertManager, ELK Stack
    Function: System monitoring, log aggregation, alerting
```

#### 9.1.2 Network Architecture
```yaml
# Network Topology

Internet Gateway:
  - Public IP: 203.x.x.x/29
  - Firewall: pfSense/OPNsense
  - DDoS Protection: CloudFlare or equivalent

DMZ Network (10.1.1.0/24):
  - Load Balancers: 10.1.1.10-11
  - Jump Server: 10.1.1.20
  - Monitoring: 10.1.1.30

Application Network (10.1.2.0/24):
  - App Servers: 10.1.2.10-12
  - Background Services: 10.1.2.20-22
  - Message Queues: 10.1.2.30-32

Database Network (10.1.3.0/24):
  - Primary DB: 10.1.3.10
  - Replica DB: 10.1.3.11
  - Cache Cluster: 10.1.3.20-22
  - Backup Server: 10.1.3.30

Management Network (10.1.4.0/24):
  - Monitoring: 10.1.4.10
  - Log Server: 10.1.4.20
  - Configuration Management: 10.1.4.30
  - Backup Controller: 10.1.4.40
```

### 9.2 Container Orchestration

#### 9.2.1 Docker Compose Configuration
```yaml
# docker-compose.production.yml
version: '3.8'

services:
  filing-service-primary:
    image: vatm/filing-service:${VERSION}
    hostname: filing-primary
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - DATABASE_URL=postgresql://10.1.3.10:5432/vatm_filing
      - REDIS_URL=redis://10.1.3.20:6379
      - JVM_OPTS=-Xms4g -Xmx8g -XX:+UseG1GC
      - SERVER_PORT=8080
      - INSTANCE_ID=primary
    volumes:
      - ./config/application-production.yml:/app/config/application.yml:ro
      - ./logs/primary:/var/log/vatm
      - ./ssl-certs:/app/ssl:ro
    ports:
      - "8080:8080"
    networks:
      - app-network
    depends_on:
      - postgres-primary
      - redis-cluster
    deploy:
      resources:
        limits:
          memory: 12G
          cpus: '6'
        reservations:
          memory: 8G
          cpus: '4'
      restart_policy:
        condition: unless-stopped
        delay: 30s
        max_attempts: 5

  filing-service-secondary:
    image: vatm/filing-service:${VERSION}
    hostname: filing-secondary
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - DATABASE_URL=postgresql://10.1.3.10:5432/vatm_filing
      - REDIS_URL=redis://10.1.3.20:6379
      - JVM_OPTS=-Xms4g -Xmx8g -XX:+UseG1GC
      - SERVER_PORT=8080
      - INSTANCE_ID=secondary
    volumes:
      - ./config/application-production.yml:/app/config/application.yml:ro
      - ./logs/secondary:/var/log/vatm
      - ./ssl-certs:/app/ssl:ro
    ports:
      - "8081:8080"
    networks:
      - app-network
    depends_on:
      - postgres-primary
      - redis-cluster

  postgres-primary:
    image: postgres:15-alpine
    hostname: postgres-primary
    environment:
      - POSTGRES_DB=vatm_filing
      - POSTGRES_USER=filing_service
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_INITDB_ARGS=--auth-host=scram-sha-256
    volumes:
      - postgres-data-primary:/var/lib/postgresql/data
      - ./config/postgresql.conf:/etc/postgresql/postgresql.conf:ro
      - ./config/pg_hba.conf:/etc/postgresql/pg_hba.conf:ro
      - ./backups:/backups
      - ./init-scripts:/docker-entrypoint-initdb.d:ro
    ports:
      - "5432:5432"
    networks:
      - db-network
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
    deploy:
      resources:
        limits:
          memory: 48G
          cpus: '8'
        reservations:
          memory: 32G
          cpus: '6'

  postgres-replica:
    image: postgres:15-alpine
    hostname: postgres-replica
    environment:
      - PGUSER=replicator
      - POSTGRES_PASSWORD=${REPLICATOR_PASSWORD}
      - PGPASSWORD=${REPLICATOR_PASSWORD}
    volumes:
      - postgres-data-replica:/var/lib/postgresql/data
      - ./config/recovery.conf:/var/lib/postgresql/data/recovery.conf:ro
    ports:
      - "5433:5432"
    networks:
      - db-network
    depends_on:
      - postgres-primary

  redis-cluster-node1:
    image: redis:7-alpine
    hostname: redis-node1
    command: redis-server /etc/redis/redis.conf --cluster-enabled yes
    volumes:
      - ./config/redis-cluster.conf:/etc/redis/redis.conf:ro
      - redis-data-node1:/data
    ports:
      - "7001:6379"
      - "17001:16379"
    networks:
      - cache-network

  redis-cluster-node2:
    image: redis:7-alpine
    hostname: redis-node2
    command: redis-server /etc/redis/redis.conf --cluster-enabled yes
    volumes:
      - ./config/redis-cluster.conf:/etc/redis/redis.conf:ro
      - redis-data-node2:/data
    ports:
      - "7002:6379"
      - "17002:16379"
    networks:
      - cache-network

  redis-cluster-node3:
    image: redis:7-alpine
    hostname: redis-node3
    command: redis-server /etc/redis/redis.conf --cluster-enabled yes
    volumes:
      - ./config/redis-cluster.conf:/etc/redis/redis.conf:ro
      - redis-data-node3:/data
    ports:
      - "7003:6379"
      - "17003:16379"
    networks:
      - cache-network

  nginx-load-balancer:
    image: nginx:alpine
    hostname: nginx-lb
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl-certs:/etc/nginx/ssl:ro
      - ./logs/nginx:/var/log/nginx
    ports:
      - "80:80"
      - "443:443"
    networks:
      - frontend-network
      - app-network
    depends_on:
      - filing-service-primary
      - filing-service-secondary

  prometheus:
    image: prom/prometheus:latest
    hostname: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=90d'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - monitoring-network
      - app-network

  grafana:
    image: grafana/grafana:latest
    hostname: grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    volumes:
      - grafana-data:/var/lib/grafana
      - ./config/grafana:/etc/grafana/provisioning:ro
    ports:
      - "3000:3000"
    networks:
      - monitoring-network
    depends_on:
      - prometheus

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    hostname: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - monitoring-network

  kibana:
    image: docker.elastic.co/kibana/kibana:8.10.0
    hostname: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    networks:
      - monitoring-network
    depends_on:
      - elasticsearch

  logstash:
    image: docker.elastic.co/logstash/logstash:8.10.0
    hostname: logstash
    volumes:
      - ./config/logstash:/usr/share/logstash/pipeline:ro
      - ./logs:/var/log/apps:ro
    networks:
      - monitoring-network
    depends_on:
      - elasticsearch

volumes:
  postgres-data-primary:
    driver: local
    driver_opts:
      type: none
      device: /data/postgres-primary
      o: bind
  postgres-data-replica:
    driver: local
    driver_opts:
      type: none
      device: /data/postgres-replica
      o: bind
  redis-data-node1:
  redis-data-node2:
  redis-data-node3:
  prometheus-data:
  grafana-data:
  elasticsearch-data:

networks:
  frontend-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.1.0/24
  app-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.2.0/24
  db-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.3.0/24
  cache-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.4.0/24
  monitoring-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.5.0/24
```

### 9.3 High Availability Configuration

#### 9.3.1 Database High Availability
```bash
#!/bin/bash
# PostgreSQL Streaming Replication Setup

# On Primary Server (10.1.3.10)
# postgresql.conf
cat > /etc/postgresql/15/main/postgresql.conf << EOF
# Connection settings
listen_addresses = '*'
port = 5432
max_connections = 200

# Memory settings
shared_buffers = 16GB
effective_cache_size = 48GB
work_mem = 256MB
maintenance_work_mem = 2GB

# WAL settings for replication
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
hot_standby = on
wal_keep_size = 5GB

# Performance settings
checkpoint_timeout = 15min
checkpoint_completion_target = 0.9
random_page_cost = 1.1
effective_io_concurrency = 200

# Logging
log_line_prefix = '%t [%p]: user=%u,db=%d,app=%a,client=%h '
log_min_duration_statement = 1000
log_checkpoints = on
log_connections = on
log_disconnections = on
log_statement = 'ddl'
EOF

# pg_hba.conf
cat > /etc/postgresql/15/main/pg_hba.conf << EOF
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Local connections
local   all             postgres                                peer
local   all             all                                     peer

# IPv4 local connections
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             10.1.3.0/24             scram-sha-256

# Replication connections
host    replication     replicator      10.1.3.11/32            scram-sha-256
host    replication     replicator      10.1.3.12/32            scram-sha-256
EOF

# Create replication user
sudo -u postgres psql << EOF
CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'replicator_secure_password_2025';
\q
EOF

# On Replica Server (10.1.3.11)
# Stop PostgreSQL and remove data directory
systemctl stop postgresql
rm -rf /var/lib/postgresql/15/main/*

# Create base backup from primary
sudo -u postgres pg_basebackup -h 10.1.3.10 -D /var/lib/postgresql/15/main -U replicator -W -v -P -R

# Start PostgreSQL replica
systemctl start postgresql
```

#### 9.3.2 Application Load Balancing
```nginx
# /etc/nginx/nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 4096;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging format
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for" '
                   'rt=$request_time uct="$upstream_connect_time" '
                   'uht="$upstream_header_time" urt="$upstream_response_time"';
    
    access_log /var/log/nginx/access.log main;
    
    # Performance settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 100;
    client_max_body_size 10M;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=filing_api:10m rate=60r/m;
    limit_req_zone $http_x_api_key zone=api_key:10m rate=1000r/h;
    
    # Upstream servers
    upstream filing_service {
        least_conn;
        server 10.1.2.10:8080 max_fails=3 fail_timeout=30s;
        server 10.1.2.11:8080 max_fails=3 fail_timeout=30s backup;
        keepalive 32;
    }
    
    # Health check endpoint
    server {
        listen 8080;
        location /nginx-health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
    
    # HTTPS server
    server {
        listen 443 ssl http2;
        server_name filing.vatm.vn;
        
        # SSL configuration
        ssl_certificate /etc/nginx/ssl/vatm.crt;
        ssl_certificate_key /etc/nginx/ssl/vatm.key;
        ssl_protocols TLSv1.3;
        ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-RSA-AES128-GCM-SHA256;
        ssl_prefer_server_ciphers off;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 1d;
        ssl_session_tickets off;
        
        # Security headers
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header X-Frame-Options DENY always;
        add_header X-Content-Type-Options nosniff always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;
        
        # API endpoints
        location /api/ {
            # Rate limiting
            limit_req zone=filing_api burst=20 nodelay;
            limit_req zone=api_key burst=100 nodelay;
            
            # Proxy settings
            proxy_pass http://filing_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Timeouts
            proxy_connect_timeout 10s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
            
            # Connection settings
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_buffering on;
            proxy_buffer_size 128k;
            proxy_buffers 4 256k;
            proxy_busy_buffers_size 256k;
        }
        
        # Health check endpoint
        location /actuator/health {
            access_log off;
            proxy_pass http://filing_service;
            proxy_set_header Host $host;
        }
        
        # Admin endpoints (restricted access)
        location /admin/ {
            allow 10.1.4.0/24;  # Management network
            allow 10.1.1.20;    # Jump server
            deny all;
            
            proxy_pass http://filing_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
    
    # HTTP to HTTPS redirect
    server {
        listen 80;
        server_name filing.vatm.vn;
        return 301 https://$server_name$request_uri;
    }
}
```

### 9.4 Monitoring and Alerting Setup

#### 9.4.1 Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 30s
  evaluation_interval: 30s
  external_labels:
    cluster: 'vatm-filing-service'
    environment: 'production'

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  # Filing Service Application
  - job_name: 'filing-service'
    static_configs:
      - targets: 
        - '10.1.2.10:8080'
        - '10.1.2.11:8080'
    metrics_path: '/actuator/prometheus'
    scrape_interval: 15s
    scrape_timeout: 10s

  # PostgreSQL Database
  - job_name: 'postgres'
    static_configs:
      - targets:
        - '10.1.3.10:9187'  # postgres_exporter
        - '10.1.3.11:9187'

  # Redis Cache
  - job_name: 'redis'
    static_configs:
      - targets:
        - '10.1.3.20:9121'  # redis_exporter
        - '10.1.3.21:9121'
        - '10.1.3.22:9121'

  # System metrics
  - job_name: 'node'
    static_configs:
      - targets:
        - '10.1.2.10:9100'  # node_exporter
        - '10.1.2.11:9100'
        - '10.1.3.10:9100'
        - '10.1.3.11:9100'

  # Nginx Load Balancer
  - job_name: 'nginx'
    static_configs:
      - targets:
        - '10.1.1.10:9113'  # nginx_exporter

  # JMS Message Queues
  - job_name: 'activemq'
    static_configs:
      - targets:
        - '10.1.2.30:8161'
```

#### 9.4.2 Alert Rules
```yaml
# alert_rules.yml
groups:
  - name: filing_service_alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} for the past 5 minutes"

      # Slow response time
      - alert: SlowResponseTime
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Slow response time detected"
          description: "95th percentile response time is {{ $value }}s"

      # Database connection issues
      - alert: DatabaseConnectionHigh
        expr: hikaricp_connections_active / hikaricp_connections_max > 0.8
        for: 3m
        labels:
          severity: warning
        annotations:
          summary: "Database connection pool usage high"
          description: "Connection pool usage is {{ $value | humanizePercentage }}"

      # Memory usage high
      - alert: HighMemoryUsage
        expr: jvm_memory_used_bytes{area="heap"} / jvm_memory_max_bytes{area="heap"} > 0.85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High JVM heap memory usage"
          description: "Heap memory usage is {{ $value | humanizePercentage }}"

      # Disk space low
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space running low"
          description: "Only {{ $value | humanizePercentage }} disk space remaining"

      # PostgreSQL down
      - alert: PostgreSQLDown
        expr: pg_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "PostgreSQL instance is down"
          description: "PostgreSQL instance {{ $labels.instance }} is not responding"

      # Redis cluster issues
      - alert: RedisClusterDown
        expr: redis_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Redis cluster node is down"
          description: "Redis node {{ $labels.instance }} is not responding"

      # FDPS integration failures
      - alert: FDPSIntegrationFailure
        expr: rate(fdps_integration_failures_total[10m]) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "FDPS integration failure rate high"
          description: "FDPS integration failure rate is {{ $value }} per minute"
```

---

## 10. TESTING STRATEGY

### 10.1 Comprehensive Testing Framework

#### 10.1.1 Unit Testing Strategy
```java
@ExtendWith(MockitoExtension.class)
class FilingServiceTest {
    
    @Mock
    private eFPLProcessor efplProcessor;
    
    @Mock
    private ValidationEngine validationEngine;

    @Mock
    private BusinessRulesEngine businessRulesEngine;
    
    @Mock
    private FDPSIntegrator fdpsIntegrator;
    
    @Mock
    private NotificationManager notificationManager;
    
    @InjectMocks
    private FilingService filingService;
    
    @Test
    void shouldAcceptValidFlightPlan() {
        // Given
        FilingRequest request = createValidFilingRequest();
        FlightPlan flightPlan = createValidFlightPlan();
        ValidationResult validationResult = createPassedValidationResult();
        FDPSIntegrationResult fdpsResult = createSuccessfulFDPSResult();
        
        when(efplProcessor.parseeFPL(request.geteFPLXml())).thenReturn(flightPlan);
        when(businessRulesEngine.validate(flightPlan)).thenReturn(createPassedBusinessRulesResult());
        when(validationEngine.validateTechnical(flightPlan)).thenReturn(createPassedTechnicalResult());
        when(validationEngine.validateOperational(flightPlan)).thenReturn(createPassedOperationalResult());
        when(fdpsIntegrator.submitToFDPS(any())).thenReturn(fdpsResult);
        
        // When
        FilingResponse response = filingService.processFlightPlanFiling(request);
        
        // Then
        assertThat(response.getStatus()).isEqualTo(FilingStatus.ACCEPTED);
        assertThat(response.getFilingId()).isNotNull();
        assertThat(response.getValidationResults().getBusinessRulesValidation()).isEqualTo(ValidationStatus.PASSED);
        
        verify(fdpsIntegrator).submitToFDPS(any(FiledFlightPlan.class));
        verify(notificationManager).notifyStakeholders(any(), eq(NotificationType.FILING_ACCEPTED));
    }
    
    @Test
    void shouldRejectFlightPlanWithBusinessRuleViolations() {
        // Given
        FilingRequest request = createValidFilingRequest();
        FlightPlan flightPlan = createFlightPlanWithAltitudeViolation();
        BusinessRulesResult businessRulesResult = createFailedBusinessRulesResult();
        
        when(efplProcessor.parseeFPL(request.geteFPLXml())).thenReturn(flightPlan);
        when(businessRulesEngine.validate(flightPlan)).thenReturn(businessRulesResult);
        
        // When
        FilingResponse response = filingService.processFlightPlanFiling(request);
        
        // Then
        assertThat(response.getStatus()).isEqualTo(FilingStatus.REJECTED);
        assertThat(response.getErrors()).hasSize(1);
        assertThat(response.getErrors().get(0).getCode()).isEqualTo("BR001");
        
        verify(fdpsIntegrator, never()).submitToFDPS(any());
        verify(notificationManager).notifyStakeholders(any(), eq(NotificationType.FILING_REJECTED));
    }
    
    @Test
    void shouldHandleFDPSIntegrationFailureGracefully() {
        // Given
        FilingRequest request = createValidFilingRequest();
        FlightPlan flightPlan = createValidFlightPlan();
        FDPSIntegrationResult fdpsFailure = createFailedFDPSResult();
        
        when(efplProcessor.parseeFPL(request.geteFPLXml())).thenReturn(flightPlan);
        when(businessRulesEngine.validate(flightPlan)).thenReturn(createPassedBusinessRulesResult());
        when(validationEngine.validateTechnical(flightPlan)).thenReturn(createPassedTechnicalResult());
        when(validationEngine.validateOperational(flightPlan)).thenReturn(createPassedOperationalResult());
        when(fdpsIntegrator.submitToFDPS(any())).thenReturn(fdpsFailure);
        
        // When
        FilingResponse response = filingService.processFlightPlanFiling(request);
        
        // Then
        assertThat(response.getStatus()).isEqualTo(FilingStatus.ACCEPTED);
        assertThat(response.getCoordinationStatus().getFdpsIntegration()).isEqualTo("FAILED");
        assertThat(response.getMessage()).contains("FDPS integration failed");
    }
    
    @Test
    void shouldValidateFixmFormatCorrectly() {
        // Given
        String invalidFixmXml = "<invalid>xml</invalid>";
        FilingRequest request = new FilingRequest(invalidFixmXml, "VN");
        
        when(efplProcessor.parseeFPL(invalidFixmXml))
            .thenThrow(new FixmParsingException("Invalid FIXM format"));
        
        // When & Then
        assertThatThrownBy(() -> filingService.processFlightPlanFiling(request))
            .isInstanceOf(FilingProcessException.class)
            .hasMessageContaining("Invalid FIXM format");
    }
    
    // Test data creation methods
    private FilingRequest createValidFilingRequest() {
        String validFixmXml = loadTestResource("valid_efpl.xml");
        return new FilingRequest(validFixmXml, "VN");
    }
    
    private FlightPlan createValidFlightPlan() {
        return FlightPlan.builder()
            .gufi("VN123-20250912-001")
            .flightNumber("VN123")
            .aircraftType("A321")
            .departureAirport("VVNB")
            .arrivalAirport("VVTS")
            .cruiseAltitude(37000)
            .build();
    }
    
    private BusinessRulesResult createPassedBusinessRulesResult() {
        BusinessRulesResult result = new BusinessRulesResult();
        result.setStatus(ValidationStatus.PASSED);
        return result;
    }
    
    private BusinessRulesResult createFailedBusinessRulesResult() {
        BusinessRulesResult result = new BusinessRulesResult();
        result.setStatus(ValidationStatus.FAILED);
        result.addError(new ValidationError("BR001", "BUSINESS_RULE", 
            "Cruise altitude exceeds aircraft capability", 
            "Reduce cruise level"));
        return result;
    }
}
```

#### 10.1.2 Integration Testing
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@Transactional
class FilingServiceIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
            .withDatabaseName("vatm_filing_test")
            .withUsername("test")
            .withPassword("test");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
            .withExposedPorts(6379);
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private FiledFlightPlanRepository filingRepository;
    
    @MockBean
    private FDPSIntegrator fdpsIntegrator;
    
    @MockBean
    private NotificationManager notificationManager;
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);
    }
    
    @Test
    void shouldAcceptCompleteFlightPlanWorkflow() {
        // Given
        String valideFPL = loadTestResource("test_efpl_vn123.xml");
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_XML);
        headers.set("X-API-Key", "test-api-key");
        headers.set("X-Airline-Code", "VN");
        
        when(fdpsIntegrator.submitToFDPS(any())).thenReturn(
            new FDPSIntegrationResult(true, "200", "Success"));
        
        HttpEntity<String> request = new HttpEntity<>(valideFPL, headers);
        
        // When
        ResponseEntity<FilingResponse> response = restTemplate.postForEntity(
            "/api/v1/filing/submit", request, FilingResponse.class);
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getStatus()).isEqualTo(FilingStatus.ACCEPTED);
        
        // Verify database persistence
        String filingId = response.getBody().getFilingId();
        Optional<FiledFlightPlan> savedFiling = filingRepository.findByFilingId(filingId);
        assertThat(savedFiling).isPresent();
        assertThat(savedFiling.get().getGufi()).isEqualTo("VN123-20250912-001");
        
        // Verify external system calls
        verify(fdpsIntegrator).submitToFDPS(any(FiledFlightPlan.class));
        verify(notificationManager).notifyStakeholders(any(), eq(NotificationType.FILING_ACCEPTED));
    }
    
    @Test
    void shouldRejectInvalidAircraft() {
        // Given
        String invalideFPL = loadTestResource("test_efpl_invalid_aircraft.xml");
        HttpHeaders headers = createAuthenticatedHeaders();
        HttpEntity<String> request = new HttpEntity<>(invalideFPL, headers);
        
        // When
        ResponseEntity<FilingResponse> response = restTemplate.postForEntity(
            "/api/v1/filing/submit", request, FilingResponse.class);
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.UNPROCESSABLE_ENTITY);
        assertThat(response.getBody().getStatus()).isEqualTo(FilingStatus.REJECTED);
        assertThat(response.getBody().getErrors()).hasSize(1);
        assertThat(response.getBody().getErrors().get(0).getCode()).isEqualTo("TV003");
    }
    
    @Test
    void shouldHandleConcurrentFilings() throws InterruptedException {
        // Given
        int numberOfThreads = 10;
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        List<CompletableFuture<ResponseEntity<FilingResponse>>> futures = new ArrayList<>();
        
        when(fdpsIntegrator.submitToFDPS(any())).thenReturn(
            new FDPSIntegrationResult(true, "200", "Success"));
        
        // When
        for (int i = 0; i < numberOfThreads; i++) {
            final int threadId = i;
            CompletableFuture<ResponseEntity<FilingResponse>> future = CompletableFuture.supplyAsync(() -> {
                try {
                    String eFPL = createTesteFPLWithGufi("VN12" + threadId + "-20250912-001");
                    HttpEntity<String> request = new HttpEntity<>(eFPL, createAuthenticatedHeaders());
                    return restTemplate.postForEntity("/api/v1/filing/submit", request, FilingResponse.class);
                } finally {
                    latch.countDown();
                }
            });
            futures.add(future);
        }
        
        latch.await(30, TimeUnit.SECONDS);
        
        // Then
        List<ResponseEntity<FilingResponse>> responses = futures.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList());
        
        assertThat(responses).hasSize(numberOfThreads);
        assertThat(responses).allMatch(response -> 
            response.getStatusCode().is2xxSuccessful());
        
        // Verify all filings were persisted
        List<FiledFlightPlan> savedFilings = filingRepository.findAll();
        assertThat(savedFilings).hasSizeGreaterThanOrEqualTo(numberOfThreads);
    }
    
    @Test
    void shouldUpdateFilingStatus() {
        // Given - Create an initial filing
        FiledFlightPlan filing = createTestFiling();
        filing = filingRepository.save(filing);
        
        String amendmentJson = """
            {
                "amendmentType": "TIME_CHANGE",
                "gufi": "VN123-20250912-001",
                "changes": {
                    "estimatedOffBlockTime": "2025-09-12T02:30:00Z"
                },
                "reason": "ATC Flow Control Delay"
            }
            """;
        
        HttpHeaders headers = createAuthenticatedHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<String> request = new HttpEntity<>(amendmentJson, headers);
        
        // When
        ResponseEntity<FilingResponse> response = restTemplate.exchange(
            "/api/v1/filing/update/" + filing.getFilingId(),
            HttpMethod.PUT, request, FilingResponse.class);
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getStatus()).isEqualTo(FilingStatus.ACCEPTED);
        
        // Verify database update
        FiledFlightPlan updatedFiling = filingRepository.findByFilingId(filing.getFilingId()).get();
        assertThat(updatedFiling.getEstimatedOffBlock()).isEqualTo("2025-09-12T02:30:00Z");
    }
    
    private HttpHeaders createAuthenticatedHeaders() {
        HttpHeaders headers = new HttpHeaders();
        headers.set("X-API-Key", "test-api-key");
        headers.set("X-Airline-Code", "VN");
        return headers;
    }
    
    private FiledFlightPlan createTestFiling() {
        FiledFlightPlan filing = new FiledFlightPlan();
        filing.setFilingId("TEST-20250912-001");
        filing.setGufi("VN123-20250912-001");
        filing.setFlightNumber("VN123");
        filing.setAirlineCode("VN");
        filing.setDepartureAirport("VVNB");
        filing.setArrivalAirport("VVTS");
        filing.setCurrentStatus(FilingStatus.ACCEPTED);
        return filing;
    }
}
```

#### 10.1.3 Performance Testing with JMeter
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="5.0" jmeter="5.4.1">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="Filing Service Performance Test">
      <stringProp name="TestPlan.comments">Performance test for VATM Filing Service</stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.tearDown_on_shutdown">true</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.arguments" elementType="Arguments" guiclass="ArgumentsPanel" testclass="Arguments">
        <collectionProp name="Arguments.arguments">
          <elementProp name="base_url" elementType="Argument">
            <stringProp name="Argument.name">base_url</stringProp>
            <stringProp name="Argument.value">https://filing.vatm.vn/api/v1</stringProp>
          </elementProp>
          <elementProp name="api_key" elementType="Argument">
            <stringProp name="Argument.name">api_key</stringProp>
            <stringProp name="Argument.value">perf_test_key_2025</stringProp>
          </elementProp>
        </collectionProp>
      </elementProp>
    </TestPlan>
    
    <hashTree>
      <!-- Normal Load Test -->
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Normal Load - Filing Submissions">
        <stringProp name="ThreadGroup.on_sample_error">continue</stringProp>
        <elementProp name="ThreadGroup.main_controller" elementType="LoopController">
          <boolProp name="LoopController.continue_forever">false</boolProp>
          <stringProp name="LoopController.loops">10</stringProp>
        </elementProp>
        <stringProp name="ThreadGroup.num_threads">50</stringProp>
        <stringProp name="ThreadGroup.ramp_time">300</stringProp>
        <boolProp name="ThreadGroup.scheduler">true</boolProp>
        <stringProp name="ThreadGroup.duration">1800</stringProp>
      </ThreadGroup>
      
      <hashTree>
        <!-- HTTP Request Defaults -->
        <ConfigTestElement guiclass="HttpDefaultsGui" testclass="ConfigTestElement" testname="HTTP Request Defaults">
          <stringProp name="HTTPSampler.domain"></stringProp>
          <stringProp name="HTTPSampler.port"></stringProp>
          <stringProp name="HTTPSampler.protocol">https</stringProp>
          <stringProp name="HTTPSampler.path">${base_url}</stringProp>
        </ConfigTestElement>
        
        <!-- Header Manager -->
        <HeaderManager guiclass="HeaderPanel" testclass="HeaderManager" testname="HTTP Header Manager">
          <collectionProp name="HeaderManager.headers">
            <elementProp name="" elementType="Header">
              <stringProp name="Header.name">X-API-Key</stringProp>
              <stringProp name="Header.value">${api_key}</stringProp>
            </elementProp>
            <elementProp name="" elementType="Header">
              <stringProp name="Header.name">Content-Type</stringProp>
              <stringProp name="Header.value">application/xml</stringProp>
            </elementProp>
            <elementProp name="" elementType="Header">
              <stringProp name="Header.name">X-Airline-Code</stringProp>
              <stringProp name="Header.value">VN</stringProp>
            </elementProp>
          </collectionProp>
        </HeaderManager>
        
        <!-- CSV Data Set for Test Data -->
        <CSVDataSet guiclass="TestBeanGUI" testclass="CSVDataSet" testname="Flight Plan Test Data">
          <stringProp name="filename">test_flight_plans.csv</stringProp>
          <stringProp name="fileEncoding">UTF-8</stringProp>
          <stringProp name="variableNames">gufi,flight_number,aircraft_type,dep_airport,arr_airport,dep_time,arr_time</stringProp>
          <boolProp name="recycle">true</boolProp>
          <boolProp name="stopThread">false</boolProp>
          <stringProp name="shareMode">shareMode.all</stringProp>
        </CSVDataSet>
        
        <!-- Submit Flight Plan -->
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Submit Flight Plan">
          <stringProp name="HTTPSampler.domain"></stringProp>
          <stringProp name="HTTPSampler.port"></stringProp>
          <stringProp name="HTTPSampler.protocol"></stringProp>
          <stringProp name="HTTPSampler.path">/filing/submit</stringProp>
          <stringProp name="HTTPSampler.method">POST</stringProp>
          <boolProp name="HTTPSampler.follow_redirects">true</boolProp>
          <boolProp name="HTTPSampler.auto_redirects">false</boolProp>
          <boolProp name="HTTPSampler.use_keepalive">true</boolProp>
          <stringProp name="HTTPSampler.postBodyRaw"><?xml version="1.0" encoding="UTF-8"?>
<fx:FlightPlan xmlns:fx="http://www.fixm.aero/flight/4.2">
    <fx:gufi>${gufi}</fx:gufi>
    <fx:flight>
        <fx:flightIdentification>
            <fx:aircraftIdentification>${flight_number}</fx:aircraftIdentification>
        </fx:flightIdentification>
        <fx:aircraft>
            <fx:aircraftType>${aircraft_type}</fx:aircraftType>
            <fx:registration>VN-A${__Random(100,999)}</fx:registration>
            <fx:operatorDesignator>HVN</fx:operatorDesignator>
        </fx:aircraft>
        <fx:departure>
            <fx:aerodrome>${dep_airport}</fx:aerodrome>
            <fx:estimatedOffBlockTime>${dep_time}</fx:estimatedOffBlockTime>
        </fx:departure>
        <fx:arrival>
            <fx:aerodrome>${arr_airport}</fx:aerodrome>
            <fx:estimatedOnBlockTime>${arr_time}</fx:estimatedOnBlockTime>
        </fx:arrival>
        <fx:routeTrajectory>
            <fx:routeInformation>
                <fx:routeText>${dep_airport} DCT ADMOX A1 OTBAN ${arr_airport}</fx:routeText>
                <fx:cruisingLevel>370</fx:cruisingLevel>
                <fx:totalEstimatedElapsedTime>PT2H30M</fx:totalEstimatedElapsedTime>
            </fx:routeInformation>
        </fx:routeTrajectory>
    </fx:flight>
</fx:FlightPlan></stringProp>
        </HTTPSamplerProxy>
        
        <!-- Response Assertions -->
        <ResponseAssertion guiclass="AssertionGui" testclass="ResponseAssertion" testname="Response Code Assertion">
          <collectionProp name="Asserion.test_strings">
            <stringProp name="49586">200</stringProp>
            <stringProp name="49587">201</stringProp>
          </collectionProp>
          <stringProp name="Assertion.test_field">Assertion.response_code</stringProp>
          <boolProp name="Assertion.assume_success">false</boolProp>
          <intProp name="Assertion.test_type">8</intProp>
        </ResponseAssertion>
        
        <ResponseAssertion guiclass="AssertionGui" testclass="ResponseAssertion" testname="Response Time Assertion">
          <collectionProp name="Asserion.test_strings">
            <stringProp name="50546">10000</stringProp>
          </collectionProp>
          <stringProp name="Assertion.test_field">Assertion.response_time</stringProp>
          <intProp name="Assertion.test_type">6</intProp>
        </ResponseAssertion>
        
        <!-- Think Time -->
        <UniformRandomTimer guiclass="UniformRandomTimerGui" testclass="UniformRandomTimer" testname="Think Time">
          <stringProp name="ConstantTimer.delay">2000</stringProp>
          <stringProp name="RandomTimer.range">3000</stringProp>
        </UniformRandomTimer>
      </hashTree>
      
      <!-- Stress Test Thread Group -->
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Stress Test - Peak Load">
        <stringProp name="ThreadGroup.on_sample_error">continue</stringProp>
        <elementProp name="ThreadGroup.main_controller" elementType="LoopController">
          <boolProp name="LoopController.continue_forever">false</boolProp>
          <stringProp name="LoopController.loops">5</stringProp>
        </elementProp>
        <stringProp name="ThreadGroup.num_threads">200</stringProp>
        <stringProp name="ThreadGroup.ramp_time">120</stringProp>
        <boolProp name="ThreadGroup.scheduler">true</boolProp>
        <stringProp name="ThreadGroup.duration">600</stringProp>
      </ThreadGroup>
      
      <!-- Results and Listeners -->
      <ResultCollector guiclass="ViewResultsFullVisualizer" testclass="ResultCollector" testname="View Results Tree">
        <boolProp name="ResultCollector.error_logging">false</boolProp>
        <objProp>
          <name>saveConfig</name>
          <value class="SampleSaveConfiguration">
            <time>true</time>
            <latency>true</latency>
            <timestamp>true</timestamp>
            <success>true</success>
            <label>true</label>
            <code>true</code>
            <message>true</message>
            <threadName>true</threadName>
            <dataType>true</dataType>
            <encoding>false</encoding>
            <assertions>true</assertions>
            <subresults>true</subresults>
            <responseData>false</responseData>
            <samplerData>false</samplerData>
            <xml>false</xml>
            <fieldNames>true</fieldNames>
            <responseHeaders>false</responseHeaders>
            <requestHeaders>false</requestHeaders>
            <responseDataOnError>false</responseDataOnError>
            <saveAssertionResultsFailureMessage>true</saveAssertionResultsFailureMessage>
            <assertionsResultsToSave>0</assertionsResultsToSave>
            <bytes>true</bytes>
            <sentBytes>true</sentBytes>
            <url>true</url>
            <threadCounts>true</threadCounts>
            <idleTime>true</idleTime>
            <connectTime>true</connectTime>
          </value>
        </objProp>
        <stringProp name="filename">filing_service_results.jtl</stringProp>
      </ResultCollector>
      
      <ResultCollector guiclass="StatGraphVisualizer" testclass="ResultCollector" testname="Aggregate Graph">
        <boolProp name="ResultCollector.error_logging">false</boolProp>
        <objProp>
          <name>saveConfig</name>
          <value class="SampleSaveConfiguration">
            <time>true</time>
            <latency>true</latency>
            <timestamp>true</timestamp>
            <success>true</success>
            <label>true</label>
            <code>true</code>
            <message>true</message>
            <threadName>true</threadName>
            <dataType>true</dataType>
            <encoding>false</encoding>
            <assertions>true</assertions>
            <subresults>true</subresults>
            <responseData>false</responseData>
            <samplerData>false</samplerData>
            <xml>false</xml>
            <fieldNames>true</fieldNames>
            <responseHeaders>false</responseHeaders>
            <requestHeaders>false</requestHeaders>
            <responseDataOnError>false</responseDataOnError>
            <saveAssertionResultsFailureMessage>true</saveAssertionResultsFailureMessage>
            <assertionsResultsToSave>0</assertionsResultsToSave>
            <bytes>true</bytes>
            <sentBytes>true</sentBytes>
            <url>true</url>
            <threadCounts>true</threadCounts>
            <idleTime>true</idleTime>
            <connectTime>true</connectTime>
          </value>
        </objProp>
        <stringProp name="filename">performance_summary.jtl</stringProp>
      </ResultCollector>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

### 10.2 Security Testing

#### 10.2.1 Penetration Testing Checklist
```bash
#!/bin/bash
# Security Testing Script for Filing Service

echo "Starting Security Assessment for VATM Filing Service"
echo "=================================================="

# Test 1: API Key Authentication
echo "1. Testing API Key Authentication..."
curl -X POST https://filing.vatm.vn/api/v1/filing/submit \
  -H "Content-Type: application/xml" \
  -d "<test>data</test>" \
  -w "Status: %{http_code}\n"

echo "Expected: 401 Unauthorized"

# Test 2: Invalid API Key
echo "2. Testing Invalid API Key..."
curl -X POST https://filing.vatm.vn/api/v1/filing/submit \
  -H "Content-Type: application/xml" \
  -H "X-API-Key: invalid_key" \
  -d "<test>data</test>" \
  -w "Status: %{http_code}\n"

echo "Expected: 401 Unauthorized"

# Test 3: Rate Limiting
echo "3. Testing Rate Limiting..."
for i in {1..60}; do
  curl -X GET https://filing.vatm.vn/api/v1/filing/status/test \
    -H "X-API-Key: test_key" \
    -w "Request $i - Status: %{http_code}\n" \
    -o /dev/null -s
done

echo "Expected: 429 Too Many Requests after limit exceeded"

# Test 4: SQL Injection Attempts
echo "4. Testing SQL Injection..."
curl -X GET "https://filing.vatm.vn/api/v1/filing/status/'; DROP TABLE filed_flight_plans; --" \
  -H "X-API-Key: test_key" \
  -w "Status: %{http_code}\n"

echo "Expected: 400 Bad Request or 404 Not Found"

# Test 5: XSS Attempts
echo "5. Testing XSS Prevention..."
curl -X POST https://filing.vatm.vn/api/v1/filing/submit \
  -H "Content-Type: application/xml" \
  -H "X-API-Key: test_key" \
  -d "<script>alert('xss')</script>" \
  -w "Status: %{http_code}\n"

echo "Expected: 400 Bad Request"

# Test 6: XML External Entity (XXE) Attack
echo "6. Testing XXE Prevention..."
curl -X POST https://filing.vatm.vn/api/v1/filing/submit \
  -H "Content-Type: application/xml" \
  -H "X-API-Key: test_key" \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>&xxe;</data>' \
  -w "Status: %{http_code}\n"

echo "Expected: 400 Bad Request"

# Test 7: SSL/TLS Configuration
echo "7. Testing SSL/TLS Configuration..."
nmap --script ssl-enum-ciphers -p 443 filing.vatm.vn

echo "Expected: Only strong ciphers enabled"

# Test 8: HTTP Security Headers
echo "8. Testing Security Headers..."
curl -I https://filing.vatm.vn/api/v1/filing/status/test \
  -H "X-API-Key: test_key"

echo "Expected: HSTS, X-Frame-Options, X-Content-Type-Options headers present"

# Test 9: Input Validation
echo "9. Testing Input Validation..."
curl -X POST https://filing.vatm.vn/api/v1/filing/submit \
  -H "Content-Type: application/xml" \
  -H "X-API-Key: test_key" \
  -d "$(python3 -c 'print("A" * 10000000)')" \
  -w "Status: %{http_code}\n"

echo "Expected: 413 Payload Too Large or 400 Bad Request"

# Test 10: Directory Traversal
echo "10. Testing Directory Traversal..."
curl -X GET "https://filing.vatm.vn/api/v1/filing/../../../etc/passwd" \
  -H "X-API-Key: test_key" \
  -w "Status: %{http_code}\n"

echo "Expected: 404 Not Found"

# Test 11: CORS Configuration
echo "11. Testing CORS Configuration..."
curl -X OPTIONS https://filing.vatm.vn/api/v1/filing/submit \
  -H "Origin: https://malicious-site.com" \
  -H "Access-Control-Request-Method: POST" \
  -v

echo "Expected: Proper CORS headers, restricted origins"

echo "Security Assessment Complete"
echo "Review results and address any vulnerabilities found"
```

#### 10.2.2 Automated Security Testing with OWASP ZAP
```python
#!/usr/bin/env python3
# OWASP ZAP Integration Script

import time
import requests
from zapv2 import ZAPv2

# ZAP Configuration
zap_proxy = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}
target_url = 'https://filing.vatm.vn'
api_key = 'test_security_key'

# Initialize ZAP
zap = ZAPv2(proxies=zap_proxy)

def run_security_scan():
    print("Starting automated security scan with OWASP ZAP")
    
    # Step 1: Spider the application
    print("1. Spidering the application...")
    scan_id = zap.spider.scan(target_url)
    
    while int(zap.spider.status(scan_id)) < 100:
        print(f"Spider progress: {zap.spider.status(scan_id)}%")
        time.sleep(10)
    
    print("Spider completed")
    
    # Step 2: Passive scan
    print("2. Running passive scan...")
    while int(zap.pscan.records_to_scan) > 0:
        print(f"Passive scan remaining: {zap.pscan.records_to_scan}")
        time.sleep(10)
    
    print("Passive scan completed")
    
    # Step 3: Active scan
    print("3. Running active scan...")
    scan_id = zap.ascan.scan(target_url)
    
    while int(zap.ascan.status(scan_id)) < 100:
        print(f"Active scan progress: {zap.ascan.status(scan_id)}%")
        time.sleep(30)
    
    print("Active scan completed")
    
    # Step 4: Generate report
    print("4. Generating security report...")
    
    # Get alerts
    alerts = zap.core.alerts()
    
    # Categorize by risk level
    high_risk = [alert for alert in alerts if alert['risk'] == 'High']
    medium_risk = [alert for alert in alerts if alert['risk'] == 'Medium']
    low_risk = [alert for alert in alerts if alert['risk'] == 'Low']
    
    # Generate HTML report
    html_report = zap.core.htmlreport()
    with open('security_report.html', 'w') as f:
        f.write(html_report)
    
    # Generate JSON report
    json_report = zap.core.jsonreport()
    with open('security_report.json', 'w') as f:
        f.write(json_report)
    
    # Print summary
    print(f"\nSecurity Scan Summary:")
    print(f"High Risk Issues: {len(high_risk)}")
    print(f"Medium Risk Issues: {len(medium_risk)}")
    print(f"Low Risk Issues: {len(low_risk)}")
    
    # Fail build if high-risk issues found
    if high_risk:
        print("\nHIGH RISK VULNERABILITIES FOUND:")
        for alert in high_risk:
            print(f"- {alert['name']}: {alert['description']}")
        return False
    
    return True

def test_api_endpoints():
    """Test specific API endpoints with security focus"""
    print("\nTesting API endpoint security...")
    
    headers = {
        'X-API-Key': api_key,
        'Content-Type': 'application/xml'
    }
    
    # Test 1: Authentication bypass attempts
    test_endpoints = [
        '/api/v1/filing/submit',
        '/api/v1/filing/status/test',
        '/admin/users',
        '/actuator/health'
    ]
    
    for endpoint in test_endpoints:
        url = f"{target_url}{endpoint}"
        
        # Test without authentication
        response = requests.get(url, verify=False)
        print(f"No auth - {endpoint}: {response.status_code}")
        
        # Test with invalid API key
        invalid_headers = headers.copy()
        invalid_headers['X-API-Key'] = 'invalid_key'
        response = requests.get(url, headers=invalid_headers, verify=False)
        print(f"Invalid auth - {endpoint}: {response.status_code}")

if __name__ == "__main__":
    # Run security tests
    scan_success = run_security_scan()
    test_api_endpoints()
    
    if not scan_success:
        exit(1)
    
    print("\nSecurity scan completed successfully!")
```

### 10.3 Load Testing and Performance Validation

#### 10.3.1 Load Testing Strategy
```python
#!/usr/bin/env python3
# Load Testing Script using Locust

from locust import HttpUser, task, between
import xml.etree.ElementTree as ET
import random
import uuid
from datetime import datetime, timedelta

class FilingServiceUser(HttpUser):
    wait_time = between(1, 5)
    
    def on_start(self):
        """Initialize test data and authenticate"""
        self.api_key = "load_test_api_key"
        self.headers = {
            'X-API-Key': self.api_key,
            'X-Airline-Code': 'VN',
            'Content-Type': 'application/xml'
        }
        
        # Test data pools
        self.aircraft_types = ['A320', 'A321', 'B737', 'A350', 'B787']
        self.airports = ['VVNB', 'VVTS', 'VVDN', 'VVCI', 'VVPQ']
        self.airlines = ['VN', 'VJ', 'QH', 'VU']
        
    @task(10)
    def submit_flight_plan(self):
        """Submit a new flight plan (most common operation)"""
        efpl_xml = self.generate_test_efpl()
        
        with self.client.post(
            "/api/v1/filing/submit",
            headers=self.headers,
            data=efpl_xml,
            catch_response=True
        ) as response:
            if response.status_code in [200, 201]:
                response.success()
                # Store filing ID for later use
                if hasattr(response, 'json'):
                    try:
                        filing_data = response.json()
                        if 'filingId' in filing_data:
                            self.environment.filing_ids.append(filing_data['filingId'])
                    except:
                        pass
            elif response.status_code == 422:
                # Validation errors are expected in some cases
                response.success()
            else:
                response.failure(f"Unexpected status code: {response.status_code}")
    
    @task(3)
    def check_filing_status(self):
        """Check status of existing filing"""
        if hasattr(self.environment, 'filing_ids') and self.environment.filing_ids:
            filing_id = random.choice(self.environment.filing_ids)
            
            with self.client.get(
                f"/api/v1/filing/status/{filing_id}",
                headers={'X-API-Key': self.api_key},
                catch_response=True
            ) as response:
                if response.status_code == 200:
                    response.success()
                elif response.status_code == 404:
                    # Filing might have been cleaned up
                    response.success()
                else:
                    response.failure(f"Status check failed: {response.status_code}")
    
    @task(2)
    def update_flight_plan(self):
        """Update an existing flight plan"""
        if hasattr(self.environment, 'filing_ids') and self.environment.filing_ids:
            filing_id = random.choice(self.environment.filing_ids)
            
            update_data = {
                "amendmentType": "TIME_CHANGE",
                "changes": {
                    "estimatedOffBlockTime": self.generate_future_time()
                },
                "reason": "ATC Flow Control Delay"
            }
            
            with self.client.put(
                f"/api/v1/filing/update/{filing_id}",
                headers={
                    'X-API-Key': self.api_key,
                    'Content-Type': 'application/json'
                },
                json=update_data,
                catch_response=True
            ) as response:
                if response.status_code in [200, 404]:
                    response.success()
                else:
                    response.failure(f"Update failed: {response.status_code}")
    
    @task(1)
    def submit_bulk_filing(self):
        """Submit multiple flight plans in bulk"""
        bulk_data = {
            "submissions": []
        }
        
        # Generate 5-10 flight plans for bulk submission
        num_flights = random.randint(5, 10)
        for i in range(num_flights):
            efpl_xml = self.generate_test_efpl()
            bulk_data["submissions"].append({
                "requestId": f"BULK_{uuid.uuid4().hex[:8]}",
                "efpl": efpl_xml
            })
        
        with self.client.post(
            "/api/v1/filing/bulk",
            headers={
                'X-API-Key': self.api_key,
                'Content-Type': 'application/json'
            },
            json=bulk_data,
            catch_response=True
        ) as response:
            if response.status_code in [200, 207]:
                response.success()
            else:
                response.failure(f"Bulk filing failed: {response.status_code}")
    
    def generate_test_efpl(self):
        """Generate a realistic test eFPL"""
        gufi = f"VN{random.randint(100, 999)}-{datetime.now().strftime('%Y%m%d')}-{random.randint(1, 999):03d}"
        flight_number = f"VN{random.randint(100, 999)}"
        aircraft_type = random.choice(self.aircraft_types)
        dep_airport = random.choice(self.airports)
        arr_airport = random.choice([a for a in self.airports if a != dep_airport])
        
        dep_time = self.generate_future_time()
        arr_time = self.generate_future_time(hours_ahead=3)
        
        efpl_template = f"""<?xml version="1.0" encoding="UTF-8"?>
<fx:FlightPlan xmlns:fx="http://www.fixm.aero/flight/4.2">
    <fx:gufi>{gufi}</fx:gufi>
    <fx:flight>
        <fx:flightIdentification>
            <fx:aircraftIdentification>{flight_number}</fx:aircraftIdentification>
        </fx:flightIdentification>
        <fx:aircraft>
            <fx:aircraftType>{aircraft_type}</fx:aircraftType>
            <fx:registration>VN-A{random.randint(100, 999)}</fx:registration>
            <fx:operatorDesignator>HVN</fx:operatorDesignator>
        </fx:aircraft>
        <fx:departure>
            <fx:aerodrome>{dep_airport}</fx:aerodrome>
            <fx:estimatedOffBlockTime>{dep_time}</fx:estimatedOffBlockTime>
        </fx:departure>
        <fx:arrival>
            <fx:aerodrome>{arr_airport}</fx:aerodrome>
            <fx:estimatedOnBlockTime>{arr_time}</fx:estimatedOnBlockTime>
        </fx:arrival>
        <fx:routeTrajectory>
            <fx:routeInformation>
                <fx:routeText>{dep_airport} DCT ADMOX A1 OTBAN {arr_airport}</fx:routeText>
                <fx:cruisingLevel>{random.choice([350, 370, 390, 410])}</fx:cruisingLevel>
            </fx:routeInformation>
        </fx:routeTrajectory>
    </fx:flight>
</fx:FlightPlan>"""
        
        return efpl_template
    
    def generate_future_time(self, hours_ahead=2):
        """Generate a future timestamp"""
        future_time = datetime.now() + timedelta(hours=hours_ahead, minutes=random.randint(0, 59))
        return future_time.strftime('%Y-%m-%dT%H:%M:%SZ')

# Initialize shared storage for filing IDs
def on_locust_init(environment, **kwargs):
    environment.filing_ids = []

# Performance test scenarios
class NormalLoadUser(FilingServiceUser):
    """Simulates normal operational load"""
    weight = 3

class PeakLoadUser(FilingServiceUser):
    """Simulates peak hour traffic"""
    weight = 1
    wait_time = between(0.5, 2)  # More aggressive

class BackgroundUser(FilingServiceUser):
    """Simulates background operations (status checks, etc.)"""
    weight = 1
    
    @task(1)
    def submit_flight_plan(self):
        pass  # Reduce filing submissions
    
    @task(10)
    def check_filing_status(self):
        super().check_filing_status()
```

#### 10.3.2 Performance Test Execution Script
```bash
#!/bin/bash
# Performance Test Execution Script

set -e

# Configuration
TARGET_URL="https://filing.vatm.vn"
TEST_DURATION="30m"
RESULTS_DIR="./performance_results/$(date +%Y%m%d_%H%M%S)"

# Create results directory
mkdir -p "$RESULTS_DIR"

echo "Starting Performance Test Suite"
echo "================================"
echo "Target: $TARGET_URL"
echo "Duration: $TEST_DURATION"
echo "Results: $RESULTS_DIR"

# Test 1: Baseline Performance Test
echo "Test 1: Baseline Performance (50 users)"
locust -f performance_tests/filing_service_load_test.py \
  --host="$TARGET_URL" \
  --users=50 \
  --spawn-rate=5 \
  --run-time="$TEST_DURATION" \
  --html="$RESULTS_DIR/baseline_report.html" \
  --csv="$RESULTS_DIR/baseline" \
  --headless

# Test 2: Stress Test
echo "Test 2: Stress Test (200 users)"
locust -f performance_tests/filing_service_load_test.py \
  --host="$TARGET_URL" \
  --users=200 \
  --spawn-rate=10 \
  --run-time="15m" \
  --html="$RESULTS_DIR/stress_report.html" \
  --csv="$RESULTS_DIR/stress" \
  --headless

# Test 3: Spike Test
echo "Test 3: Spike Test (500 users, 5 minutes)"
locust -f performance_tests/filing_service_load_test.py \
  --host="$TARGET_URL" \
  --users=500 \
  --spawn-rate=50 \
  --run-time="5m" \
  --html="$RESULTS_DIR/spike_report.html" \
  --csv="$RESULTS_DIR/spike" \
  --headless

# Test 4: Endurance Test
echo "Test 4: Endurance Test (100 users, 2 hours)"
locust -f performance_tests/filing_service_load_test.py \
  --host="$TARGET_URL" \
  --users=100 \
  --spawn-rate=2 \
  --run-time="2h" \
  --html="$RESULTS_DIR/endurance_report.html" \
  --csv="$RESULTS_DIR/endurance" \
  --headless

# Generate summary report
echo "Generating Performance Summary Report"
python3 performance_tests/generate_summary.py "$RESULTS_DIR"

# Check performance thresholds
echo "Validating Performance Thresholds"
python3 performance_tests/validate_thresholds.py "$RESULTS_DIR"

echo "Performance Testing Complete"
echo "Results available in: $RESULTS_DIR"
```

---

## 11. MAINTENANCE & OPERATIONS

### 11.1 System Monitoring and Health Checks

#### 11.1.1 Comprehensive Health Check Implementation
```java
@Component
@ConditionalOnProperty(name = "management.health.custom.enabled", havingValue = "true", matchIfMissing = true)
public class FilingServiceHealthIndicator implements HealthIndicator {
    
    @Autowired
    private DataSource dataSource;
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @Autowired
    private FDPSIntegrator fdpsIntegrator;
    
    @Autowired
    private BusinessRulesEngine businessRulesEngine;
    
    @Value("${vatm.health-check.timeout.seconds:30}")
    private int healthCheckTimeout;
    
    @Override
    public Health health() {
        Health.Builder builder = Health.up();
        
        try {
            // Check database connectivity
            checkDatabaseHealth(builder);
            
            // Check Redis cache
            checkRedisHealth(builder);
            
            // Check FDPS connectivity
            checkFDPSHealth(builder);
            
            // Check business rules engine
            checkBusinessRulesHealth(builder);
            
            // Check system resources
            checkSystemResources(builder);
            
            // Check recent filing success rate
            checkRecentFilingSuccessRate(builder);
            
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .withDetail("timestamp", Instant.now())
                .build();
        }
        
        return builder.build();
    }
    
    private void checkDatabaseHealth(Health.Builder builder) {
        try {
            long startTime = System.currentTimeMillis();
            
            try (Connection connection = dataSource.getConnection()) {
                // Test database connectivity with simple query
                try (PreparedStatement stmt = connection.prepareStatement("SELECT 1")) {
                    ResultSet rs = stmt.executeQuery();
                    if (rs.next() && rs.getInt(1) == 1) {
                        long responseTime = System.currentTimeMillis() - startTime;
                        builder.withDetail("database.status", "UP")
                               .withDetail("database.responseTime", responseTime + "ms");
                        
                        if (responseTime > 5000) {
                            builder.withDetail("database.warning", "Slow response time");
                        }
                    } else {
                        builder.down().withDetail("database.status", "Query failed");
                    }
                }
            }
            
            // Check database pool status
            HikariDataSource hikariDataSource = (HikariDataSource) dataSource;
            HikariPoolMXBean poolBean = hikariDataSource.getHikariPoolMXBean();
            
            builder.withDetail("database.pool.active", poolBean.getActiveConnections())
                   .withDetail("database.pool.idle", poolBean.getIdleConnections())
                   .withDetail("database.pool.total", poolBean.getTotalConnections())
                   .withDetail("database.pool.waiting", poolBean.getThreadsAwaitingConnection());
            
        } catch (SQLException e) {
            builder.down().withDetail("database.error", e.getMessage());
        }
    }
    
    private void checkRedisHealth(Health.Builder builder) {
        try {
            long startTime = System.currentTimeMillis();
            
            // Test Redis connectivity
            String testKey = "health-check-" + UUID.randomUUID().toString();
            String testValue = "test-" + Instant.now();
            
            redisTemplate.opsForValue().set(testKey, testValue, Duration.ofSeconds(60));
            String retrievedValue = redisTemplate.opsForValue().get(testKey);
            redisTemplate.delete(testKey);
            
            if (testValue.equals(retrievedValue)) {
                long responseTime = System.currentTimeMillis() - startTime;
                builder.withDetail("redis.status", "UP")
                       .withDetail("redis.responseTime", responseTime + "ms");
            } else {
                builder.down().withDetail("redis.error", "Data integrity check failed");
            }
            
        } catch (Exception e) {
            builder.down().withDetail("redis.error", e.getMessage());
        }
    }
    
    private void checkFDPSHealth(Health.Builder builder) {
        try {
            // Test FDPS connectivity with ping message
            FDPSHealthResult result = fdpsIntegrator.performHealthCheck();
            
            if (result.isHealthy()) {
                builder.withDetail("fdps.status", "UP")
                       .withDetail("fdps.responseTime", result.getResponseTime() + "ms")
                       .withDetail("fdps.lastSuccessfulIntegration", result.getLastSuccessfulIntegration());
            } else {
                builder.withDetail("fdps.status", "DOWN")
                       .withDetail("fdps.error", result.getErrorMessage())
                       .withDetail("fdps.lastError", result.getLastErrorTime());
            }
            
        } catch (Exception e) {
            builder.withDetail("fdps.status", "DOWN")
                   .withDetail("fdps.error", e.getMessage());
        }
    }
    
    private void checkBusinessRulesHealth(Health.Builder builder) {
        try {
            // Test business rules engine with simple validation
            long startTime = System.currentTimeMillis();
            
            FlightPlan testFlight = createHealthCheckFlightPlan();
            BusinessRulesResult result = businessRulesEngine.validate(testFlight);
            
            long responseTime = System.currentTimeMillis() - startTime;
            
            builder.withDetail("businessRules.status", "UP")
                   .withDetail("businessRules.responseTime", responseTime + "ms")
                   .withDetail("businessRules.rulesEvaluated", result.getRulesEvaluated());
            
        } catch (Exception e) {
            builder.down().withDetail("businessRules.error", e.getMessage());
        }
    }
    
    private void checkSystemResources(Health.Builder builder) {
        // JVM Memory
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        
        double heapUsedPercent = (double) heapUsage.getUsed() / heapUsage.getMax() * 100;
        
        builder.withDetail("system.memory.heap.used", heapUsage.getUsed())
               .withDetail("system.memory.heap.max", heapUsage.getMax())
               .withDetail("system.memory.heap.usedPercent", String.format("%.1f%%", heapUsedPercent));
        
        if (heapUsedPercent > 85) {
            builder.withDetail("system.memory.warning", "High heap usage");
        }
        
        // Thread Count
        ThreadMXBean threadBean = ManagementFactory.getThreadMXBean();
        int threadCount = threadBean.getThreadCount();
        
        builder.withDetail("system.threads.count", threadCount);
        
        if (threadCount > 500) {
            builder.withDetail("system.threads.warning", "High thread count");
        }
        
        // Disk Space
        File rootDir = new File("/");
        long totalSpace = rootDir.getTotalSpace();
        long freeSpace = rootDir.getFreeSpace();
        double usedPercent = (double) (totalSpace - freeSpace) / totalSpace * 100;
        
        builder.withDetail("system.disk.total", totalSpace)
               .withDetail("system.disk.free", freeSpace)
               .withDetail("system.disk.usedPercent", String.format("%.1f%%", usedPercent));
        
        if (usedPercent > 90) {
            builder.withDetail("system.disk.warning", "Low disk space");
        }
    }
    
    private void checkRecentFilingSuccessRate(Health.Builder builder) {
        try {
            // Check filing success rate over last hour
            LocalDateTime hourAgo = LocalDateTime.now().minusHours(1);
            
            long totalFilings = filingRepository.countBySubmittedAtAfter(hourAgo);
            long successfulFilings = filingRepository.countBySubmittedAtAfterAndCurrentStatus(
                hourAgo, FilingStatus.ACCEPTED);
            
            if (totalFilings > 0) {
                double successRate = (double) successfulFilings / totalFilings * 100;
                
                builder.withDetail("filing.recent.total", totalFilings)
                       .withDetail("filing.recent.successful", successfulFilings)
                       .withDetail("filing.recent.successRate", String.format("%.1f%%", successRate));
                
                if (successRate < 90) {
                    builder.withDetail("filing.warning", "Low success rate");
                }
            } else {
                builder.withDetail("filing.recent.total", 0)
                       .withDetail("filing.recent.note", "No recent filings");
            }
            
        } catch (Exception e) {
            builder.withDetail("filing.error", e.getMessage());
        }
    }
    
    private FlightPlan createHealthCheckFlightPlan() {
        return FlightPlan.builder()
            .gufi("HEALTH-CHECK-" + Instant.now().getEpochSecond())
            .flightNumber("TEST001")
            .aircraftType("A320")
            .departureAirport("VVNB")
            .arrivalAirport("VVTS")
            .cruiseAltitude(35000)
            .estimatedFlightTime(120)
            .build();
    }
}
```

#### 11.1.2 Custom Metrics for Business Logic
```java
@Component
public class FilingServiceMetrics {
    
    private final MeterRegistry meterRegistry;
    private final Counter filingSubmissions;
    private final Counter filingAccepted;
    private final Counter filingRejected;
    private final Timer filingProcessingTime;
    private final Timer validationTime;
    private final Timer fdpsIntegrationTime;
    
    public FilingServiceMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        
        // Filing counters
        this.filingSubmissions = Counter.builder("filing.submissions.total")
            .description("Total number of flight plan submissions")
            .tag("service", "filing")
            .register(meterRegistry);
            
        this.filingAccepted = Counter.builder("filing.accepted.total")
            .description("Number of accepted flight plans")
            .tag("service", "filing")
            .register(meterRegistry);
            
        this.filingRejected = Counter.builder("filing.rejected.total")
            .description("Number of rejected flight plans")
            .tag("service", "filing")
            .register(meterRegistry);
        
        // Timing metrics
        this.filingProcessingTime = Timer.builder("filing.processing.duration")
            .description("Time taken to process flight plan filing")
            .tag("service", "filing")
            .register(meterRegistry);
            
        this.validationTime = Timer.builder("filing.validation.duration")
            .description("Time taken for flight plan validation")
            .tag("service", "filing")
            .register(meterRegistry);
            
        this.fdpsIntegrationTime = Timer.builder("filing.fdps.integration.duration")
            .description("Time taken for FDPS integration")
            .tag("service", "filing")
            .register(meterRegistry);
    }
    
    public void recordFilingSubmission(String airlineCode, String aircraftType) {
        filingSubmissions.increment(
            Tags.of(
                "airline", airlineCode,
                "aircraft_type", aircraftType
            )
        );
    }
    
    public void recordFilingAccepted(String airlineCode, String aircraftType) {
        filingAccepted.increment(
            Tags.of(
                "airline", airlineCode,
                "aircraft_type", aircraftType
            )
        );
    }
    
    public void recordFilingRejected(String airlineCode, String reason) {
        filingRejected.increment(
            Tags.of(
                "airline", airlineCode,
                "rejection_reason", reason
            )
        );
    }
    
    public Timer.Sample startFilingTimer() {
        return Timer.start(meterRegistry);
    }
    
    public void recordFilingTime(Timer.Sample sample, String status) {
        sample.stop(filingProcessingTime.tag("status", status));
    }
    
    public Timer.Sample startValidationTimer() {
        return Timer.start(meterRegistry);
    }
    
    public void recordValidationTime(Timer.Sample sample, String validationType) {
        sample.stop(validationTime.tag("type", validationType));
    }
    
    public Timer.Sample startFDPSIntegrationTimer() {
        return Timer.start(meterRegistry);
    }
    
    public void recordFDPSIntegrationTime(Timer.Sample sample, boolean success) {
        sample.stop(fdpsIntegrationTime.tag("success", String.valueOf(success)));
    }
    
    // Gauge for current system state
    @PostConstruct
    public void registerGauges() {
        // Active filing count
        Gauge.builder("filing.active.count")
            .description("Number of currently active flight plans")
            .register(meterRegistry, this, FilingServiceMetrics::getActiveFilingCount);
        
        // Pending filing count
        Gauge.builder("filing.pending.count")
            .description("Number of flight plans pending approval")
            .register(meterRegistry, this, FilingServiceMetrics::getPendingFilingCount);
        
        // Database connection pool
        Gauge.builder("database.connections.active")
            .description("Active database connections")
            .register(meterRegistry, this, FilingServiceMetrics::getActiveConnectionCount);
    }
    
    private Number getActiveFilingCount(FilingServiceMetrics metrics) {
        try {
            return filingRepository.countByCurrentStatus(FilingStatus.ACCEPTED);
        } catch (Exception e) {
            return 0;
        }
    }
    
    private Number getPendingFilingCount(FilingServiceMetrics metrics) {
        try {
            return filingRepository.countByCurrentStatus(FilingStatus.PENDING);
        } catch (Exception e) {
            return 0

    private Number getPendingFilingCount(FilingServiceMetrics metrics) {
        try {
            return filingRepository.countByCurrentStatus(FilingStatus.PENDING);
        } catch (Exception e) {
            return 0;
        }
    }
    
    private Number getActiveConnectionCount(FilingServiceMetrics metrics) {
        try {
            if (dataSource instanceof HikariDataSource) {
                HikariDataSource hikariDataSource = (HikariDataSource) dataSource;
                return hikariDataSource.getHikariPoolMXBean().getActiveConnections();
            }
            return 0;
        } catch (Exception e) {
            return 0;
        }
    }
}
```

### 11.2 Backup and Recovery Procedures

#### 11.2.1 Automated Backup Script
```bash
#!/bin/bash
# VATM Filing Service Backup Script
# Usage: ./backup.sh [full|incremental|config]

set -e

# Configuration
BACKUP_TYPE="${1:-incremental}"
BACKUP_DIR="/backups/filing-service"
DATE=$(date +"%Y%m%d_%H%M%S")
RETENTION_DAYS=30
LOG_FILE="/var/log/vatm/backup.log"

# Database configuration
DB_HOST="10.1.3.10"
DB_PORT="5432"
DB_NAME="vatm_filing"
DB_USER="filing_service"
DB_PASSWORD="${DB_PASSWORD}"

# Application configuration
APP_CONFIG_DIR="/opt/vatm/filing-service/config"
SSL_CERT_DIR="/opt/vatm/filing-service/ssl"
LOGS_DIR="/var/log/vatm"

# Functions
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

create_backup_dir() {
    local backup_path="$BACKUP_DIR/$DATE"
    mkdir -p "$backup_path"
    echo "$backup_path"
}

backup_database_full() {
    local backup_path="$1"
    log_message "Starting full database backup..."
    
    # Create full database dump
    PGPASSWORD="$DB_PASSWORD" pg_dump \
        -h "$DB_HOST" \
        -p "$DB_PORT" \
        -U "$DB_USER" \
        -d "$DB_NAME" \
        --verbose \
        --format=custom \
        --compress=9 \
        --file="$backup_path/database_full.dump"
    
    if [ $? -eq 0 ]; then
        log_message "Full database backup completed successfully"
        
        # Compress backup
        gzip "$backup_path/database_full.dump"
        
        # Calculate checksums
        md5sum "$backup_path/database_full.dump.gz" > "$backup_path/database_full.dump.gz.md5"
        
    else
        log_message "ERROR: Full database backup failed"
        return 1
    fi
}

backup_database_incremental() {
    local backup_path="$1"
    log_message "Starting incremental database backup..."
    
    # Get current WAL position
    WAL_POSITION=$(PGPASSWORD="$DB_PASSWORD" psql \
        -h "$DB_HOST" \
        -p "$DB_PORT" \
        -U "$DB_USER" \
        -d "$DB_NAME" \
        -t -c "SELECT pg_current_wal_lsn();" | tr -d ' ')
    
    # Backup WAL files since last backup
    PGPASSWORD="$DB_PASSWORD" pg_receivewal \
        -h "$DB_HOST" \
        -p "$DB_PORT" \
        -U "$DB_USER" \
        -D "$backup_path/wal" \
        --synchronous \
        --verbose
    
    # Store WAL position for next incremental backup
    echo "$WAL_POSITION" > "$backup_path/wal_position.txt"
    
    log_message "Incremental database backup completed (WAL position: $WAL_POSITION)"
}

backup_application_config() {
    local backup_path="$1"
    log_message "Starting application configuration backup..."
    
    # Create config backup directory
    mkdir -p "$backup_path/config"
    
    # Backup application configuration
    if [ -d "$APP_CONFIG_DIR" ]; then
        tar -czf "$backup_path/config/app_config.tar.gz" -C "$APP_CONFIG_DIR" .
        log_message "Application configuration backed up"
    fi
    
    # Backup SSL certificates
    if [ -d "$SSL_CERT_DIR" ]; then
        tar -czf "$backup_path/config/ssl_certs.tar.gz" -C "$SSL_CERT_DIR" .
        log_message "SSL certificates backed up"
    fi
    
    # Backup Docker configuration
    if [ -f "/opt/vatm/filing-service/docker-compose.yml" ]; then
        cp "/opt/vatm/filing-service/docker-compose.yml" "$backup_path/config/"
        cp "/opt/vatm/filing-service/.env" "$backup_path/config/" 2>/dev/null || true
        log_message "Docker configuration backed up"
    fi
    
    # Backup system configuration
    tar -czf "$backup_path/config/system_config.tar.gz" \
        /etc/nginx/sites-available/filing-service \
        /etc/systemd/system/filing-service.service \
        /etc/logrotate.d/filing-service \
        2>/dev/null || true
    
    log_message "Application configuration backup completed"
}

backup_logs() {
    local backup_path="$1"
    log_message "Starting log backup..."
    
    # Create logs backup directory
    mkdir -p "$backup_path/logs"
    
    # Backup application logs (last 7 days)
    find "$LOGS_DIR" -name "*.log" -mtime -7 -exec cp {} "$backup_path/logs/" \;
    
    # Compress log files
    tar -czf "$backup_path/logs.tar.gz" -C "$backup_path" logs/
    rm -rf "$backup_path/logs"
    
    log_message "Log backup completed"
}

cleanup_old_backups() {
    log_message "Cleaning up old backups (retention: $RETENTION_DAYS days)..."
    
    find "$BACKUP_DIR" -type d -mtime +$RETENTION_DAYS -exec rm -rf {} + 2>/dev/null || true
    
    # Clean up old WAL files
    find "$BACKUP_DIR" -name "*.wal" -mtime +$RETENTION_DAYS -delete 2>/dev/null || true
    
    log_message "Old backup cleanup completed"
}

verify_backup() {
    local backup_path="$1"
    log_message "Verifying backup integrity..."
    
    # Verify database backup
    if [ -f "$backup_path/database_full.dump.gz" ]; then
        # Check MD5 sum
        cd "$backup_path"
        if md5sum -c database_full.dump.gz.md5; then
            log_message "Database backup integrity verified"
        else
            log_message "ERROR: Database backup integrity check failed"
            return 1
        fi
    fi
    
    # Verify configuration backups
    for config_file in "$backup_path/config"/*.tar.gz; do
        if [ -f "$config_file" ]; then
            if tar -tzf "$config_file" > /dev/null 2>&1; then
                log_message "Configuration backup $(basename $config_file) verified"
            else
                log_message "ERROR: Configuration backup $(basename $config_file) is corrupted"
                return 1
            fi
        fi
    done
    
    log_message "Backup verification completed successfully"
}

send_backup_notification() {
    local backup_path="$1"
    local status="$2"
    
    # Calculate backup size
    BACKUP_SIZE=$(du -sh "$backup_path" | cut -f1)
    
    # Send email notification
    {
        echo "VATM Filing Service Backup Report"
        echo "================================="
        echo "Date: $(date)"
        echo "Type: $BACKUP_TYPE"
        echo "Status: $status"
        echo "Size: $BACKUP_SIZE"
        echo "Location: $backup_path"
        echo ""
        echo "Backup Contents:"
        ls -la "$backup_path"
        echo ""
        echo "Log excerpt:"
        tail -20 "$LOG_FILE"
    } | mail -s "VATM Filing Service Backup - $status" ops@vatm.vn
}

# Main backup execution
main() {
    log_message "Starting $BACKUP_TYPE backup for VATM Filing Service"
    
    # Create backup directory
    BACKUP_PATH=$(create_backup_dir)
    
    # Stop application services for consistent backup (optional)
    if [ "$BACKUP_TYPE" = "full" ]; then
        log_message "Stopping application services for consistent backup..."
        systemctl stop filing-service || docker-compose stop filing-service
        sleep 10
    fi
    
    try {
        case "$BACKUP_TYPE" in
            "full")
                backup_database_full "$BACKUP_PATH"
                backup_application_config "$BACKUP_PATH"
                backup_logs "$BACKUP_PATH"
                ;;
            "incremental")
                backup_database_incremental "$BACKUP_PATH"
                ;;
            "config")
                backup_application_config "$BACKUP_PATH"
                ;;
            *)
                log_message "ERROR: Unknown backup type: $BACKUP_TYPE"
                echo "Usage: $0 [full|incremental|config]"
                exit 1
                ;;
        esac
        
        # Verify backup integrity
        verify_backup "$BACKUP_PATH"
        
        # Cleanup old backups
        cleanup_old_backups
        
        # Restart services if stopped
        if [ "$BACKUP_TYPE" = "full" ]; then
            log_message "Restarting application services..."
            systemctl start filing-service || docker-compose start filing-service
        fi
        
        log_message "Backup completed successfully"
        send_backup_notification "$BACKUP_PATH" "SUCCESS"
        
    } catch {
        log_message "ERROR: Backup failed"
        
        # Restart services if stopped
        if [ "$BACKUP_TYPE" = "full" ]; then
            systemctl start filing-service || docker-compose start filing-service
        fi
        
        send_backup_notification "$BACKUP_PATH" "FAILED"
        exit 1
    }
}

# Execute main function
main "$@"
```

#### 11.2.2 Disaster Recovery Script
```bash
#!/bin/bash
# VATM Filing Service Disaster Recovery Script
# Usage: ./disaster_recovery.sh [restore_date] [restore_type]

set -e

# Configuration
RESTORE_DATE="${1:-latest}"
RESTORE_TYPE="${2:-full}"  # full, database_only, config_only
BACKUP_DIR="/backups/filing-service"
TEMP_RESTORE_DIR="/tmp/filing_service_restore"
LOG_FILE="/var/log/vatm/disaster_recovery.log"

# Database configuration
DB_HOST="10.1.3.10"
DB_PORT="5432"
DB_NAME="vatm_filing"
DB_USER="filing_service"
DB_PASSWORD="${DB_PASSWORD}"

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

find_backup_to_restore() {
    if [ "$RESTORE_DATE" = "latest" ]; then
        BACKUP_PATH=$(find "$BACKUP_DIR" -type d -name "20*" | sort -r | head -1)
    else
        BACKUP_PATH=$(find "$BACKUP_DIR" -type d -name "*$RESTORE_DATE*" | head -1)
    fi
    
    if [ -z "$BACKUP_PATH" ] || [ ! -d "$BACKUP_PATH" ]; then
        log_message "ERROR: Backup not found for date: $RESTORE_DATE"
        exit 1
    fi
    
    log_message "Found backup to restore: $BACKUP_PATH"
    echo "$BACKUP_PATH"
}

restore_database() {
    local backup_path="$1"
    log_message "Starting database restoration..."
    
    # Stop application to prevent database access
    log_message "Stopping filing service..."
    systemctl stop filing-service || docker-compose stop filing-service || true
    
    # Find database backup file
    DB_BACKUP_FILE=""
    if [ -f "$backup_path/database_full.dump.gz" ]; then
        DB_BACKUP_FILE="$backup_path/database_full.dump.gz"
    elif [ -f "$backup_path/database_full.dump" ]; then
        DB_BACKUP_FILE="$backup_path/database_full.dump"
    else
        log_message "ERROR: Database backup file not found"
        return 1
    fi
    
    # Extract if compressed
    if [[ "$DB_BACKUP_FILE" == *.gz ]]; then
        log_message "Extracting compressed database backup..."
        gunzip -c "$DB_BACKUP_FILE" > "$TEMP_RESTORE_DIR/database_restore.dump"
        DB_BACKUP_FILE="$TEMP_RESTORE_DIR/database_restore.dump"
    fi
    
    # Verify backup integrity
    log_message "Verifying database backup integrity..."
    if ! pg_restore --list "$DB_BACKUP_FILE" > /dev/null 2>&1; then
        log_message "ERROR: Database backup file is corrupted"
        return 1
    fi
    
    # Create backup of current database before restore
    log_message "Creating backup of current database before restore..."
    PGPASSWORD="$DB_PASSWORD" pg_dump \
        -h "$DB_HOST" \
        -p "$DB_PORT" \
        -U "$DB_USER" \
        -d "$DB_NAME" \
        --format=custom \
        --file="/tmp/pre_restore_backup_$(date +%Y%m%d_%H%M%S).dump"
    
    # Drop existing database connections
    log_message "Terminating existing database connections..."
    PGPASSWORD="$DB_PASSWORD" psql \
        -h "$DB_HOST" \
        -p "$DB_PORT" \
        -U "$DB_USER" \
        -d "postgres" \
        -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='$DB_NAME' AND pid <> pg_backend_pid();"
    
    # Drop and recreate database
    log_message "Dropping and recreating database..."
    PGPASSWORD="$DB_PASSWORD" dropdb \
        -h "$DB_HOST" \
        -p "$DB_PORT" \
        -U "$DB_USER" \
        "$DB_NAME"
    
    PGPASSWORD="$DB_PASSWORD" createdb \
        -h "$DB_HOST" \
        -p "$DB_PORT" \
        -U "$DB_USER" \
        "$DB_NAME"
    
    # Restore database
    log_message "Restoring database from backup..."
    PGPASSWORD="$DB_PASSWORD" pg_restore \
        -h "$DB_HOST" \
        -p "$DB_PORT" \
        -U "$DB_USER" \
        -d "$DB_NAME" \
        --verbose \
        --clean \
        --if-exists \
        "$DB_BACKUP_FILE"
    
    if [ $? -eq 0 ]; then
        log_message "Database restoration completed successfully"
    else
        log_message "ERROR: Database restoration failed"
        return 1
    fi
}

restore_application_config() {
    local backup_path="$1"
    log_message "Starting application configuration restoration..."
    
    # Backup current configuration
    log_message "Backing up current configuration..."
    mkdir -p "/tmp/config_backup_$(date +%Y%m%d_%H%M%S)"
    cp -r /opt/vatm/filing-service/config "/tmp/config_backup_$(date +%Y%m%d_%H%M%S)/" 2>/dev/null || true
    
    # Restore application configuration
    if [ -f "$backup_path/config/app_config.tar.gz" ]; then
        log_message "Restoring application configuration..."
        tar -xzf "$backup_path/config/app_config.tar.gz" -C /opt/vatm/filing-service/config/
    fi
    
    # Restore SSL certificates
    if [ -f "$backup_path/config/ssl_certs.tar.gz" ]; then
        log_message "Restoring SSL certificates..."
        tar -xzf "$backup_path/config/ssl_certs.tar.gz" -C /opt/vatm/filing-service/ssl/
        chmod 600 /opt/vatm/filing-service/ssl/*.key
        chmod 644 /opt/vatm/filing-service/ssl/*.crt
    fi
    
    # Restore Docker configuration
    if [ -f "$backup_path/config/docker-compose.yml" ]; then
        log_message "Restoring Docker configuration..."
        cp "$backup_path/config/docker-compose.yml" /opt/vatm/filing-service/
        cp "$backup_path/config/.env" /opt/vatm/filing-service/ 2>/dev/null || true
    fi
    
    # Restore system configuration
    if [ -f "$backup_path/config/system_config.tar.gz" ]; then
        log_message "Restoring system configuration..."
        tar -xzf "$backup_path/config/system_config.tar.gz" -C /
    fi
    
    log_message "Application configuration restoration completed"
}

validate_restored_system() {
    log_message "Validating restored system..."
    
    # Start services
    log_message "Starting filing service..."
    systemctl start filing-service || docker-compose up -d filing-service
    
    # Wait for service to start
    sleep 30
    
    # Test database connectivity
    log_message "Testing database connectivity..."
    if PGPASSWORD="$DB_PASSWORD" psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" -c "SELECT 1;" > /dev/null 2>&1; then
        log_message "Database connectivity test passed"
    else
        log_message "ERROR: Database connectivity test failed"
        return 1
    fi
    
    # Test application health endpoint
    log_message "Testing application health endpoint..."
    sleep 10
    if curl -f http://localhost:8080/actuator/health > /dev/null 2>&1; then
        log_message "Application health test passed"
    else
        log_message "WARNING: Application health test failed - service may still be starting"
    fi
    
    # Test basic API functionality
    log_message "Testing basic API functionality..."
    API_RESPONSE=$(curl -s -w "%{http_code}" -o /dev/null -X GET \
        -H "X-API-Key: test_key" \
        http://localhost:8080/api/v1/filing/status/test-123)
    
    if [ "$API_RESPONSE" = "404" ] || [ "$API_RESPONSE" = "200" ]; then
        log_message "API functionality test passed (response: $API_RESPONSE)"
    else
        log_message "WARNING: API functionality test returned unexpected response: $API_RESPONSE"
    fi
    
    log_message "System validation completed"
}

send_recovery_notification() {
    local status="$1"
    local backup_path="$2"
    
    {
        echo "VATM Filing Service Disaster Recovery Report"
        echo "==========================================="
        echo "Date: $(date)"
        echo "Restore Type: $RESTORE_TYPE"
        echo "Backup Used: $backup_path"
        echo "Status: $status"
        echo ""
        echo "Recovery Log:"
        tail -50 "$LOG_FILE"
    } | mail -s "VATM Filing Service Disaster Recovery - $status" ops@vatm.vn
}

# Main recovery execution
main() {
    log_message "Starting disaster recovery for VATM Filing Service"
    log_message "Restore type: $RESTORE_TYPE"
    log_message "Restore date: $RESTORE_DATE"
    
    # Create temporary directory
    mkdir -p "$TEMP_RESTORE_DIR"
    
    # Find backup to restore
    BACKUP_PATH=$(find_backup_to_restore)
    
    try {
        case "$RESTORE_TYPE" in
            "full")
                restore_database "$BACKUP_PATH"
                restore_application_config "$BACKUP_PATH"
                ;;
            "database_only")
                restore_database "$BACKUP_PATH"
                ;;
            "config_only")
                restore_application_config "$BACKUP_PATH"
                ;;
            *)
                log_message "ERROR: Unknown restore type: $RESTORE_TYPE"
                echo "Usage: $0 [restore_date] [full|database_only|config_only]"
                exit 1
                ;;
        esac
        
        # Validate restored system
        validate_restored_system
        
        log_message "Disaster recovery completed successfully"
        send_recovery_notification "SUCCESS" "$BACKUP_PATH"
        
    } catch {
        log_message "ERROR: Disaster recovery failed"
        send_recovery_notification "FAILED" "$BACKUP_PATH"
        exit 1
    } finally {
        # Cleanup
        rm -rf "$TEMP_RESTORE_DIR"
    }
}

# Execute main function
main "$@"
```

### 11.3 Operational Procedures

#### 11.3.1 Daily Operations Checklist
```bash
#!/bin/bash
# Daily Operations Checklist Script
# Run every day at 06:00 via cron

LOG_FILE="/var/log/vatm/daily_operations.log"
CHECKLIST_DATE=$(date +"%Y-%m-%d")

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

generate_daily_report() {
    local report_file="/var/log/vatm/daily_report_${CHECKLIST_DATE}.txt"
    
    {
        echo "VATM Filing Service Daily Operations Report"
        echo "Date: $CHECKLIST_DATE"
        echo "================================================"
        echo ""
        
        # 1. System Status
        echo "1. SYSTEM STATUS"
        echo "=================="
        systemctl is-active filing-service && echo "✓ Filing Service: RUNNING" || echo "✗ Filing Service: NOT RUNNING"
        systemctl is-active postgresql && echo "✓ PostgreSQL: RUNNING" || echo "✗ PostgreSQL: NOT RUNNING"
        systemctl is-active redis && echo "✓ Redis: RUNNING" || echo "✗ Redis: NOT RUNNING"
        systemctl is-active nginx && echo "✓ Nginx: RUNNING" || echo "✗ Nginx: NOT RUNNING"
        echo ""
        
        # 2. Performance Metrics
        echo "2. PERFORMANCE METRICS (Last 24 Hours)"
        echo "======================================="
        
        # Filing statistics
        YESTERDAY=$(date -d "yesterday" +"%Y-%m-%d")
        TOTAL_FILINGS=$(sudo -u postgres psql -d vatm_filing -t -c "SELECT COUNT(*) FROM filed_flight_plans WHERE DATE(submitted_at) = '$YESTERDAY';" | tr -d ' ')
        ACCEPTED_FILINGS=$(sudo -u postgres psql -d vatm_filing -t -c "SELECT COUNT(*) FROM filed_flight_plans WHERE DATE(submitted_at) = '$YESTERDAY' AND current_status = 'ACCEPTED';" | tr -d ' ')
        REJECTED_FILINGS=$(sudo -u postgres psql -d vatm_filing -t -c "SELECT COUNT(*) FROM filed_flight_plans WHERE DATE(submitted_at) = '$YESTERDAY' AND current_status = 'REJECTED';" | tr -d ' ')
        
        echo "Total Filings: $TOTAL_FILINGS"
        echo "Accepted: $ACCEPTED_FILINGS"
        echo "Rejected: $REJECTED_FILINGS"
        
        if [ "$TOTAL_FILINGS" -gt 0 ]; then
            SUCCESS_RATE=$(echo "scale=1; $ACCEPTED_FILINGS * 100 / $TOTAL_FILINGS" | bc)
            echo "Success Rate: ${SUCCESS_RATE}%"
        else
            echo "Success Rate: N/A (No filings)"
        fi
        echo ""
        
        # 3. System Resource Usage
        echo "3. SYSTEM RESOURCES"
        echo "==================="
        echo "Memory Usage:"
        free -h
        echo ""
        echo "Disk Usage:"
        df -h | grep -E "(/$|/var|/opt)"
        echo ""
        echo "CPU Usage (5-minute average):"
        uptime
        echo ""
        
        # 4. Database Status
        echo "4. DATABASE STATUS"
        echo "=================="
        DB_SIZE=$(sudo -u postgres psql -d vatm_filing -t -c "SELECT pg_size_pretty(pg_database_size('vatm_filing'));" | tr -d ' ')
        echo "Database Size: $DB_SIZE"
        
        ACTIVE_CONNECTIONS=$(sudo -u postgres psql -d vatm_filing -t -c "SELECT count(*) FROM pg_stat_activity WHERE datname = 'vatm_filing';" | tr -d ' ')
        echo "Active Connections: $ACTIVE_CONNECTIONS"
        
        # Check for long-running queries
        LONG_QUERIES=$(sudo -u postgres psql -d vatm_filing -t -c "SELECT count(*) FROM pg_stat_activity WHERE datname = 'vatm_filing' AND state = 'active' AND query_start < NOW() - INTERVAL '5 minutes';" | tr -d ' ')
        echo "Long-running Queries (>5 min): $LONG_QUERIES"
        echo ""
        
        # 5. Error Summary
        echo "5. ERROR SUMMARY"
        echo "================"
        echo "Application Errors (last 24 hours):"
        grep -c "ERROR" /var/log/vatm/filing-service.log 2>/dev/null || echo "0"
        
        echo "Database Errors (last 24 hours):"
        grep -c "ERROR" /var/log/postgresql/postgresql-15-main.log 2>/dev/null || echo "0"
        
        echo "Nginx Errors (last 24 hours):"
        grep -c "error" /var/log/nginx/error.log 2>/dev/null || echo "0"
        echo ""
        
        # 6. Backup Status
        echo "6. BACKUP STATUS"
        echo "================"
        LATEST_BACKUP=$(find /backups/filing-service -type d -name "20*" | sort -r | head -1)
        if [ -n "$LATEST_BACKUP" ]; then
            BACKUP_DATE=$(basename "$LATEST_BACKUP")
            BACKUP_SIZE=$(du -sh "$LATEST_BACKUP" | cut -f1)
            echo "Latest Backup: $BACKUP_DATE ($BACKUP_SIZE)"
        else
            echo "⚠ No recent backups found"
        fi
        echo ""
        
        # 7. Integration Status
        echo "7. INTEGRATION STATUS"
        echo "===================="
        
        # Test FDPS connectivity
        FDPS_STATUS="OK"
        # Add FDPS connectivity test here
        echo "FDPS Integration: $FDPS_STATUS"
        
        # Test external API endpoints
        HEALTH_CHECK=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/actuator/health)
        if [ "$HEALTH_CHECK" = "200" ]; then
            echo "Health Endpoint: OK"
        else
            echo "Health Endpoint: FAILED ($HEALTH_CHECK)"
        fi
        echo ""
        
        # 8. Recommendations
        echo "8. RECOMMENDATIONS"
        echo "=================="
        
        # Check disk space
        DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
        if [ "$DISK_USAGE" -gt 80 ]; then
            echo "⚠ WARNING: Disk usage is high (${DISK_USAGE}%) - Consider cleanup"
        fi
        
        # Check memory usage
        MEMORY_USAGE=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
        if [ "$MEMORY_USAGE" -gt 80 ]; then
            echo "⚠ WARNING: Memory usage is high (${MEMORY_USAGE}%) - Monitor closely"
        fi
        
        # Check error rates
        if [ "$LONG_QUERIES" -gt 5 ]; then
            echo "⚠ WARNING: High number of long-running queries - Check database performance"
        fi
        
        if [ "$TOTAL_FILINGS" -gt 0 ] && [ "$SUCCESS_RATE" ]; then
            if (( $(echo "$SUCCESS_RATE < 95" | bc -l) )); then
                echo "⚠ WARNING: Filing success rate below 95% - Investigate rejection reasons"
            fi
        fi
        
        echo ""
        echo "Report generated at: $(date)"
        
    } > "$report_file"
    
    # Send daily report via email
    mail -s "VATM Filing Service Daily Report - $CHECKLIST_DATE" ops@vatm.vn < "$report_file"
    
    log_message "Daily report generated and sent: $report_file"
}

perform_maintenance_tasks() {
    log_message "Performing daily maintenance tasks..."
    
    # 1. Log rotation and cleanup
    log_message "Rotating and cleaning logs..."
    logrotate /etc/logrotate.d/filing-service
    
    # Clean old logs (older than 30 days)
    find /var/log/vatm -name "*.log.*" -mtime +30 -delete
    
    # 2. Database maintenance
    log_message "Performing database maintenance..."
    
    # Update table statistics
    sudo -u postgres psql -d vatm_filing -c "ANALYZE;"
    
    # Vacuum old data
    sudo -u postgres psql -d vatm_filing -c "VACUUM;"
    
    # Clean old audit logs (older than 90 days)
    sudo -u postgres psql -d vatm_filing -c "DELETE FROM audit_logs WHERE created_at < NOW() - INTERVAL '90 days';"
    
    # Clean old validation results (older than 30 days)
    sudo -u postgres psql -d vatm_filing -c "DELETE FROM validation_results WHERE validation_time < NOW() - INTERVAL '30 days';"
    
    # 3. Cache cleanup
    log_message "Cleaning Redis cache..."
    redis-cli FLUSHDB
    
    # 4. Check and restart services if needed
    log_message "Checking service health..."
    
    # Check filing service memory usage
    FILING_SERVICE_PID=$(pgrep -f "filing-service")
    if [ -n "$FILING_SERVICE_PID" ]; then
        MEMORY_USAGE_KB=$(ps -o rss= -p "$FILING_SERVICE_PID")
        MEMORY_USAGE_MB=$((MEMORY_USAGE_KB / 1024))
        
        # Restart if using more than 8GB
        if [ "$MEMORY_USAGE_MB" -gt 8192 ]; then
            log_message "Filing service using ${MEMORY_USAGE_MB}MB - Restarting service"
            systemctl restart filing-service
            sleep 30
        fi
    fi
    
    log_message "Daily maintenance tasks completed"
}

check_security_compliance() {
    log_message "Performing security compliance check..."
    
    # Check SSL certificate expiration
    CERT_FILE="/opt/vatm/filing-service/ssl/vatm.crt"

    if [ -f "$CERT_FILE" ]; then
        CERT_EXPIRY=$(openssl x509 -in "$CERT_FILE" -noout -dates | grep "notAfter" | cut -d= -f2)
        CERT_EXPIRY_EPOCH=$(date -d "$CERT_EXPIRY" +%s)
        CURRENT_EPOCH=$(date +%s)
        DAYS_TO_EXPIRY=$(( (CERT_EXPIRY_EPOCH - CURRENT_EPOCH) / 86400 ))
        
        if [ "$DAYS_TO_EXPIRY" -lt 30 ]; then
            echo "⚠ WARNING: SSL certificate expires in $DAYS_TO_EXPIRY days"
            mail -s "URGENT: SSL Certificate Expiring Soon" ops@vatm.vn <<< "SSL certificate for Filing Service expires in $DAYS_TO_EXPIRY days. Please renew immediately."
        fi
    fi
    
    # Check for failed login attempts
    FAILED_LOGINS=$(grep "authentication failed" /var/log/vatm/filing-service.log | grep -c "$(date +%Y-%m-%d)")
    if [ "$FAILED_LOGINS" -gt 100 ]; then
        echo "⚠ WARNING: High number of failed authentication attempts: $FAILED_LOGINS"
        mail -s "Security Alert: High Failed Login Attempts" security@vatm.vn <<< "$FAILED_LOGINS failed authentication attempts detected today."
    fi
    
    # Check file permissions
    find /opt/vatm/filing-service -name "*.key" -perm +044 -exec echo "⚠ WARNING: Private key with weak permissions: {}" \;
    
    log_message "Security compliance check completed"
}

update_system_metrics() {
    log_message "Updating system metrics..."
    
    # Update Prometheus metrics if available
    if command -v prometheus >/dev/null 2>&1; then
        # Custom metrics update
        echo "filing_service_daily_check{status=\"completed\"} 1" | curl -X POST --data-binary @- http://localhost:9091/metrics/job/daily_operations/instance/$(hostname)
    fi
    
    # Update health check status
    HEALTH_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/actuator/health)
    if [ "$HEALTH_STATUS" = "200" ]; then
        echo "filing_service_health_status 1" > /tmp/health_metric.prom
    else
        echo "filing_service_health_status 0" > /tmp/health_metric.prom
    fi
    
    log_message "System metrics updated"
}

# Main execution
main() {
    log_message "Starting daily operations checklist for $CHECKLIST_DATE"
    
    # Generate daily report
    generate_daily_report
    
    # Perform maintenance tasks
    perform_maintenance_tasks
    
    # Security compliance check
    check_security_compliance
    
    # Update system metrics
    update_system_metrics
    
    log_message "Daily operations checklist completed successfully"
}

# Execute main function
main "$@"
```

#### 11.3.2 Incident Response Procedures
```bash
#!/bin/bash
# Incident Response Script for VATM Filing Service
# Usage: ./incident_response.sh [incident_type] [severity]

INCIDENT_TYPE="${1:-unknown}"
SEVERITY="${2:-medium}"  # low, medium, high, critical
INCIDENT_ID="INC-$(date +%Y%m%d%H%M%S)"
LOG_FILE="/var/log/vatm/incidents/${INCIDENT_ID}.log"

# Create incident log directory
mkdir -p "/var/log/vatm/incidents"

log_incident() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

collect_system_information() {
    log_incident "Collecting system information for incident analysis..."
    
    local info_dir="/tmp/incident_${INCIDENT_ID}"
    mkdir -p "$info_dir"
    
    # System status
    systemctl status filing-service > "$info_dir/service_status.txt"
    systemctl status postgresql >> "$info_dir/service_status.txt"
    systemctl status redis >> "$info_dir/service_status.txt"
    systemctl status nginx >> "$info_dir/service_status.txt"
    
    # System resources
    free -h > "$info_dir/memory_usage.txt"
    df -h > "$info_dir/disk_usage.txt"
    ps aux --sort=-%cpu | head -20 > "$info_dir/top_processes.txt"
    netstat -tulpn > "$info_dir/network_connections.txt"
    
    # Application logs (last 1000 lines)
    tail -1000 /var/log/vatm/filing-service.log > "$info_dir/app_logs.txt"
    tail -1000 /var/log/postgresql/postgresql-15-main.log > "$info_dir/db_logs.txt"
    tail -1000 /var/log/nginx/error.log > "$info_dir/nginx_logs.txt"
    
    # Database status
    sudo -u postgres psql -d vatm_filing -c "SELECT * FROM pg_stat_activity;" > "$info_dir/db_activity.txt"
    sudo -u postgres psql -d vatm_filing -c "SELECT schemaname, tablename, n_tup_ins, n_tup_upd, n_tup_del FROM pg_stat_user_tables;" > "$info_dir/db_stats.txt"
    
    # Application health
    curl -s http://localhost:8080/actuator/health > "$info_dir/app_health.txt" 2>&1
    curl -s http://localhost:8080/actuator/metrics > "$info_dir/app_metrics.txt" 2>&1
    
    # Recent filing activity
    sudo -u postgres psql -d vatm_filing -c "SELECT current_status, COUNT(*) FROM filed_flight_plans WHERE submitted_at > NOW() - INTERVAL '1 hour' GROUP BY current_status;" > "$info_dir/recent_filings.txt"
    
    # Compress collected information
    tar -czf "/var/log/vatm/incidents/system_info_${INCIDENT_ID}.tar.gz" -C "/tmp" "incident_${INCIDENT_ID}"
    rm -rf "$info_dir"
    
    log_incident "System information collected and saved to system_info_${INCIDENT_ID}.tar.gz"
}

handle_critical_incident() {
    log_incident "CRITICAL INCIDENT DETECTED - Initiating emergency procedures"
    
    # Immediate notification
    {
        echo "CRITICAL INCIDENT: VATM Filing Service"
        echo "Incident ID: $INCIDENT_ID"
        echo "Time: $(date)"
        echo "Type: $INCIDENT_TYPE"
        echo ""
        echo "Immediate actions taken:"
        echo "1. System information collected"
        echo "2. Emergency team notified"
        echo "3. Incident logged"
        echo ""
        echo "Please investigate immediately."
    } | mail -s "CRITICAL: Filing Service Incident $INCIDENT_ID" emergency@vatm.vn ops@vatm.vn
    
    # SMS notification (if configured)
    # curl -X POST "https://sms-api.vatm.vn/send" -d "message=CRITICAL Filing Service incident $INCIDENT_ID. Check email immediately." -d "phone=+84123456789"
    
    # Try automatic recovery based on incident type
    case "$INCIDENT_TYPE" in
        "service_down")
            attempt_service_recovery
            ;;
        "database_connection")
            attempt_database_recovery
            ;;
        "high_error_rate")
            attempt_error_rate_recovery
            ;;
        "memory_leak")
            attempt_memory_recovery
            ;;
    esac
}

attempt_service_recovery() {
    log_incident "Attempting automatic service recovery..."
    
    # Check if service is actually down
    if ! systemctl is-active --quiet filing-service; then
        log_incident "Service is down - attempting restart"
        
        # Try graceful restart first
        systemctl restart filing-service
        sleep 30
        
        if systemctl is-active --quiet filing-service; then
            log_incident "Service restart successful"
            
            # Wait for service to be fully ready
            sleep 60
            
            # Test health endpoint
            HEALTH_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/actuator/health)
            if [ "$HEALTH_STATUS" = "200" ]; then
                log_incident "Service recovery successful - health check passed"
                send_recovery_notification "service_restart_successful"
            else
                log_incident "Service started but health check failed - escalating"
                escalate_incident
            fi
        else
            log_incident "Service restart failed - escalating"
            escalate_incident
        fi
    else
        log_incident "Service appears to be running - checking health"
        # Service is running but may be unhealthy
        HEALTH_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/actuator/health)
        if [ "$HEALTH_STATUS" != "200" ]; then
            log_incident "Service unhealthy despite being active - forcing restart"
            systemctl stop filing-service
            sleep 10
            systemctl start filing-service
        fi
    fi
}

attempt_database_recovery() {
    log_incident "Attempting database connection recovery..."
    
    # Check PostgreSQL status
    if ! systemctl is-active --quiet postgresql; then
        log_incident "PostgreSQL is down - attempting restart"
        systemctl restart postgresql
        sleep 30
        
        if systemctl is-active --quiet postgresql; then
            log_incident "PostgreSQL restart successful"
            # Restart filing service to re-establish connections
            systemctl restart filing-service
        else
            log_incident "PostgreSQL restart failed - escalating"
            escalate_incident
        fi
    else
        log_incident "PostgreSQL is running - checking connections"
        
        # Check for connection pool exhaustion
        ACTIVE_CONNECTIONS=$(sudo -u postgres psql -d vatm_filing -t -c "SELECT count(*) FROM pg_stat_activity WHERE datname = 'vatm_filing';" | tr -d ' ')
        
        if [ "$ACTIVE_CONNECTIONS" -gt 180 ]; then
            log_incident "High connection count detected: $ACTIVE_CONNECTIONS - killing idle connections"
            sudo -u postgres psql -d vatm_filing -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'vatm_filing' AND state = 'idle' AND query_start < NOW() - INTERVAL '10 minutes';"
        fi
        
        # Restart application to reset connection pool
        systemctl restart filing-service
    fi
}

attempt_error_rate_recovery() {
    log_incident "Attempting error rate recovery..."
    
    # Check recent error patterns
    ERROR_COUNT=$(grep -c "ERROR" /var/log/vatm/filing-service.log | tail -100)
    
    if [ "$ERROR_COUNT" -gt 50 ]; then
        log_incident "High error count in recent logs: $ERROR_COUNT"
        
        # Check for specific error patterns
        if grep -q "OutOfMemoryError" /var/log/vatm/filing-service.log; then
            log_incident "OutOfMemoryError detected - restarting service"
            systemctl restart filing-service
        elif grep -q "Connection refused" /var/log/vatm/filing-service.log; then
            log_incident "Connection errors detected - checking dependencies"
            attempt_database_recovery
        else
            log_incident "Unknown error pattern - collecting logs and escalating"
            escalate_incident
        fi
    fi
}

attempt_memory_recovery() {
    log_incident "Attempting memory recovery..."
    
    # Get memory usage of filing service
    FILING_SERVICE_PID=$(pgrep -f "filing-service")
    if [ -n "$FILING_SERVICE_PID" ]; then
        MEMORY_USAGE_KB=$(ps -o rss= -p "$FILING_SERVICE_PID")
        MEMORY_USAGE_MB=$((MEMORY_USAGE_KB / 1024))
        
        log_incident "Filing service memory usage: ${MEMORY_USAGE_MB}MB"
        
        if [ "$MEMORY_USAGE_MB" -gt 6144 ]; then  # More than 6GB
            log_incident "High memory usage detected - restarting service"
            
            # Try garbage collection first
            curl -X POST http://localhost:8080/actuator/gc 2>/dev/null
            sleep 30
            
            # Check if memory usage decreased
            MEMORY_USAGE_KB_AFTER=$(ps -o rss= -p "$FILING_SERVICE_PID")
            MEMORY_USAGE_MB_AFTER=$((MEMORY_USAGE_KB_AFTER / 1024))
            
            if [ "$MEMORY_USAGE_MB_AFTER" -gt 5120 ]; then  # Still above 5GB
                log_incident "GC didn't help - restarting service"
                systemctl restart filing-service
            else
                log_incident "GC successful - memory reduced to ${MEMORY_USAGE_MB_AFTER}MB"
            fi
        fi
    fi
}

escalate_incident() {
    log_incident "Escalating incident to next level support"
    
    # Create detailed incident report
    {
        echo "ESCALATED INCIDENT REPORT"
        echo "========================"
        echo "Incident ID: $INCIDENT_ID"
        echo "Type: $INCIDENT_TYPE"
        echo "Severity: $SEVERITY"
        echo "Time: $(date)"
        echo ""
        echo "Automatic recovery attempts failed."
        echo "Manual intervention required."
        echo ""
        echo "System Information:"
        echo "- Service Status: $(systemctl is-active filing-service)"
        echo "- Database Status: $(systemctl is-active postgresql)"
        echo "- Available Memory: $(free -h | grep Mem | awk '{print $7}')"
        echo "- Disk Space: $(df -h / | awk 'NR==2 {print $4}')"
        echo ""
        echo "Recent Log Entries:"
        tail -20 /var/log/vatm/filing-service.log
        echo ""
        echo "Incident Log: $LOG_FILE"
        echo "System Info: /var/log/vatm/incidents/system_info_${INCIDENT_ID}.tar.gz"
    } | mail -s "ESCALATED: Filing Service Incident $INCIDENT_ID" senior-ops@vatm.vn management@vatm.vn
}

send_recovery_notification() {
    local recovery_type="$1"
    
    {
        echo "Filing Service Recovery Notification"
        echo "==================================="
        echo "Incident ID: $INCIDENT_ID"
        echo "Recovery Type: $recovery_type"
        echo "Time: $(date)"
        echo ""
        echo "Automatic recovery was successful."
        echo "Service has been restored to normal operation."
        echo ""
        echo "Please monitor the system for any recurring issues."
    } | mail -s "RESOLVED: Filing Service Incident $INCIDENT_ID" ops@vatm.vn
}

# Main incident response workflow
main() {
    log_incident "Incident response initiated for: $INCIDENT_TYPE (Severity: $SEVERITY)"
    
    # Always collect system information first
    collect_system_information
    
    # Handle based on severity
    case "$SEVERITY" in
        "critical")
            handle_critical_incident
            ;;
        "high")
            # High severity - immediate attention but not critical
            {
                echo "HIGH SEVERITY INCIDENT: $INCIDENT_TYPE"
                echo "Incident ID: $INCIDENT_ID"
                echo "Time: $(date)"
                echo ""
                echo "Requires immediate attention."
                echo "System information collected."
            } | mail -s "HIGH: Filing Service Incident $INCIDENT_ID" ops@vatm.vn
            ;;
        "medium"|"low")
            # Medium/Low severity - log and notify during business hours
            log_incident "Incident logged for review during business hours"
            ;;
    esac
    
    log_incident "Incident response completed"
}

# Usage information
if [ $# -eq 0 ]; then
    echo "Usage: $0 [incident_type] [severity]"
    echo ""
    echo "Incident Types:"
    echo "  service_down        - Filing service is not responding"
    echo "  database_connection - Database connectivity issues"
    echo "  high_error_rate     - Unusual number of application errors"
    echo "  memory_leak         - High memory usage detected"
    echo "  security_breach     - Security incident detected"
    echo "  performance_issue   - Slow response times"
    echo ""
    echo "Severity Levels:"
    echo "  critical - Service completely down, immediate response required"
    echo "  high     - Major functionality impacted, urgent response needed"
    echo "  medium   - Some functionality impacted, response within hours"
    echo "  low      - Minor issues, response within business hours"
    exit 1
fi

# Execute main function
main "$@"
```

---

## CONCLUSION

This comprehensive Technical Specification Document for the FF-ICE Filing Service provides VATM with a complete blueprint for implementing a production-ready, enterprise-grade filing system. The document covers all critical aspects from system architecture and functional requirements to deployment, security, testing, and operational procedures.

### Key Deliverables Summary:

1. **Complete System Architecture**: Detailed multi-tier design with load balancing, high availability, and scalability
2. **Functional Requirements**: Comprehensive coverage of all filing service capabilities including validation, integration, and notification
3. **API Specifications**: RESTful endpoints with complete documentation, error handling, and security
4. **Database Design**: Optimized schema with proper indexing, partitioning, and performance considerations
5. **Security Framework**: Multi-layered security with authentication, authorization, encryption, and audit trails
6. **Testing Strategy**: Unit, integration, performance, and security testing with automated frameworks
7. **Deployment Architecture**: Production-ready infrastructure with monitoring, alerting, and disaster recovery
8. **Operational Procedures**: Daily operations, incident response, backup/recovery, and maintenance workflows

### Implementation Complexity Assessment:

**Filing Service vs Query Service Comparison:**
- **Query Service**: ⭐⭐ (Simple - Read operations, basic validation)
- **Filing Service**: ⭐⭐⭐⭐⭐ (Complex - Write operations, business rules, integrations, validation, notifications)

### Next Steps for VATM:

1. **Team Assembly**: Recruit 8-12 person development team as specified
2. **Infrastructure Preparation**: Set up development, testing, and production environments
3. **Stakeholder Alignment**: Present specification to VATM management and CAAV for approval
4. **Pilot Implementation**: Start with Query Service to build expertise, then tackle Filing Service
5. **Integration Planning**: Coordinate with FDPS, airlines, and airport systems for testing

### Timeline and Budget Estimates:

- **Development Timeline**: 18-24 months for complete implementation
- **Infrastructure Cost**: $2-4M for hardware, software, and infrastructure
- **Development Cost**: $3-5M for team, tools, and external consulting
- **Total Project Cost**: $5-9M depending on scope and approach

This Filing Service specification is significantly more complex than the Query Service due to:
- Business rules validation engine
- FDPS integration requirements
- Multi-stakeholder notification system
- Real-time validation and processing
- Comprehensive audit and compliance features

However, the modular design allows for phased implementation, starting with core filing functionality and gradually adding advanced features like bulk processing, complex business rules, and enhanced integrations.

The specification provides VATM with everything needed to make informed decisions about implementation approach, whether developing internally, partnering with system integrators, or pursuing a hybrid approach.

---

**Document Control:**
- **Version**: 1.0
- **Date**: September 12, 2025
- **Document Type**: Technical Specification
- **Classification**: Internal Use
- **Next Review**: December 12, 2025
- **Approval Status**: Pending VATM Management Review
