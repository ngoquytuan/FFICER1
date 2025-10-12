

Rất đúng khi bạn muốn so sánh phần **SWIM Router** với **Solace Event Broker** — vì đây là chỗ dễ nhầm nhất trong các triển khai thực tế.

### ⚙️ **1. Bản chất của SWIM Router**

SWIM Router (hoặc Message Router trong SWIM architecture) là **trung tâm định tuyến dịch vụ và thông điệp** trong hạ tầng SWIM.
Nó không chỉ "chuyển gói tin", mà còn:

* Xử lý **định danh dịch vụ** (service discovery)
* Thực hiện **message routing theo business logic**
* Hỗ trợ **format transformation** (ví dụ FIXM ↔ JSON ↔ XML)
* Quản lý **security và xác thực chứng thư PKI**
* Có thể tự xử lý **retry, QoS, logging, audit**

> Đây chính là “não” của SWIM — chứ không phải chỉ là message bus.

---

### ⚡ **2. Solace Event Broker là gì**

Solace Event Broker (thường được quảng cáo như PubSub+ Event Mesh) là **middleware message broker** chuyên cho **event-driven architecture**.
Nó làm tốt các chức năng sau:

* Message queue + pub/sub + topic routing
* Low-latency message transport
* Protocol bridging (MQTT, AMQP, JMS, REST, WebSocket)
* High availability clustering
* Hardware-accelerated throughput (ở dòng appliance)

💡 **Điểm khác:** Solace **không xử lý FIXM hay ICAO business logic**, mà chỉ truyền tin.
=> Bạn vẫn phải viết thêm lớp "SWIM logic" bên trên (router + service registry + validator).

---

### 🧩 **3. Về phần cứng chuyên dụng**

Đúng là **Solace ban đầu** có **hardware appliance** – các thiết bị phần cứng chuyên dụng (Solace PubSub+ 3560/3565) với card FPGA để tăng tốc độ xử lý.

* Throughput lên tới hàng triệu message/giây
* Dùng trong hệ thống **ATM (Air Traffic Management)** cũ của Mỹ và Canada giai đoạn 2010–2018
* Sau này họ chuyển sang bản **software-only** (Solace PubSub+ Software Broker), chạy trên Linux hoặc Docker

=> Từ năm 2019 trở đi, **Solace đã bỏ phần cứng chuyên dụng**, chỉ còn software edition.

---

### 🛰️ **4. So sánh SWIM Router vs Solace Broker**

| Tiêu chí                  | SWIM Router                                                   | Solace Event Broker                                          |
| ------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------ |
| **Mục tiêu chính**        | Routing & transformation theo logic hàng không (FIXM, FF-ICE) | Message distribution platform (pub/sub, topic-based routing) |
| **Tầng hoạt động**        | Service layer (SOA / SWIM-TI)                                 | Middleware layer (event mesh / messaging)                    |
| **Hiểu FIXM/XML**         | Có (qua parser và schema validator)                           | Không (chỉ pass-through)                                     |
| **Service Registry**      | Có (UDDI, WSDL metadata)                                      | Không có                                                     |
| **Security (PKI, X.509)** | Có, bắt buộc theo SWIM TI                                     | Chỉ hỗ trợ TLS, không bắt buộc PKI chain                     |
| **Transform logic**       | Có (SOAP ↔ REST ↔ FIXM XML)                                   | Không, chỉ payload-agnostic                                  |
| **High throughput**       | Trung bình (vài nghìn msg/s)                                  | Rất cao (hàng triệu msg/s)                                   |
| **Triển khai**            | Spring Boot / Camel / Java EE stack                           | Solace PubSub+ software (Docker/VM)                          |
| **Chi phí**               | Free nếu tự build (open source stack)                         | Có phí license hoặc cloud subscription                       |
| **Dùng trong ATM**        | EUROCONTROL, FAA SWIM, CAAS                                   | Dùng làm backbone cho FAA SWIM (chỉ transport layer)         |

---

### 🧠 **5. Thực tế tại FAA / EUROCONTROL**

* **FAA SWIM (Mỹ)**: dùng **Solace appliances** cho message backbone (Transport Tier),
  nhưng phần **SWIM Router và Gateway logic** là do Lockheed Martin và Harris xây dựng riêng.
