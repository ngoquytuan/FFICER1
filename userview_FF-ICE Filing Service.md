Rất hay 👨‍✈️ — đúng trọng tâm câu hỏi của một **người dùng vận hành (Airline Dispatcher / Filer)** trong hệ thống FF-ICE Filing Service.
Ta tạm **rời vai nhà phát triển**, và xem Filing Service hoạt động **từ phía người sử dụng**.
Giờ bạn có tệp `test_fixm.xml` (điện văn eFPL) rồi — hãy xem hành trình của nó **bên trong hệ thống** theo đúng luồng ICAO chuẩn.

---

## 🧭 **1️⃣ Vai trò người dùng và mục tiêu**

Bạn là **người nộp kế hoạch bay điện tử (eFPL)** — ví dụ: hãng hàng không, trung tâm OCC, hoặc một ATC local tester.

Nhiệm vụ của bạn:

> Gửi điện văn FIXM (XML) chứa kế hoạch bay tới Filing Service,
> nhận về phản hồi **ACK (ACCEPTED)** hoặc **NACK (REJECTED)**.

---

## ⚙️ **2️⃣ Cách nạp điện văn test_fixm.xml**

Giả sử file XML của bạn nằm trong thư mục:

```
~/filing-service/test_fixm.xml
```

Bạn gửi lên hệ thống bằng lệnh:

```bash
curl -X POST http://localhost:8080/api/v1/filing/submit \
  -H "Content-Type: application/json" \
  -d @<(cat <<EOF
{
  "flightNumber": "VN123",
  "departure": "VVNB",
  "arrival": "VVTS",
  "etd": "2025-10-09T02:00:00",
  "eta": "2025-10-09T04:00:00",
  "fixmXml": "$(tr -d '\n' < test_fixm.xml)"
}
EOF
)
```

> Hệ thống không đọc file trực tiếp, mà bạn **nạp nội dung XML** vào trường `"fixmXml"`.
> (Trong thực tế, SWIM Gateway sẽ gửi XML gốc qua SOAP, nhưng trong bản stand-alone này ta dùng REST/JSON cho dễ test.)

---

## 🔁 **3️⃣ Hệ thống xử lý điện văn theo 7 bước nội bộ**

### 🧩 Bước 1. **Tiếp nhận và ghi log ban đầu**

* API `/filing/submit` nhận payload.
* Lưu **raw XML** và metadata (flightNumber, departure, arrival, timestamp) vào bảng `validation_log` với trạng thái **PENDING**.
* Sinh GUFI (Global Unique Flight Identifier).

📘 *Ví dụ trong DB:*

```
gufi=7b6d-...-aa12
raw_request=<FlightPlan>...</FlightPlan>
received_at=2025-10-09T01:00:02Z
status=PENDING
```

---

### 🧠 Bước 2. **Kiểm tra hợp lệ FIXM (XML Schema Validation)**

* Module `FixmValidator` tải file `fixm.xsd` từ `src/main/resources/schema/fixm_4_2_0/`.
* Parser kiểm tra cấu trúc XML: tag mở/đóng, namespace, kiểu dữ liệu, định dạng thời gian, v.v.

🧩 Nếu XML **đúng schema** → pass ✅
🧩 Nếu **sai tag hoặc namespace** → **NACK**, ghi lỗi “Invalid FIXM XML”.

📘 *Ví dụ log:*

```
fixm_is_valid=false
validation_message="Missing element 'departure'"
```

---

### 🧮 Bước 3. **Kiểm tra nghiệp vụ (Rule Engine)**

Chạy lần lượt các quy tắc:

1. Departure và Arrival có đúng định dạng ICAO `[A-Z]{4}` không?
2. Flight number hợp lệ? (VD: `VN123`, `VJ5120`)
3. ETD < ETA?
4. Có trùng GUFI không?

Nếu bất kỳ rule nào sai → REJECTED, ghi nguyên nhân.

📘 *Ví dụ log:*

```
business_is_valid=false
validation_message="ETD must be before ETA"
```

---

### 📂 Bước 4. **Lưu dữ liệu hợp lệ vào CSDL**

Nếu vượt qua cả FIXM + Rule Engine:

* Sinh GUFI (UUID)
* Tạo bản ghi mới trong bảng `flight_plans`
* Ghi status = “FILED”

📘 *Ví dụ trong DB:*

```sql
SELECT gufi, flight_number, status FROM flight_plans;
-- 7b6d... | VN123 | FILED
```

---

### 📨 Bước 5. **Sinh thông điệp phản hồi (ACK/NACK)**

Tạo bản tin phản hồi theo cấu trúc ICAO-like (simplified JSON hoặc XML).

