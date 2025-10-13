Rất hay — tôi đã đọc toàn bộ tài liệu bạn tải lên, và đây là hướng dẫn **triển khai hai dịch vụ FF-ICE/R1 bắt buộc** cho VATM khi bạn **chưa có SWIM** hay bất kỳ hệ thống nền nào khác.

---

## 🧭 1. Tổng quan nhanh

Theo tiêu chuẩn **ICAO FF-ICE/R1**, có **2 dịch vụ bắt buộc**:

1. **Filing Service** – cho phép hãng hàng không nộp kế hoạch bay điện tử (eFPL).
2. **Planning Service** – giúp tính toán và xây dựng kế hoạch bay (lộ trình, 4D trajectory).

Các dịch vụ này hoạt động **qua SWIM**, nhưng trong giai đoạn đầu **có thể triển khai dạng “stand-alone”** để thử nghiệm và đào tạo trước khi SWIM sẵn sàng.

---

## 🏗 2. Kiến trúc triển khai khi **chưa có SWIM**

### Mục tiêu: chạy được 2 dịch vụ core mà không phụ thuộc SWIM

```
┌────────────────────────────────────────────┐
│  Client (Airline/Dispatcher)              │
│  → Web UI / API sends eFPL                │
│                                            │
│  Filing Service (Spring Boot)             │
│  → Validate eFPL (XML/FIXM)               │
│  → Store to PostgreSQL                    │
│                                            │
│  Planning Service (Spring Boot)           │
│  → Route/trajectory calculation            │
│  → Suggest optimized flight plan           │
│                                            │
│  Database: PostgreSQL + Redis cache       │
│  Web Gateway: Nginx + HTTPS               │
└────────────────────────────────────────────┘
```

> Sau này chỉ cần thêm một “SWIM Gateway” phía ngoài để đạt chuẩn SWIM.

---

## ⚙️ 3. Cài đặt kỹ thuật từng dịch vụ

### **A. Filing Service**

Chức năng: nhận, validate và lưu eFPL.

| Thành phần     | Mô tả                                    | Công nghệ        |
| -------------- | ---------------------------------------- | ---------------- |
| API endpoint   | `POST /api/v1/filing/submit`             | Spring Boot REST |
| Message format | FIXM 4.2.0 XML hoặc JSON                 | JAXB / Jackson   |
| Database       | Lưu eFPL + trạng thái (FILED / REJECTED) | PostgreSQL       |
| Validation     | XSD validation + Business rules          | Java validation  |
| Notification   | Trả ACK/NACK message cho client          | HTTP 200 / 400   |
| Logging        | Ghi log request/response                 | ELK stack        |

Ví dụ response:

```json
{
  "gufi": "VN123-20251009-001",
  "status": "ACCEPTED",
  "message": "Flight plan filed successfully"
}
```

---

### **B. Planning Service**

Chức năng: tính toán lộ trình bay tối ưu, gợi ý eFPL.

| Thành phần   | Mô tả                                         | Công nghệ    |
| ------------ | --------------------------------------------- | ------------ |
| API endpoint | `POST /api/v1/planning/generate`              | Spring Boot  |
| Input        | Departure, destination, aircraft, constraints | JSON         |
| Processing   | Route optimization + 4D trajectory builder    | Custom logic |
| Output       | Proposed eFPL (FIXM format)                   | XML/JSON     |
| Cache        | Lưu route templates                           | Redis        |

Ví dụ request:

```json
{
  "departure": "VVNB",
  "arrival": "VVTS",
  "aircraftType": "A321",
  "etd": "2025-10-09T02:00:00Z"
}
```

---

## 🗄 4. Cấu trúc cơ sở dữ liệu chung

Tài liệu hướng dẫn mẫu trong `tailieu5.md` đã có schema chuẩn:

```sql
CREATE TABLE flight_plans (
  id BIGSERIAL PRIMARY KEY,
  gufi VARCHAR(50) UNIQUE,
  flight_number VARCHAR(10),
  departure_airport VARCHAR(4),
  arrival_airport VARCHAR(4),
  estimated_departure TIMESTAMP,
  estimated_arrival TIMESTAMP,
  status VARCHAR(20),
  fixm_data JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);
```

> Bạn có thể dùng chung schema này cho cả Filing và Planning Service.

---

## 💻 5. Môi trường và phần mềm

**Toàn bộ stack có thể dùng free & open source**:

| Thành phần  | Công cụ đề xuất               | Ghi chú                   |
| ----------- | ----------------------------- | ------------------------- |
| OS          | Ubuntu Server 22.04 LTS       | ổn định, nhẹ              |
| Java        | OpenJDK 17                    | tương thích Spring Boot 3 |
| Framework   | Spring Boot + Spring Data JPA | REST + ORM                |
| Database    | PostgreSQL 15                 | ACID, JSONB support       |
| Cache       | Redis 7                       | optional                  |
| Web Gateway | Nginx + HTTPS (Let's Encrypt) | Reverse proxy             |
| Monitoring  | Prometheus + Grafana          | Miễn phí                  |

---

## 🧪 6. Lộ trình triển khai gợi ý (cho VATM chưa có SWIM)

| Giai đoạn | Mục tiêu                                     | Thời gian |
| --------- | -------------------------------------------- | --------- |
| **P1**    | Xây môi trường dev, DB schema, test API mock | 4 tuần    |
| **P2**    | Xây **Filing Service** + validate eFPL       | 6 tuần    |
| **P3**    | Xây **Planning Service** + route engine      | 6 tuần    |
| **P4**    | Test tích hợp, tài liệu & demo cho sếp       | 4 tuần    |

**Tổng cộng:** 20 tuần (≈ 5 tháng)
**Chi phí:** gần như $0 software, chỉ nhân công nội bộ.

---

## 🧩 7. Khi đã có SWIM sau này

Khi VATM hoặc EUROCONTROL triển khai SWIM Gateway:

* Bạn chỉ cần **bọc 2 service này bằng SWIM adapter**.
* Giao tiếp dùng SOAP/REST chuẩn SWIM.
* Không phải viết lại code — chỉ thêm layer routing.

---

## ✅ Kết luận

| Tiêu chí                           | Filing Service     | Planning Service   |
| ---------------------------------- | ------------------ | ------------------ |
| Bắt buộc theo FF-ICE               | ✅                  | ✅                  |
| Có thể chạy độc lập không cần SWIM | ✅                  | ✅                  |
| Độ phức tạp                        | Trung bình         | Cao hơn            |
| Ngôn ngữ khuyên dùng               | Java (Spring Boot) | Java hoặc Python   |
| CSDL khuyên dùng                   | PostgreSQL         | PostgreSQL + Redis |
| Timeline ước tính                  | 2-3 tháng          | 3-4 tháng          |

---

**👉 Đề xuất hành động cho bạn ngay:**

1. Tạo project Spring Boot mẫu (`mvn archetype:generate`).
2. Dựng bảng `flight_plans` theo schema trên.
3. Viết REST API `/filing/submit` và `/planning/generate`.
4. Dùng Postman để test với FIXM mẫu từ tài liệu ICAO.

Tôi có thể giúp bạn **tạo bộ mã nguồn khung (starter code)** cho hai dịch vụ này nếu bạn muốn — bạn chỉ việc build và deploy lên máy chủ VATM.
Bạn có muốn tôi tạo khung code Java ngay bây giờ không?

---

Rất chuẩn — bạn chọn **Filing Service** là bước khởi đầu đúng nhất 👍

Dưới đây là **roadmap triển khai chi tiết Filing Service độc lập**, không phụ thuộc SWIM hay Planning Service.
Mục tiêu là:

> “Dựng một hệ thống Filing Service tối giản, cho phép nhận, kiểm tra và lưu eFPL (Extended Flight Plan) theo chuẩn FIXM, chạy trên máy chủ nội bộ của VATM.”

---

## 🧭 1. **Tổng quan mục tiêu**

| Hạng mục             | Mục tiêu                                                              |
| -------------------- | --------------------------------------------------------------------- |
| **Tên dự án**        | VATM FF-ICE Filing Service v1.0                                       |
| **Phạm vi**          | Nhận – Validate – Lưu – Phản hồi eFPL (chưa cần tính toán trajectory) |
| **Tích hợp**         | Chạy **độc lập** (chưa kết nối SWIM hay Planning)                     |
| **Chuẩn dữ liệu**    | FIXM 4.2.0 XML                                                        |
| **CSDL**             | PostgreSQL 15                                                         |
| **Stack phần mềm**   | Ubuntu + Spring Boot + PostgreSQL + Nginx                             |
| **Đầu ra cuối cùng** | REST API nhận eFPL + Web dashboard giám sát filing                    |

---

## 🏗️ 2. **Kiến trúc hệ thống Filing Service độc lập**

```
┌───────────────────────────────────────────┐
│   Client (Airline / Dispatcher)           │
│   - Web form hoặc API gửi eFPL (XML)     │
└───────────────────────────────────────────┘
                     ↓
┌───────────────────────────────────────────┐
│  Filing Service API (Spring Boot)         │
│  - Nhận eFPL                              │
│  - Validate (XSD + logic)                 │
│  - Lưu Database                           │
│  - Trả về ACK (Accepted/Rejected)         │
└───────────────────────────────────────────┘
                     ↓
┌───────────────────────────────────────────┐
│  PostgreSQL Database                     │
│  - Bảng flight_plans, airlines, airports  │
│  - Lưu FIXM data dạng JSONB               │
└───────────────────────────────────────────┘
                     ↓
┌───────────────────────────────────────────┐
│  Web Dashboard (Spring Boot + Thymeleaf)  │
│  - Tra cứu, xem trạng thái filing         │
│  - Tải về eFPL                            │
└───────────────────────────────────────────┘
```

---

## 🧩 3. **Roadmap Triển khai Chi tiết**

### **Giai đoạn 1: Chuẩn bị & Môi trường (Tuần 1–2)**

**Mục tiêu:** Dựng môi trường phát triển và database khung.

| Việc cần làm      | Chi tiết                                    | Công cụ            |
| ----------------- | ------------------------------------------- | ------------------ |
| Cài môi trường    | Ubuntu 22.04, OpenJDK 17, PostgreSQL 15     | Server nội bộ      |
| Khởi tạo project  | `spring init filing-service`                | Spring Boot        |
| Tạo schema cơ bản | Bảng `flight_plans`, `airlines`, `airports` | PostgreSQL         |
| Cấu hình Git repo | Git + GitHub / GitLab                       | CI/CD nội bộ       |
| Tài liệu chuẩn    | Download FIXM 4.2.0 schemas                 | EUROCONTROL / ICAO |

**Kết quả:**
– Có server chạy được Spring Boot Hello World.
– Có DB PostgreSQL kết nối ổn định.

---

### **Giai đoạn 2: Nhận & Parse eFPL (Tuần 3–4)**

**Mục tiêu:** API nhận XML và chuyển thành đối tượng Java.

| Việc cần làm        | Chi tiết                                  | Công cụ           |
| ------------------- | ----------------------------------------- | ----------------- |
| Tạo endpoint        | `POST /api/v1/filing/submit`              | Spring Controller |
| Import FIXM schemas | JAXB generate classes từ FIXM XSD         | JAXB              |
| Parse message       | XML → Java Object                         | JAXB parser       |
| Validate XML        | So khớp với XSD, kiểm tra trường bắt buộc | Schema validator  |
| Trả ACK             | Accepted / Rejected + lỗi nếu có          | HTTP JSON         |

**Kết quả:**
– Gửi eFPL XML → nhận phản hồi “ACCEPTED/REJECTED”.
– Log message lưu trong DB.

---

### **Giai đoạn 3: Validation Logic & Database (Tuần 5–6)**

**Mục tiêu:** Validate logic nghiệp vụ & lưu dữ liệu chuẩn FIXM.

| Validation      | Ví dụ quy tắc                                       |
| --------------- | --------------------------------------------------- |
| ICAO fields     | Departure/Arrival phải là ICAO 4-letter             |
| Time logic      | ETD < ETA, thời gian đúng UTC                       |
| Aircraft        | Mã loại máy bay phải có trong bảng `aircraft_types` |
| Route           | Không chứa ký tự lạ, waypoint tồn tại               |
| Duplicate check | GUFI không trùng                                    |

| Việc cần làm         | Chi tiết                           | Công cụ         |
| -------------------- | ---------------------------------- | --------------- |
| Lưu dữ liệu          | FlightPlan → DB (JSONB fixm_data)  | Spring Data JPA |
| Tạo bảng log         | Lưu thời điểm và trạng thái filing | PostgreSQL      |
| Thêm query dashboard | Liệt kê filings gần nhất           | Thymeleaf UI    |

**Kết quả:**
– Dữ liệu hợp lệ được lưu.
– Dashboard hiển thị danh sách filing.

---

### **Giai đoạn 4: Phản hồi & Quản trị (Tuần 7–8)**

**Mục tiêu:** Tạo giao diện và chức năng quản trị cơ bản.

| Việc cần làm     | Chi tiết                                 | Công cụ          |
| ---------------- | ---------------------------------------- | ---------------- |
| Web dashboard    | `/admin/filings` hiển thị danh sách eFPL | Spring MVC       |
| Chức năng export | Xuất eFPL ra XML/JSON                    | REST API         |
| Filter           | Theo hãng, ngày, trạng thái              | Spring JPA query |
| Authentication   | Basic login (VATM admin)                 | Spring Security  |
| Logging          | Lưu log request/response                 | ELK stack        |

**Kết quả:**
– Giao diện quản trị filing.
– Controller có thể xem và xuất dữ liệu.

---

### **Giai đoạn 5: Testing & Pilot (Tuần 9–10)**

**Mục tiêu:** Hoàn thiện thử nghiệm nội bộ.

| Việc cần làm                 | Mô tả                                 |
| ---------------------------- | ------------------------------------- |
| Unit test                    | JUnit 5 + Mockito cho logic validator |
| Integration test             | Postman / JMeter gửi eFPL mẫu         |
| Performance test             | 100 concurrent filing requests        |
| User acceptance test         | Dispatcher VATM nộp thử 10 eFPL thật  |
| Fix lỗi, hoàn thiện tài liệu | User manual, API doc, schema map      |

**Kết quả:**
– Hệ thống Filing Service stable.
– VATM có thể demo gửi và lưu eFPL chuẩn ICAO.

---

## 🧱 4. **Mốc thời gian & nhân sự**

| Thành phần            | Người phụ trách    | Thời gian                |
| --------------------- | ------------------ | ------------------------ |
| Kiến trúc & DB design | 1 system architect | 2 tuần                   |
| API + validation      | 2 Java dev         | 4 tuần                   |
| Web UI + auth         | 1 frontend dev     | 2 tuần                   |
| QA & testing          | 1 tester           | 2 tuần                   |
| Tổng cộng             | **4 người**        | **10 tuần (~2.5 tháng)** |

---

## 💰 5. **Chi phí ước tính**

| Hạng mục         | Chi phí                              |
| ---------------- | ------------------------------------ |
| Phần mềm         | $0 (toàn bộ open-source)             |
| Máy chủ dev/test | Dùng server nội bộ VATM              |
| Nhân công        | ~4 người x 2.5 tháng                 |
| **Tổng**         | **< $30,000 USD (nếu làm in-house)** |

---

## 🧠 6. **Lợi ích của Filing Service độc lập**

✅ Là nền tảng để:

* Sau này tích hợp SWIM Gateway → thành FF-ICE compliant.
* Kết nối Planning Service và Query Service.
* Là công cụ demo/training cho nhân viên VATM và hãng bay.

✅ Có thể dùng để:

* Nhận filing từ airlines nội địa (VN, VJ, QH…).
* Kiểm tra hợp lệ FIXM message.
* Lưu log phục vụ kiểm thử FF-ICE.

---

## 🔜 **Bước tiếp theo**

1. Tạo project Java Spring Boot + DB schema.
2. Tôi có thể **tạo skeleton code hoàn chỉnh** (REST API, DB, validator) cho bạn.
3. Bạn chỉ cần deploy lên máy test là có thể chạy demo Filing Service bản đầu.

---

Bạn có muốn tôi **tạo bộ mã khung (starter project)** cho Filing Service ngay bây giờ không?
→ Nó sẽ gồm:

* REST endpoint `/api/v1/filing/submit`
* FIXM schema validator
* PostgreSQL persistence
* Logging + sample JSON/XML test.



---


Tuyệt vời — bạn hỏi đúng vào “điểm chuyển pha” giữa **Filing Service nội bộ** và **FF-ICE/SWIM đầy đủ**.
Đây là **giai đoạn quan trọng nhất** để hệ thống của VATM trở thành “SWIM-compliant system”.
Tôi sẽ hướng dẫn bạn **toàn bộ 6 bước tích hợp Filing Service hiện tại vào SWIM Gateway**, bao gồm kỹ thuật, bảo mật và chuẩn ICAO.

---

# 🧭 MỤC TIÊU

> “Cho phép Filing Service đang chạy độc lập trở thành **một dịch vụ SWIM chính thức**, được các hãng hàng không hoặc ANSP khác gọi qua SWIM network.”

---

## 🏗️ 1. Tổng quan kiến trúc sau khi tích hợp

```
┌───────────────────────────────┐
│ Airlines / External ANSPs     │
│  (SWIM Clients)               │
└──────────────┬────────────────┘
               │  SOAP/REST + PKI
               ▼
┌────────────────────────────────────────────┐
│           SWIM Gateway (VATM)              │
│────────────────────────────────────────────│
│  ▪ Security Gateway (PKI, Auth)            │
│  ▪ Service Registry (publish/lookup)       │
│  ▪ Protocol Adapter (SOAP↔REST mapping)    │
│  ▪ Message Routing (queue/topic-based)     │
│  ▪ Logging, monitoring, transformation     │
└──────────────┬─────────────────────────────┘
               │ Internal REST (JSON)
               ▼
┌────────────────────────────────────────────┐
│          Filing Service (VATM)             │
│  ▪ Receives eFPL                          │
│  ▪ Validates + Stores + Acknowledges      │
│  ▪ No change in core logic                │
└────────────────────────────────────────────┘
```

---

## ⚙️ 2. Sáu bước triển khai chi tiết

---

### **STEP 1: Chuẩn bị SWIM Gateway (Middleware layer)**

**Mục tiêu:** Tạo “lớp trung gian” giữa mạng SWIM và dịch vụ nội bộ.

| Thành phần                | Mô tả                                                                                                                 |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **SWIM Gateway software** | Có thể chọn open-source như **Solace Event Broker**, **Apache ActiveMQ Artemis**, hoặc EUROCONTROL SWIM-TI reference. |
| **Deployment**            | Dựng trên 1 VM hoặc container riêng (Ubuntu).                                                                         |
| **Interfaces**            | Northbound (SOAP/SOAP+MTOM hoặc REST) – Southbound (REST JSON).                                                       |

> Gateway này chịu trách nhiệm **nhận/gửi tin nhắn SWIM** và **chuyển đổi** chúng sang REST nội bộ cho Filing Service.

---

### **STEP 2: Tích hợp bảo mật PKI & xác thực (SWIM Security)**

**Mục tiêu:** Đảm bảo mọi giao tiếp qua SWIM đạt chuẩn ICAO Annex 10 & SWIM-TI.

| Thành phần                          | Mô tả                                                              |
| ----------------------------------- | ------------------------------------------------------------------ |
| **PKI Certificates**                | Dùng chứng chỉ X.509 được CA phát hành cho VATM (ví dụ CAAV Root). |
| **TLS 1.3 + mutual authentication** | Cả hai bên (VATM và airline) đều xác thực certificate.             |
| **Message signature**               | XML digital signature (WS-Security hoặc JWS).                      |
| **Access control**                  | Chỉ client được cấp certificate mới truy cập Filing Service.       |

> Filing Service giữ nguyên, chỉ cần thêm reverse proxy có SSL termination (Nginx hoặc Gateway).

---

### **STEP 3: Đăng ký dịch vụ trong SWIM Registry**

**Mục tiêu:** Để các đối tác (airlines/ANSPs) “tìm thấy” Filing Service của VATM.

| Thành phần           | Mô tả                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------- |
| **Service metadata** | Tên: `VATM-FFICE-FilingService`<br> Version: 1.0 <br> Interface: SOAP/REST <br> Owner: VATM |
| **Publication**      | Ghi vào SWIM Registry bằng API hoặc file WSDL/WADL.                                         |
| **Discovery**        | Airlines dùng SWIM Client có thể lookup để biết endpoint, schema, và policy.                |

> Đây là bước “đưa Filing Service lên bản đồ SWIM toàn cầu”.

---

### **STEP 4: Tạo SWIM Adapter (Service Adapter Layer)**

**Mục tiêu:** Kết nối định dạng dữ liệu SWIM với định dạng nội bộ Filing Service.

| Tác vụ                    | Mô tả                                                      |
| ------------------------- | ---------------------------------------------------------- |
| **Protocol Mapping**      | SOAP/XML (từ SWIM) → REST/JSON (vào Filing Service).       |
| **Schema Transformation** | FIXM XML từ SWIM → JSON object nội bộ (Spring).            |
| **Error Mapping**         | HTTP 400/500 → SOAP Fault hoặc SWIM Error Code.            |
| **Response wrapping**     | Trả kết quả FilingService (ACK/NACK) về dạng SWIM message. |

Ví dụ mapping:

```
SWIM: SOAP Envelope + FIXM XML
↓
Adapter: parse XML, call FilingService REST
↓
Response: 200 OK {"status":"ACCEPTED"}
↓
Adapter: wrap lại thành SOAP response
```

> Bạn có thể viết adapter này bằng **Spring Integration**, **Apache Camel**, hoặc **Node-RED** (như Solace hướng dẫn).

---

### **STEP 5: Thiết lập Message Routing & Logging**

**Mục tiêu:** Quản lý việc gửi/nhận, retry, và lưu vết (audit).

| Thành phần           | Mô tả                                                   |
| -------------------- | ------------------------------------------------------- |
| **Message broker**   | ActiveMQ / RabbitMQ / Solace (topic `vatm/filing/eFPL`) |
| **Retry & queueing** | Giữ tin khi Filing Service tạm ngưng.                   |
| **Audit trail**      | Log message ID, timestamp, status, sender.              |
| **Monitoring**       | Dùng Prometheus + Grafana hoặc SWIM monitoring tool.    |

---

### **STEP 6: Kiểm thử & chứng nhận SWIM-compliance**

**Mục tiêu:** Đảm bảo dịch vụ đạt chuẩn EUROCONTROL SWIM-TI (Yellow Profile).

| Hạng mục kiểm thử         | Mô tả                                          |
| ------------------------- | ---------------------------------------------- |
| **Connectivity**          | Xác nhận TLS mutual authentication thành công. |
| **Service discovery**     | Dịch vụ xuất hiện đúng trong registry.         |
| **Message compliance**    | Validate FIXM 4.2.0 XML schema.                |
| **Performance**           | <2s/response; >100 concurrent filings.         |
| **Security**              | Penetration test, certificate renewal test.    |
| **Operational readiness** | Logging, monitoring, backup hoạt động.         |

> Khi đạt các bước này, Filing Service được xem là **SWIM Node chính thức**.

---

## 🧩 3. Cấu trúc triển khai mẫu (thực tế VATM)

| Layer            | Component                           | Technology                      | Ghi chú     |
| ---------------- | ----------------------------------- | ------------------------------- | ----------- |
| **Presentation** | SWIM Client (airline, foreign ANSP) | Eurocontrol SWIM SDK            |             |
| **Integration**  | SWIM Gateway                        | Solace / Apache Camel / SWIM-TI | SOAP ↔ REST |
| **Application**  | Filing Service                      | Spring Boot REST API            | Đã có       |
| **Database**     | PostgreSQL                          | Internal schema                 |             |
| **Security**     | PKI, TLS, WS-Security               | CAAV PKI infra                  |             |
| **Monitoring**   | Prometheus, Grafana                 | Open-source                     |             |

---

## 🧰 4. Bộ công cụ cần chuẩn bị

| Thành phần             | Dùng cho               | Gợi ý công cụ                           |
| ---------------------- | ---------------------- | --------------------------------------- |
| SWIM Gateway container | Message routing        | Solace Event Broker (Community Edition) |
| PKI Infrastructure     | Certificate generation | OpenSSL + test CA                       |
| Registry API           | Service discovery      | Custom REST API / XML registry file     |
| Message adapter        | Protocol mapping       | Apache Camel routes                     |
| FIXM validator         | Message validation     | JAXB + FIXM XSD 4.2.0                   |
| Monitoring             | Ops visibility         | Grafana + Prometheus                    |
| Logging                | Audit compliance       | ELK stack                               |

---

## 📅 5. Lộ trình triển khai thực tế (4–6 tháng)

| Giai đoạn          | Công việc chính                        | Kết quả                            |
| ------------------ | -------------------------------------- | ---------------------------------- |
| **P1 (Tháng 1–2)** | Cài SWIM Gateway + PKI                 | Có cổng SOAP/REST bảo mật          |
| **P2 (Tháng 3)**   | Tạo SWIM Adapter cho FilingService     | FilingService callable qua Gateway |
| **P3 (Tháng 4)**   | Triển khai Registry + Service metadata | Airlines có thể lookup endpoint    |
| **P4 (Tháng 5)**   | Kiểm thử & logging                     | Kết nối eFPL từ client test        |
| **P5 (Tháng 6)**   | Chứng nhận SWIM compliance             | Đạt chuẩn SWIM Yellow Profile      |

---

## ✅ 6. Sau khi tích hợp SWIM thành công

Filing Service của bạn sẽ có thể:

* Nhận eFPL **trực tiếp từ airlines** qua SWIM network (không qua VPN nội bộ).
* Trả ACK/NACK qua SWIM message queues.
* Gửi thông tin cho Planning Service hoặc các ANSP khác (ASEAN).
* Được liệt kê trong **SWIM Registry quốc gia hoặc khu vực**.

---

## 🎯 Tóm tắt 6 bước cần làm

| # | Bước                   | Mục tiêu            | Công cụ chính                |
| - | ---------------------- | ------------------- | ---------------------------- |
| 1 | Dựng SWIM Gateway      | Middleware layer    | Solace / SWIM-TI             |
| 2 | Áp dụng PKI + TLS      | Bảo mật chuẩn SWIM  | OpenSSL / CAAV PKI           |
| 3 | Đăng ký vào Registry   | Cho discovery       | XML Registry / API           |
| 4 | Tạo Adapter SOAP↔REST  | Mapping dữ liệu     | Apache Camel                 |
| 5 | Thêm routing + logging | Reliability & audit | ActiveMQ / ELK               |
| 6 | Kiểm thử compliance    | ICAO SWIM readiness | FIXM Validator / Test client |

---

Nếu bạn muốn, tôi có thể giúp bạn **vẽ sơ đồ kiến trúc chi tiết SWIM Gateway của VATM** và **mẫu cấu hình Solace hoặc Apache Camel** để nối Filing Service hiện tại vào SWIM.
Bạn có muốn tôi làm phần đó không?



