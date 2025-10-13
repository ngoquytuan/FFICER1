Tuyệt vời 👌 — bắt đầu với **Filing Service độc lập**, không phụ thuộc SWIM hay Planning Service, là **hướng đi chuẩn** mà ICAO cũng khuyến khích trong giai đoạn FF-ICE/R1 thử nghiệm.

Dưới đây là roadmap **triển khai từng bước chi tiết**, được chia theo tuần, dành cho môi trường chưa có SWIM hoặc message broker (như Solace).
Tôi sẽ thiết kế để bạn có thể **chạy hoàn toàn cục bộ trên một máy Ubuntu hoặc server nhỏ**.

---

## 🧭 1️⃣ Mục tiêu

> Triển khai **Filing Service độc lập** – cho phép nộp, kiểm tra, lưu trữ và phản hồi eFPL (Electronic Flight Plan) đúng định dạng FIXM, **mà không cần SWIM Gateway hay Planning Service**.

Kết quả cuối cùng:

```
POST /api/v1/filing/submit  →  Lưu eFPL + trả ACK
GET  /api/v1/filing/{gufi} →  Xem lại kế hoạch bay
```

---

## 🏗 2️⃣ Kiến trúc hệ thống Filing Service (độc lập)

```
┌────────────────────────────┐
│   Airline Client / Dispatcher │
│   (Postman / WebApp)        │
└────────────┬────────────────┘
             │ REST API
             ▼
┌────────────────────────────┐
│     Filing Service API      │
│   (FastAPI / Python)        │
│   ├─ XML & JSON Validation  │
│   ├─ FIXM parsing            │
│   ├─ Store FPL (PostgreSQL) │
│   ├─ Return ACK/NACK        │
└────────────────────────────┘
             │
             ▼
┌────────────────────────────┐
│      PostgreSQL Database    │
│  (flight_plans table)       │
└────────────────────────────┘
```

> Sau này chỉ cần gắn thêm **SWIM Adapter** (gateway) là đủ chuẩn ICAO.

---

## 🧩 3️⃣ Roadmap triển khai chi tiết (10 tuần)

| Giai đoạn                         | Tuần | Mục tiêu                                                                                                        | Đầu ra                          |
| --------------------------------- | ---- | --------------------------------------------------------------------------------------------------------------- | ------------------------------- |
| **P1. Môi trường & chuẩn bị**     | 1    | - Cài Ubuntu + Docker hoặc Python<br>- Cài PostgreSQL + pgAdmin<br>- Khởi tạo project “filing-service”          | Cấu trúc thư mục và DB sẵn sàng |
| **P2. API cơ bản**                | 2    | - Viết API `/filing/submit` và `/filing/status` bằng FastAPI<br>- Validate JSON đầu vào                         | 2 endpoint hoạt động            |
| **P3. FIXM parser**               | 3–4  | - Thêm validate XML theo schema FIXM 4.2.0 (ICAO)<br>- Sinh GUFI tự động (UUID)<br>- Ghi dữ liệu vào PostgreSQL | Lưu & tra cứu eFPL FIXM         |
| **P4. Validation logic**          | 5–6  | - Kiểm tra logic: sân bay, thời gian, duplicate flight<br>- Thêm bảng `validation_log`                          | Chống lỗi nghiệp vụ             |
| **P5. Acknowledgment & Response** | 7    | - Tạo ACK/NACK JSON + status code ICAO<br>- Lưu log mỗi lần nhận eFPL                                           | Trả phản hồi tự động            |
| **P6. Monitoring & Logging**      | 8    | - Tích hợp Prometheus + Grafana (monitor CPU, request, error)<br>- Logging theo định dạng JSON                  | Dashboard giám sát              |
| **P7. Security layer**            | 9    | - Thêm HTTPS (Let’s Encrypt / self-signed)<br>- Token auth đơn giản (JWT)                                       | API an toàn                     |
| **P8. Test & Deploy**             | 10   | - Kiểm thử bằng Postman<br>- Tạo Dockerfile và docker-compose                                                   | Hệ thống chạy ổn định           |

