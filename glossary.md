# Glossary — DDD & ADLP (Ubiquitous Language, Việt/Anh)

Tài liệu này là glossary “sống” cho handbook. Mục tiêu: giữ cho team nói cùng một ngôn ngữ, và tránh anti-pattern “một từ nhiều nghĩa không ai biết”.

Nguồn thiết kế ADLP mới nhất: `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`.

> **NOTE**  
> Glossary không cần hoàn hảo ngay. Nhưng glossary phải có **owner context** và phải được cập nhật khi domain đổi (Continuous Discovery).

---

## 1) Cách dùng glossary (quy tắc vận hành)

1) Mỗi term phải có **định nghĩa 1–2 câu** theo ngôn ngữ business.  
2) Mỗi term phải có “ranh giới”: nó *không* bao gồm cái gì.  
3) Nếu cùng một từ nhưng khác nghĩa ở context khác → tạo **2 entries** và ghi owner context.  
4) Nếu term “đắt tiền” (payout, accept, dispute, lock TTL) → phải có ví dụ đúng/sai.  
5) Khi thay đổi nghĩa/luật → ghi policy/version (nếu có).

---

## 2) Thuật ngữ DDD cốt lõi (Việt/Anh)

| Term (EN) | Tiếng Việt (gợi ý) | Định nghĩa 1–2 câu | Ví dụ đúng | Ví dụ sai | Owner context |
|---|---|---|---|---|---|
| Domain | Miền nghiệp vụ | Phần “thực tại business” mà hệ thống mô hình hóa: quy tắc, quyết định, ràng buộc. | “Lock TTL là rule nghiệp vụ” | “Domain là database schema” | N/A |
| Ubiquitous Language | Ngôn ngữ chung | Bộ từ vựng + định nghĩa mà business và dev dùng chung trong một context. | “Accepted nghĩa là gì?” | “Accepted ai hiểu sao cũng được” | N/A |
| Bounded Context | Ngữ cảnh giới hạn | Ranh giới nơi một mô hình và một ngôn ngữ có nghĩa nhất quán. | Batch trong Assignment ≠ Batch trong Wallet | “Một model dùng cho toàn hệ thống” | N/A |
| Context Map | Bản đồ ngữ cảnh | Bản đồ quan hệ giữa bounded contexts (upstream/downstream, patterns). | ACL cho payment provider | “Context map = sơ đồ microservices” | N/A |
| Aggregate | Tổng thể nhất quán | Ranh giới nhất quán nơi invariants được bảo vệ trong một transaction. | Batch aggregate bảo vệ one active assignment | “Aggregate = bảng có nhiều cột” | N/A |
| Invariant | Bất biến nghiệp vụ | Rule phải luôn đúng, đặc biệt dưới concurrency/retry. | “Không double payout” | “Rule nằm rải rác trong controller” | N/A |
| Domain Event | Sự kiện miền | “Điều đã xảy ra” có ý nghĩa nghiệp vụ trong một context. | BatchAccepted | “updated” | N/A |
| Integration Event | Sự kiện tích hợp | Contract/published language để context khác tiêu thụ; có schema versioning. | BatchAccepted v1.0.0 | Dump full row | N/A |
| Saga/Process Manager | Điều phối dài hạn | Cơ chế điều phối workflow qua nhiều contexts với state + compensation. | Payout workflow nhiều bước | “Chỉ cần retry là đủ” | N/A |
| CQRS | Tách ghi/đọc | Tách mô hình ghi (commands) và đọc (query/read model) để giảm coupling. | Read model cho Ops dashboard | “CQRS = microservices” | N/A |
| Event Sourcing | Lưu sự kiện | Lưu state bằng chuỗi events để replay/audit. | Audit toàn bộ lịch sử Quality | “Event sourcing = mọi thứ async” | N/A |
| Outbox Pattern | Hộp thư giao dịch | Ghi event cùng transaction, publish sau để tránh mất event. | Outbox cho BatchAccepted | “Publish trước commit” | N/A |
| Dead Letter Queue (DLQ) | Hàng đợi lỗi | Nơi chứa message fail nhiều lần để xử lý lại có chủ đích. | DLQ cho Export jobs | “Bỏ qua lỗi” | N/A |
| Idempotency | Tính lặp an toàn | Lặp lại command/event không tạo hiệu ứng phụ thêm. | “Consume BatchAccepted 2 lần vẫn credit 1 lần” | “Retry gây double payout” | N/A |
| Causation ID | ID nguyên nhân | ID của command/event gây ra event hiện tại (trace causal chain). | causation_id = command_id | “Không trace được nguyên nhân” | N/A |
| ACL (Anti-Corruption Layer) | Lớp chống ô nhiễm | Lớp dịch/che giữa domain và external/legacy để domain không bị kéo méo. | Translate payment statuses | “Import SDK và dùng trực tiếp trong domain” | N/A |
| SLO/SLI | Mục tiêu/Chỉ số dịch vụ | Thước đo vận hành để ra quyết định kiến trúc và alert đúng. | Assignment p95 < 200ms | “Hệ thống phải nhanh” | N/A |