**ACK (Accepted):**

```json
{
  "gufi": "7b6d-...-aa12",
  "status": "ACCEPTED",
  "message": "Flight plan filed successfully",
  "timestamp": "2025-10-09T01:00:03Z"
}
```

**NACK (Rejected):**

```json
{
  "status": "REJECTED",
  "message": "Invalid FIXM XML: Missing element 'arrival'",
  "timestamp": "2025-10-09T01:00:02Z"
}
```

---

### 📜 Bước 6. **Ghi log kết quả**

Cập nhật bảng `validation_log`:

```
validation_status=ACCEPTED
validation_message="Flight Plan accepted"
fixm_is_valid=true
business_is_valid=true
```

---

### 📡 Bước 7. **Trả ACK/NACK cho người dùng**

Phản hồi HTTP 200 (OK) hoặc 400 (Bad Request).
Đây là bản tin **tương đương “Filing Acknowledgement” trong SWIM**.

Trên Postman hoặc curl bạn thấy ngay kết quả:

```
{
  "gufi": "7b6d-...-aa12",
  "status": "ACCEPTED"
}
```

---

## 🧭 **4️⃣ Trạng thái điện văn sau khi xử lý**

| Trạng thái    | Ý nghĩa                                           |
| ------------- | ------------------------------------------------- |
| **FILED**     | Kế hoạch bay hợp lệ, đã lưu vào hệ thống.         |
| **REJECTED**  | Kế hoạch bay sai (cấu trúc hoặc nghiệp vụ).       |
| **PENDING**   | Mới nhận, đang chờ xử lý (trường hợp SWIM async). |
| **CANCELLED** | (Dành cho giai đoạn sau, khi nộp FPL huỷ).        |

Bạn có thể xem bằng lệnh:

```bash
psql -U filing_user -d filing_db -c "SELECT gufi, flight_number, status FROM flight_plans;"
```

---

## 🔍 **5️⃣ Truy xuất lại kế hoạch bay**

Sau khi filing thành công:

```bash
curl http://localhost:8080/api/v1/filing/7b6d-...-aa12
```

→ Nhận lại toàn bộ nội dung FPL đã nộp.

---

## 📊 **6️⃣ Xem lịch sử xử lý (Validation Log)**

```bash
curl http://localhost:8080/api/v1/filing/logs
```

→ Kết quả:

```json
[
  {
    "gufi": "7b6d-...-aa12",
    "validationStatus": "ACCEPTED",
    "validationMessage": "Flight plan filed successfully",
    "fixmIsValid": true,
    "businessIsValid": true,
    "receivedAt": "2025-10-09T01:00:02Z"
  }
]
```

---

## 🧩 **7️⃣ Trong SWIM thực tế (sau này)**

Ở giai đoạn FF-ICE/SWIM:

* Thay vì gửi JSON, bạn sẽ gửi **SOAP XML điện văn eFPL**.
* Filing Service của bạn trở thành **SOAP endpoint** (ví dụ: `FilingService.wsdl`).
* Hệ thống sẽ tự động sinh **ACK/NACK XML** trả về qua SWIM Gateway.
* Cả hai bản tin sẽ có **X.509 signature + timestamp** để bảo đảm tính toàn vẹn.

---

## ✅ **Tóm tắt từ góc người dùng:**

| Giai đoạn           | Mô tả                            | Bạn thấy gì                      |
| ------------------- | -------------------------------- | -------------------------------- |
| 1️⃣ Gửi điện văn    | Nạp `test_fixm.xml` qua REST API | Postman/curl gửi JSON            |
| 2️⃣ FIXM Validation | Hệ thống kiểm tra XML            | Lỗi cấu trúc nếu sai             |
| 3️⃣ Rule Engine     | Kiểm tra nghiệp vụ               | Cảnh báo logic nếu sai           |
| 4️⃣ Ghi DB          | Lưu eFPL + GUFI                  | Lưu trạng thái FILED             |
| 5️⃣ ACK/NACK        | Trả phản hồi                     | JSON thông báo ACCEPTED/REJECTED |
| 6️⃣ Log             | Lưu validation log               | Có thể xem lại lịch sử           |

---

📘 **Tóm lại:**

> ✅ Bạn có thể nạp `test_fixm.xml` ngay bây giờ.
> Hệ thống sẽ đọc, validate, kiểm tra logic, lưu vào DB và trả về **ACK/NACK** — đúng như ICAO FF-ICE yêu cầu cho Filing Service.

---

Bạn có muốn tôi minh họa **toàn bộ quy trình xử lý test_fixm.xml** này bằng một **sequence diagram (sơ đồ tuần tự)** để bạn dùng trong tài liệu vận hành không?