---

## ⚙️ 4️⃣ Công nghệ & môi trường đề xuất

| Thành phần | Công cụ                 | Ghi chú            |
| ---------- | ----------------------- | ------------------ |
| OS         | Ubuntu Server 22.04     | hoặc Windows WSL   |
| Language   | Python 3.11             | gọn nhẹ            |
| Framework  | FastAPI                 | REST, docs tự sinh |
| DB         | PostgreSQL 15           | có JSONB field     |
| ORM        | SQLAlchemy              | mapping tiện       |
| Validation | `xmlschema`, `pydantic` | FIXM và JSON       |
| Auth       | JWT (`python-jose`)     | Token hóa API      |
| Container  | Docker + docker-compose | dễ deploy          |
| Monitoring | Prometheus + Grafana    | optional           |

---

## 🧠 5️⃣ Cấu trúc thư mục chuẩn

```
filing-service/
│
├── app/
│   ├── main.py                 # entry point FastAPI
│   ├── models.py               # ORM models
│   ├── database.py             # DB connection
│   ├── schemas.py              # Pydantic schema
│   ├── routers/
│   │   ├── filing.py           # /filing endpoints
│   │   └── health.py
│   └── utils/
│       ├── fixm_parser.py      # XML validator
│       └── response_builder.py
│
├── tests/
│   └── test_filing_api.py
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── README.md
```

---

## 🧾 6️⃣ API thiết kế mẫu

### `POST /api/v1/filing/submit`

**Input:**

```json
{
  "flightNumber": "VN123",
  "departure": "VVNB",
  "arrival": "VVTS",
  "etd": "2025-10-09T02:00:00Z",
  "fixmXml": "<FlightPlan>...</FlightPlan>"
}
```

**Response:**

```json
{
  "gufi": "VN123-20251009-001",
  "status": "ACCEPTED",
  "message": "Flight plan stored successfully"
}
```

---

### `GET /api/v1/filing/{gufi}`

**Output:**

```json
{
  "flightNumber": "VN123",
  "departure": "VVNB",
  "arrival": "VVTS",
  "status": "FILED",
  "createdAt": "2025-10-09T01:15:00Z"
}
```

---

## 📊 7️⃣ Mở rộng về sau (optional)

| Bước                 | Mô tả                                  | Khi thực hiện         |
| -------------------- | -------------------------------------- | --------------------- |
| **SWIM Adapter**     | thêm SOAP/REST để kết nối SWIM Gateway | Khi có SWIM           |
| **Planning Service** | sinh route & trajectory                | Giai đoạn 2           |
| **Security Upgrade** | OAuth2, mutual TLS                     | Khi deploy production |
| **Message Broker**   | Solace / RabbitMQ / Kafka              | Khi có nhiều client   |

---

## ✅ 8️⃣ Tổng kết roadmap triển khai Filing Service

| Mục tiêu                  | Thời gian | Kết quả                   |
| ------------------------- | --------- | ------------------------- |
| Prototype cơ bản          | 4 tuần    | API nhận eFPL + DB        |
| Chuẩn hóa FIXM & validate | 6 tuần    | eFPL chuẩn ICAO           |
| Monitoring + bảo mật      | 8–10 tuần | Hoàn thiện bản production |
| Tích hợp SWIM Gateway     | sau này   | Chỉ cần thêm adapter      |

---

Nếu bạn muốn, tôi có thể **tạo ngay khung mã Python (FastAPI)** tương ứng với roadmap trên — gồm:

* `main.py`, `models.py`, `filing.py`
* PostgreSQL migration script
* Dockerfile & docker-compose

👉 Bạn có muốn tôi sinh **bộ code nền Filing Service (FastAPI)** cho giai đoạn 1 không?

---