* **EUROCONTROL SWIM-TI (Châu Âu)**: không dùng Solace, mà dùng **Apache ServiceMix / Camel / ActiveMQ**, 100% software.

=> Vì vậy Solace **chỉ chiếm một phần nhỏ** trong toàn SWIM — **nó không phải SWIM Router đầy đủ**.

---

### 💬 **Kết luận ngắn gọn**

* Solace = message bus tốc độ cao, **giống “đường cao tốc”**
* SWIM Router = middleware thông minh, **giống “cảnh sát giao thông”**
* Nếu VATM muốn SWIM chuẩn ICAO:

  * Có thể **dùng Solace làm backbone** (transport)
  * Nhưng **vẫn phải tự xây SWIM Router layer** để xử lý FIXM, security, registry, và business rules.

---

Nếu bạn muốn, tôi có thể vẽ cho bạn **kiến trúc kết hợp SWIM Router + Solace Broker**, kiểu FAA đang dùng: Solace ở tầng transport, Camel/Spring ở tầng routing — bạn muốn tôi làm sơ đồ đó không?


---

Rất hay là bạn hỏi kỹ về phần “phần cứng chuyên dụng / FPGA” — vì đây là phần mà không phải ai cũng hiểu rõ khi nhìn vào Solace hay các thiết bị mạng cao hiệu năng. Tôi sẽ giải thích chi tiết từ cơ bản → lý do dùng → lý do ít dùng sau này → ảnh hưởng đến bạn khi chỉ được “application software”.

---

## 1. FPGA / thiết bị chuyên dụng là gì?

* **FPGA = Field-Programmable Gate Array**: một loại chip mà bạn có thể lập trình lại (reconfigurable) để thực thi các thuật toán/hàm số logic ở tầng phần cứng, thay vì làm qua phần mềm chạy trên CPU.
* Trong thiết bị mạng/broker cao hiệu năng, người ta dùng FPGA (và đôi khi các network processor chuyên dụng) để **xử lý phần dữ liệu (data plane)**: định tuyến gói tin, lọc, chuyển tiếp, xử lý header, mã hóa/giải mã nhẹ…
* Ưu điểm của FPGA / thiết bị mạng chuyên dụng:

  1. **Độ trễ thấp (low latency)**: vì xử lý gần “kim loại”, không phải context-switch OS, interrupt, scheduler, v.v.
  2. **Tính nhất quán (“predictability”)**: không phụ thuộc load OS, garbage collection, hoặc các tác vụ background.
  3. **Thông lượng cao (high throughput)**: có thể xử lý hàng triệu gói tin mỗi giây với chi phí năng lượng thấp hơn so với CPU nếu thiết kế tốt.
  4. **Tách biệt control plane & data plane**: thiết bị chuyên dụng có thể giữ control plane (quản lý, cấu hình) ở CPU/OS, còn data plane ở FPGA – giúp tăng hiệu suất và ổn định.

Trong sản phẩm **Solace Appliance** (ví dụ PubSub+ 3560), Solace mô tả rõ họ “embedded data movement logic … in high-speed FPGAs and network processors, eliminating OS in the datapath” để “higher throughput and more predictable latency than other solutions.” ([Solace][1])

Ví dụ thông số kỹ thuật: với Non-persistent messaging, họ đạt kênh ~ 9–10 triệu messages/giây, latency ~ 18 µs … ([Solace][1])

---

## 2. Tại sao Solace từng dùng phần cứng + FPGA?

* Vì khi cần hệ thống messaging trong các môi trường tài chính, ngân hàng, giao dịch cao tần (low latency, throughput cực cao), phần mềm thuần (trên CPU thường) có thể không đáp ứng được.
* FPGA giúp đảm bảo rằng việc định tuyến, lọc, chuyển tiếp message không bị nghẽn do OS hoặc tác vụ khác.
* Đặc biệt trong các hệ thống ATM, SWIM, trao đổi dữ liệu lớn, Solace muốn cung cấp chất lượng dịch vụ (QoS), tính ổn định cao, không mất message, mà vẫn có throughput rất lớn.

---

## 3. Vì sao bây giờ họ chủ yếu “phần mềm” (software broker) và ít dùng phần cứng?

Một số lý do:

1. **Chi phí phát triển & bảo trì cao**

   * FPGA / thiết bị chuyên dụng yêu cầu firmware, lập trình phần cứng – mỗi thay đổi logic mới, sửa lỗi firmware rất phức tạp.
   * Bảo trì phần cứng, linh kiện, nâng cấp, thay thế, kiểm định – đắt đỏ so với phần mềm trên máy chủ chung.

2. **CPU / phần mềm ngày càng mạnh hơn**

   * CPU hiện đại, multi-core, vector instructions (SIMD), NICs (Network Interface Cards) hỗ trợ offload (RDMA, DPDK, SmartNIC) giúp hiệu năng phần mềm tăng rất mạnh.
   * Với nhiều trường hợp, hệ thống phần mềm (kết hợp tuning, tối ưu cấu hình mạng) đã đáp ứng đủ nhu cầu của hầu hết ứng dụng.

3. **Tính linh hoạt & triển khai dễ hơn**

   * Phần mềm dễ deploy trên VM, container, cloud, hybrid – không phụ thuộc vào hạ tầng vật lý chuyên dụng.
   * Khi bạn làm việc với khách hàng, đối tác, bạn dễ phân phối phần mềm hơn là giao/chuyển thiết bị mạng.
   * Cập nhật, nâng cấp phần mềm nhanh hơn so với cập nhật firmware phần cứng.

4. **Phần mềm đã “đủ nhanh” cho nhiều ứng dụng**

   * Với nhiều workload, hệ thống phần mềm (bằng cách tuning mạng, OS, cấu hình TCP, NIC offload…) đã đạt hiệu năng rất cao. Solace cũng đưa ra các hướng tuning host TCP stack để tối ưu performance cho software broker. ([Solace][2])
   * Phần mềm broker có thể scale-out (nhiều node) nếu cần — chứ không nhất thiết phải đẩy một thiết bị duy nhất lên cực điểm.

5. **Thị trường & nhu cầu đa dạng**

   * Nhiều tổ chức không cần throughput cực kỳ cao, mà cần khả năng triển khai nhanh, chi phí thấp → phần mềm là lựa chọn hợp lý.
   * Solace vẫn giữ “appliance” option cho những khách hàng siêu cao hiệu năng, nhưng đối với hầu hết khách hàng, software edition là đủ.

---

## 4. Khi họ gửi cho bạn “application software” là sao? (ý của bạn: họ chỉ gửi phần mềm broker?)

* Khi bạn làm việc với Solace ngày nay, khả năng cao họ sẽ cung cấp **software event broker** (bản mềm) hoặc license để bạn chạy trên máy chủ/VM/DC của bạn — bạn không nhận được thiết bị phần cứng vật lý.
* Điều này thường là vì bạn (hoặc tổ chức bạn) không có yêu cầu throughput siêu cao như các ngân hàng giao dịch cao tần, hoặc họ nhìn thấy rằng phần mềmbroker là đủ cho trường hợp sử dụng của bạn.
* Bạn vẫn có thể hưởng lợi nếu hệ thống phần mềm được cấu hình, tuning tốt (OS, network, tài nguyên phần cứng), để đạt hiệu năng tốt.

---

## 5. Đánh giá “phần mềm đã quá nhanh rồi” — có đúng không?

* “Đủ nhanh” – đúng — nhưng không phải trong mọi trường hợp.
* Với nhiều ứng dụng trung bình hoặc lớn, phần mềm broker (với cấu hình tốt) có thể đạt hàng trăm ngàn, triệu message/s, latency thấp (vài chục µs tới trăm µs).
* Tuy nhiên, nếu bạn đè hệ thống messaging vào giới hạn cực cao (giao dịch tài chính, dữ liệu thời gian thực cực nhạy), phần mềm thuần có thể gặp vấn đề: CPU quá tải, interrupt overhead, NIC bottleneck, bất ổn latency.
* Trong những tình huống đòi hiệu năng cực đại, Solace vẫn giữ tùy chọn appliance để phục vụ “tour de force” về throughput/latency.

---

Nếu muốn, tôi có thể so sánh hiệu năng phần mềm vs phần cứng của Solace trong các case study thực tế, và xem đối với dự án của bạn cái nào đủ – bạn muốn tôi làm điều đó không?

[1]: https://solace.com/wp-content/uploads/2019/04/Solace-3560-Datasheet.pdf?utm_source=chatgpt.com "Solace PubSub+ 3560 Appliance"
[2]: https://solace.com/blog/optimizing-performance-pubsub-event-broker/?utm_source=chatgpt.com "Optimizing the Performance of your Solace Event Broker"