---

## 3) Glossary ADLP (theo bounded contexts)

> **NOTE**  
> Owner context ở đây theo Strategic Design v0.2. Nếu sau này refactor context map, hãy cập nhật cột owner.

### 3.1 Thuật ngữ xuyên suốt workflow (ADLP)

| Term | Định nghĩa 1–2 câu | Ví dụ đúng | Ví dụ sai | Owner context |
|---|---|---|---|---|
| Dataset | Tập dữ liệu mà khách hàng muốn gán nhãn; có version khi export. | “Dataset v3 đã freeze” | “Dataset = một batch” | Dataset Export |
| Data Item | Đơn vị dữ liệu thô đầu vào (audio file, image, text). | “DataItemUploaded” | “DataItem = segment” | Data Ingestion |
| Segment | Một lát cắt/đơn vị nhỏ để label (audio segment, image region…). | “segment_id thuộc một data item” | “segment = batch” | Prelabeling/Labeling |
| Batch | Đơn vị giao việc cho labeler, gồm nhiều segments; có lock/TTL. | “BatchAssigned cho labeler A” | “Batch = dataset” | Task Assignment |
| Task | Tên gọi chung; chỉ dùng khi đã chốt nghĩa theo context (thường tránh). | “SegmentTask trong Labeling” | “Task = mọi thứ” | Varies |
| Submission | Snapshot kết quả labeler nộp để Quality chấm; phải audit được. | “TranscriptSubmitted tạo submission snapshot” | “Submission chỉ là status flag” | Labeling |
| Quality Score | Điểm chất lượng tính theo policy; có policy_version. | “QualityScore v2.1 theo tier” | “Score là số rời rạc không policy” | Quality Assurance |
| Accepted | Quyết định nghiệp vụ: batch đạt chuẩn theo policy/tier. | “BatchAccepted bởi reviewer” | “Accepted nghĩa là ‘đã xử lý xong’” | Quality Assurance |
| Rework | Quyết định yêu cầu làm lại (có lý do) sau quality. | “RequestRework với reason” | “Rework là bug fix” | Quality Assurance |
| Export | Quá trình đóng gói dataset version + cấp quyền tải; có audit. | “ExportJobCreated” | “Export = zip file” | Dataset Export |
| Earning | Khoản thu nhập labeler được ghi nhận; phải idempotent. | “EarningCredited exactly once” | “Earning tính lại tùy hứng” | Wallet & Payment |

### 3.2 Một số thuật ngữ “đắt tiền” cần chốt nghĩa sớm

| Term | Định nghĩa 1–2 câu | Ví dụ đúng | Ví dụ sai | Owner context |
|---|---|---|---|---|
| Lock TTL | Thời hạn lock batch để tránh tranh chấp; hết TTL có policy unlock/reassign. | “lock_until = now + TTL” | “TTL chỉ là timeout UI” | Task Assignment |
| Premium 48h | Tier cam kết lead time end-to-end; driver cho SLO/NFR và capacity. | “alert khi lead time > 36h” | “48h chỉ là marketing” | Varies (cross-context) |
| Policy Version | Phiên bản policy (quality/payout/routing) dùng để audit và tiến hóa rule. | “policy_version ghi vào event” | “policy thay đổi không version” | Quality/Assignment/Wallet |
| Correlation ID | ID để trace end-to-end workflow qua contexts. | “correlation_id trong event envelope” | “debug bằng grep log không trace” | N/A (cross-cutting) |

---

## 4) Cách mở rộng glossary (template)

Copy/paste bảng sau khi thêm term mới:

| Term | Định nghĩa 1–2 câu | Ví dụ đúng | Ví dụ sai | Owner context |
|---|---|---|---|---|
|  |  |  |  |  |