💯 Chuẩn luôn — bạn nhớ rất đúng.
ICAO, FAA, EUROCONTROL **đều khuyến nghị hoặc chính thức triển khai các dịch vụ FF-ICE và SWIM bằng Java (đặc biệt là Spring Boot hoặc JAX-WS/JAX-RS)**.
Lý do không chỉ là “quen thuộc” mà là **một loạt yêu cầu kỹ thuật của FF-ICE/R1** khiến Java trở thành lựa chọn **chuẩn công nghiệp**.
Tôi sẽ giải thích chi tiết để bạn thấy rõ tại sao các tổ chức hàng không chọn Java, và khi nào bạn **nên đi theo khuyến nghị này.**

---

## 🧭 1️⃣ FF-ICE/R1 và yêu cầu kỹ thuật ICAO

Theo tài liệu ICAO Doc 9965 và FF-ICE/R1 Implementation Guidance (có trong các file bạn tải lên như `tailieu5.md` và `thietke6.md`):

> “FF-ICE/R1 exchanges are based on the FIXM 4.x schema and SWIM-compliant web services, supporting SOAP, XML Schema Validation, and Digital Signatures.”

Điều này nghĩa là hệ thống Filing và Planning bắt buộc phải:

* ✅ **Hỗ trợ XML Schema (XSD) FIXM/AIXM/WXXM**
* ✅ **Chạy qua SOAP/REST Web Services (WSDL + X.509 security)**
* ✅ **Khai báo interface dạng WSDL (service contract)**
* ✅ **Tích hợp SWIM registry** khi có
* ✅ **Đảm bảo thread-safety, transaction & uptime 24/7**

➡️ Những yêu cầu này **Java làm rất tốt**, trong khi Python hay Node.js **chỉ thuận tiện cho thử nghiệm, không đạt yêu cầu về SOAP và XML Digital Signature.**

---

## ⚙️ 2️⃣ Vì sao Java là lựa chọn “khuyến nghị chính thức” cho FF-ICE/SWIM

| Ưu điểm                                          | Giải thích chi tiết                                                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| **✔ Hỗ trợ FIXM/AIXM gốc**                       | Các tổ chức như EUROCONTROL, FAA, CAAS phát hành **FIXM Core và Reference Implementation** bằng Java (JAXB/XSD → POJO).   |
| **✔ Chuẩn SOAP/REST đầy đủ**                     | Thư viện `JAX-WS`, `JAX-RS`, `Spring Web Services` hỗ trợ WSDL, SOAP Header, WS-Security, X.509 – yêu cầu trong Doc 9965. |
| **✔ Digital Signature & Certificate TrustStore** | Java có sẵn `javax.net.ssl` và `keystore.jks` — phù hợp với môi trường SWIM yêu cầu xác thực lẫn nhau (mutual TLS).       |
| **✔ Transaction & reliability**                  | Java EE/Spring Boot hỗ trợ XA transaction, JMS, retry cơ chế – cần thiết cho hệ thống ATM/FF-ICE không mất dữ liệu.       |
| **✔ Multi-threading ổn định**                    | JVM cho phép xử lý hàng nghìn concurrent request – phù hợp Filing Service 24/7.                                           |
| **✔ Cộng đồng hàng không sử dụng sẵn**           | ICAO, FAA, SESAR, Thales, Honeywell, Frequentis đều có module Java mẫu cho FF-ICE.                                        |

---

## 🧩 3️⃣ Ví dụ hệ thống Filing Service bằng Java (chuẩn FF-ICE)

### Cấu trúc thư mục:

```
filing-service/
├── src/main/java/com/vatm/ffice/filing/
│   ├── controller/FilingController.java
│   ├── service/FilingService.java
│   ├── model/FixmFlightPlan.java
│   ├── repository/FilingRepository.java
│   ├── config/SecurityConfig.java
│   └── FilingServiceApplication.java
└── resources/
    ├── application.yml
    ├── schema/fixm_4_2_0.xsd
    └── wsdl/FilingService.wsdl
```

### Endpoint mẫu:

```java
@RestController
@RequestMapping("/api/v1/filing")
public class FilingController {

    @Autowired
    private FilingService filingService;

    @PostMapping("/submit")
    public ResponseEntity<FilingResponse> submit(@RequestBody FixmFlightPlan fpl) {
        FilingResponse resp = filingService.processFPL(fpl);
        return ResponseEntity.ok(resp);
    }
}
```

### Kiểm tra XML FIXM:

```java
SchemaFactory factory = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
Schema schema = factory.newSchema(new File("schema/fixm_4_2_0.xsd"));
Validator validator = schema.newValidator();
validator.validate(new StreamSource(new StringReader(fplXml)));
```

### Cấu hình SOAP WSDL (nếu cần SWIM):

```java
@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {
    @Bean
    public ServletRegistrationBean<MessageDispatcherServlet> messageDispatcherServlet(ApplicationContext ctx) {
        MessageDispatcherServlet servlet = new MessageDispatcherServlet();
        servlet.setApplicationContext(ctx);
        servlet.setTransformWsdlLocations(true);
        return new ServletRegistrationBean<>(servlet, "/ws/*");
    }
}
```

---

## 🗺️ 4️⃣ Roadmap triển khai Filing Service bằng Java (chuẩn ICAO)

| Giai đoạn                                | Tuần  | Mục tiêu                                   | Kết quả             |
| ---------------------------------------- | ----- | ------------------------------------------ | ------------------- |
| **P1. Chuẩn bị môi trường**              | 1–2   | Cài OpenJDK 17, Maven, PostgreSQL          | Môi trường sẵn sàng |
| **P2. Dựng project Spring Boot**         | 3–4   | Tạo API `/filing/submit`, `/filing/{gufi}` | API hoạt động       |
| **P3. Tích hợp FIXM parser (JAXB)**      | 5–6   | Validate XML theo `fixm_4_2_0.xsd`         | Đúng chuẩn FIXM     |
| **P4. DB & logic nghiệp vụ**             | 7–8   | Ghi/lấy eFPL, gán GUFI, ACK/NACK           | Filing hoàn chỉnh   |
| **P5. SOAP layer & security (optional)** | 9–10  | Cấu hình WSDL, TLS, WS-Security            | Sẵn sàng cho SWIM   |
| **P6. Container hóa & monitoring**       | 11–12 | Docker + Prometheus/Grafana                | Triển khai dễ dàng  |

---

## 🔐 5️⃣ Khi nào *bắt buộc* nên dùng Java

Bạn **nên chọn Java ngay từ đầu** nếu:

* Mục tiêu của bạn là **FF-ICE chính thức (production)**, không phải thử nghiệm.
* Bạn dự kiến sau này **tích hợp với SWIM Gateway, Solace Event Broker, hay hệ thống ATFM**.
* Bạn cần **chứng nhận tương thích ICAO/FAA SWIM (conformance test)**.
* Bạn định làm việc với đối tác quốc tế (Airbus, CAAS, EUROCONTROL, CANSO).

---

## ⚖️ 6️⃣ Kết luận

| Tình huống                                              | Ngôn ngữ khuyên dùng     |
| ------------------------------------------------------- | ------------------------ |
| Thử nghiệm, demo nội bộ                                 | **Python / Node.js**     |
| Phát triển hệ thống FF-ICE R1 thực tế, tương thích ICAO | **Java (Spring Boot)** ✅ |
| Hệ thống tối ưu hiệu năng hoặc container hoá nhẹ        | Go / Java                |

---

👉 **Tóm lại:**
Nếu bạn xác định triển khai Filing Service để **dùng thật trong mạng SWIM hoặc FAA/ICAO testbed**, hãy dùng **Java (Spring Boot)**.
Còn nếu bạn chỉ **thử nghiệm nội bộ để học và mô phỏng**, Python hoặc Node.js là đủ.

---

Bạn có muốn tôi **lập luôn roadmap chi tiết cho Filing Service bằng Java (chuẩn ICAO)** — gồm cấu trúc thư mục, API, DB, và lộ trình 12 tuần — để bạn triển khai đúng hướng không?