---

Câu hỏi này **rất sắc** — và đúng là điều nhiều người mới dùng Solace cũng thắc mắc:

> “Nếu Solace PubSub+ chạy Docker miễn phí, không cần license key, vậy họ kiếm tiền kiểu gì và giới hạn ở đâu?”

Hãy tôi giải thích cặn kẽ, theo từng tầng kỹ thuật và thương mại.

---

## 🧩 1. Solace có **3 bản khác nhau**

| Phiên bản                      | Hình thức triển khai      | License                 | Mục đích                                    |
| ------------------------------ | ------------------------- | ----------------------- | ------------------------------------------- |
| **PubSub+ Community Edition**  | Docker / VM (self-hosted) | **Miễn phí 100%**       | Học tập, PoC, test, low-volume apps         |
| **PubSub+ Enterprise Edition** | Docker / VM / Bare metal  | **Cần license key**     | Production lớn, clustering, HA, replication |
| **PubSub+ Cloud**              | SaaS (họ host)            | **Trả tiền theo usage** | Cloud-native, global mesh, managed service  |

👉 **Bạn đang dùng bản “Community Edition”**, nên nó không yêu cầu license key — nhưng bị giới hạn về **hiệu năng, cluster, tính năng enterprise**.

---

## ⚙️ 2. Các **giới hạn kỹ thuật** (ẩn trong cấu hình Docker image)

Khi bạn kéo image về, nó là cùng một binary với bản enterprise — chỉ khác ở **cấu hình runtime**: container khi khởi động **tự kiểm tra license tier**, và áp dụng các “feature flags”.

### 🔒 Giới hạn mặc định của bản free (Community Edition):

| Loại giới hạn                         | Mức giới hạn (approx)                      | Ghi chú                                     |
| ------------------------------------- | ------------------------------------------ | ------------------------------------------- |
| **Max connections (client sessions)** | 100                                        | Tổng số publisher + subscriber              |
| **Max message rate**                  | ~10.000 msg/s                              | Cứng giới hạn trong core engine             |
| **Max throughput (payload)**          | ~30 MB/s                                   | Giới hạn logic chứ không phải băng thông OS |
| **Max queues / topics**               | 100                                        | Không tạo thêm được sau ngưỡng              |
| **Max VPNs**                          | 1 (default: defaultVPN)                    | Không tạo VPN khác                          |
| **Max message spool (persistence)**   | 8 GB                                       | Giới hạn dung lượng lưu message             |
| **HA replication**                    | ❌ Không hỗ trợ                             | Chỉ có 1 node đơn lẻ                        |
| **DMR (mesh routing)**                | ❌ Không có                                 | Không thể tạo event mesh multi-node         |
| **Monitoring (CLI, API)**             | ✅ Cơ bản, không có advanced stats          |                                             |
| **Cluster links / multi-node**        | ❌ Bị tắt                                   | Không làm HA cluster                        |
| **TLS connections**                   | ✅ Có, nhưng tối đa số lượng giới hạn       |                                             |
| **Replay / persistent store**         | ✅ Có, nhưng lưu trữ nhỏ và không scale-out |                                             |

📦 Docker container khi start sẽ in ra trong log như:

```
INFO: Starting Solace PubSub+ (Community Edition)
INFO: License type: Community Edition
INFO: Max Connections: 100, Max Queues: 100, Max Spool: 8GB
```

→ Nghĩa là **nó không cần key**, nhưng engine nội bộ có “license descriptor” hard-coded.

---

## 🧠 3. Họ giới hạn bằng cách nào?

Solace broker khi khởi động đọc:

* **license descriptor** (gắn trong binary hoặc file `.license` nếu có),
* **runtime profile** (ví dụ: `community`, `enterprise`, `evaluation`).

Nếu không có file `.license` hợp lệ → nó tự rơi về **community profile**.

Cấu hình này quy định các ngưỡng (`max_connections`, `max_spool_usage`, `max_topics`, v.v.)
Các ngưỡng này **được compile sẵn** trong code C++ core engine của họ, không thay đổi được bằng config.

🔍 Bạn có thể xem bằng lệnh CLI trong container:

```
show system limits
show message-spool detail
show client-connection-summary
```

Sẽ thấy các giá trị “Maximum allowed …” tương ứng.

---

## 💰 4. Họ kiếm tiền thế nào nếu bản Docker free?

Họ áp dụng mô hình **freemium / tiered licensing**:

1. **Community Edition (free)** → cho phép bạn làm PoC, test, demo, hoặc deploy nhỏ.
2. Khi bạn cần:

   * nhiều connection hơn 100,
   * lưu message nhiều hơn 8GB,
   * clustering, HA, DR,
   * quản trị central (PubSub+ Broker Manager),
   * hoặc support chính thức,
     → bạn phải nâng lên **Enterprise Edition**, mua license key (thường từ vài ngàn đến vài chục ngàn USD/năm tùy throughput).
3. Nếu bạn không muốn tự vận hành → họ có **PubSub+ Cloud**, trả tiền theo usage (số message, throughput, connections).

---

## 🚀 5. Về mặt kỹ thuật, bạn vẫn có thể dùng bản miễn phí rất tốt

Community Edition đủ để:

* Dựng **SWIM prototype hoặc FF-ICE thử nghiệm**
* Test mô hình **publish/subscribe hoặc message routing**
* Tích hợp **Spring Boot, MQTT, JMS, REST**
* Làm POC hoặc mô phỏng event mesh trong 1 node

Chỉ khi bạn:

* cần liên kết giữa **Hà Nội ↔ TPHCM ↔ Đà Nẵng (nhiều node)**,
* cần **HA clustering**,
* cần **replication, replay, DR**,
  → mới phải dùng bản Enterprise có license key.

---

## 🧭 6. Tóm lại cho bạn:

* ✅ Bạn **không cần license key** vì đang chạy bản Community.
* ⚙️ Solace vẫn **giới hạn kỹ thuật** trong core broker — không vượt ngưỡng được.
* 💼 Họ **thu tiền ở bản Enterprise/Cloud**, nơi mở khóa giới hạn và có support SLA.
* 🔐 Không có cách “hack” mở giới hạn — vì bị compile sẵn, container restart sẽ kiểm tra lại.
* 🚀 Cho mục đích **dự án thử nghiệm SWIM của bạn**, bản Community là **đủ dùng**.

---

Nếu bạn muốn, tôi có thể chỉ bạn lệnh CLI để kiểm tra chính xác các giới hạn hiện tại trong container Solace của bạn — bạn có muốn tôi hướng dẫn không?


---

Câu hỏi của bạn **rất sâu và chính xác** 👏 — và đúng, nhiều người khi mới dùng Solace hoặc các message broker “event-driven” cũng hay thắc mắc điều này:

> “Nếu Solace chỉ là router (chuyển tiếp message), thì sao lại có giới hạn lưu trữ 8GB message spool? Router thì có lưu tin nhắn đâu mà giới hạn?”

Để hiểu rõ, ta cần tách **hai chế độ hoạt động** của Solace (và của các hệ thống message broker nói chung):

---

## ⚙️ 1. Hai kiểu truyền message của Solace

### **(1) Non-persistent messaging (transient / in-memory)**

* Broker chỉ **đóng vai trò router**,
  message được gửi đi ngay lập tức tới subscribers đang online.
* Nếu subscriber chưa kết nối → message **bị bỏ qua (drop)**.
* Không có lưu trữ, không có retry, không có spool.
* Rất nhanh, latency vài micro-giây.
* Dùng cho data stream “real-time”, telemetry, market data…

🧩 Khi bạn dùng Solace như “SWIM Router”, đây chính là **mode Non-persistent**.

---

### **(2) Persistent messaging (store-and-forward)**

* Broker **có lưu tạm (spool)** các message trong ổ đĩa,
  để đảm bảo **giao hàng chắc chắn (Guaranteed Delivery)**.
* Dành cho queue hoặc topic có subscribers không online ngay:
  message được **ghi vào ổ đĩa (spool)** rồi gửi lại khi client quay lại.
* Có thể thiết lập độ bền (durable queue), confirm, replay…

🗄️ Đây là nơi mà **“8GB message spool”** của bản miễn phí được áp dụng.
Nó không phải là cache, mà là **vùng lưu trữ an toàn cho persistent messages**.

---

## 📦 2. Vậy “message spool 8GB” là cái gì?

Đó là **vùng đĩa mà broker dùng để lưu các message “chưa giao xong”**.

Khi bạn tạo queue hoặc topic với `persistent delivery`, Solace sẽ:

1. Ghi message vào ổ đĩa spool (write-ahead log hoặc journal file).
2. Khi subscriber nhận và xác nhận (`ack`), message mới bị xóa khỏi spool.
3. Nếu subscriber offline lâu → broker vẫn giữ message trong spool.

📉 Trong bản **Community Edition**, Solace chỉ cho phép tổng dung lượng spool (message lưu trữ + metadata) = **8GB**.
Nếu vượt, broker từ chối message mới hoặc xóa message cũ tùy policy.

---

## 🛰️ 3. Cách Solace “router” và “store” cùng tồn tại

Solace thực chất là **message broker đa chế độ**:

* Khi bạn publish message, broker định tuyến theo topic map.
* Với mỗi subscriber:

  * Nếu subscriber online và non-persistent → gửi trực tiếp (no storage).
  * Nếu subscriber yêu cầu persistent delivery → broker ghi message vào spool, đảm bảo reliability.

Ví dụ bạn có:

```text
Publisher → Topic: /flightplan/vn123
Subscriber A (real-time monitor) : non-persistent
Subscriber B (analysis system)   : persistent queue
```

👉 Broker sẽ:

* Gửi message ngay cho A (non-persistent),
* Ghi message vào spool cho B (persistent),
* Khi B online hoặc ack → xóa khỏi spool.

---

## 🔐 4. Tại sao SWIM/FF-ICE có thể cần “persistent”?

Trong mạng SWIM (hoặc FF-ICE):

* Một số dịch vụ cần **Guaranteed Delivery** (ví dụ Filing Service, NOTAM feed, Flight Plan confirmations…).
* Nếu phía nhận (client hoặc gateway) bị down, message không thể mất.
  → Broker cần có vùng lưu trữ (spool) để đảm bảo không mất tin.

🧭 Vì vậy, Solace cho phép bạn:

* Chạy **non-persistent mode** cho luồng real-time (VD: surveillance, trajectory updates).
* Chạy **persistent mode** cho luồng critical (VD: filing status, coordination messages).

---

## 💾 5. Với bản bạn đang chạy (Community Docker)

Bạn có thể kiểm tra bằng CLI:

```
show message-spool detail
```

Kết quả ví dụ:

```
Message Spool Type : Disk
Max Spool Usage    : 8192 MB
Current Usage      : 120 MB
Messages Queued    : 152
Messages Delivered : 10234
```

Nghĩa là tổng vùng lưu trữ cho tất cả queues/topic durable = 8GB.

Nếu bạn chỉ dùng broker như router thuần (non-persistent) →
✔ Bạn **gần như không đụng tới 8GB spool đó**,
✔ Broker hoạt động như **event router / SWIM Router** đúng nghĩa.

---

## ⚡ 6. Tại sao bản thương mại có spool lớn hơn?

Ở bản Enterprise, người dùng có thể:

* Tăng spool lên hàng trăm GB hoặc vài TB,
* Dùng **replicated spool** (HA cluster),
* Cấu hình replay (message re-delivery),
* Giữ lịch sử lâu dài (audit trail),
* Phục vụ message “replay” cho các client mới.

---

## ✅ **Tóm lại cho bạn:**

| Chức năng                      | Có trong Solace?           | Lưu spool 8GB ảnh hưởng gì?                |
| ------------------------------ | -------------------------- | ------------------------------------------ |
| Chuyển tiếp real-time (router) | ✅ Có (non-persistent)      | Không cần spool, latency cực thấp          |
| Lưu trữ đảm bảo (guaranteed)   | ✅ Có (persistent)          | Giới hạn 8GB spool trong bản free          |
| Đảm bảo không mất message      | ✅ Có (nếu dùng persistent) | Khi vượt 8GB → ngưng nhận message mới      |
| Dùng cho SWIM Router prototype | ✅ Rất phù hợp              | Dùng non-persistent, spool không ảnh hưởng |

---

Nếu bạn muốn, tôi có thể cho bạn ví dụ **cấu hình Solace chạy SWIM router mode** (non-persistent, throughput tối đa, spool = 0%) — để bạn dùng đúng cách cho dự án của mình. Bạn muốn tôi gửi ví dụ đó không?
