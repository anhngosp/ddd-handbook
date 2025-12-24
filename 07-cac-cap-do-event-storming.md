# Chương 7 — Các cấp độ Event Storming (Big Picture, Process, Design-level)

Chương 6 giải thích “Event Storming là gì và vì sao đứng đầu quy trình”. Chương này đi sâu vào **ba cấp độ** của Event Storming và trả lời câu hỏi rất thực tế:

> “Mình đang ở giai đoạn nào, nên dùng cấp độ nào, và output phải trông như thế nào?”

Nhiều team thất bại vì dùng sai cấp độ: họ làm design-level quá sớm (khi domain chưa rõ), hoặc chỉ dừng ở big picture rồi nhảy thẳng vào code. Kết quả là hoặc over-engineering, hoặc thiếu nền tảng để giữ đúng domain.

Trong ADLP, ba cấp độ này đặc biệt quan trọng:
- **Big Picture** giúp chốt pipeline end-to-end (ingest → prelabel → assignment → labeling → quality → export).  
- **Process-level** làm lộ policy như lock TTL, review thresholds, escalation.  
- **Design-level** chuẩn bị aggregate boundaries và commands/events để code.

---

## Bạn sẽ nhận được gì sau chương này?

1) Hiểu rõ mục tiêu, thời điểm dùng, và output của từng cấp độ Event Storming.  
2) Biết cách “đi đúng cấp độ” để tránh làm design-level khi domain chưa chín.  
3) Có ví dụ cụ thể áp dụng vào ADLP cho cả 3 cấp độ.  
4) Có best practices, trade-offs, anti-patterns.  
5) Có exercise hướng dẫn để tự chạy cả 3 cấp độ theo một workflow.

---

## 1) Ba cấp độ, ba câu hỏi khác nhau

Event Storming không phải một hoạt động duy nhất. Nó là một chuỗi cấp độ, mỗi cấp trả lời một câu hỏi khác nhau:

1) **Big Picture**: “Workflow end-to-end là gì?”  
2) **Process-level**: “Ai làm gì, khi nào, theo policy nào?”  
3) **Design-level**: “Aggregate nào bảo vệ invariant? Event nào phát ra? Command nào kích hoạt?”

Mỗi cấp độ có **độ chi tiết khác nhau**, và quan trọng nhất: **output của cấp trước là input của cấp sau**.

### 1.1 Bảng so sánh 3 cấp độ

| Cấp độ | Thời gian | Participants | Output | Khi dừng |
|---|---|---|---|---|
| Big Picture | 2–3h | All stakeholders | 10–20 events + hotspots | Khi có shared understanding |
| Process-level | 4–6h | Domain experts + leads | Commands + policies + actors | Khi policies rõ |
| Design-level | 2–3h/context | Tech leads + architects | Aggregates + invariants | Khi sẵn sàng code |

### 1.2 Zoom levels analogy

```
Big Picture   = Google Maps (toàn cảnh thành phố)
Process-level = Street View (nhìn rõ đường phố)
Design-level  = Floor plan (chi tiết phòng, cửa, tường)
```

---

## 2) Big Picture Event Storming

### 2.1 Mục tiêu
Big Picture nhằm tạo shared understanding. Bạn muốn có một timeline “điều đã xảy ra” bao phủ toàn bộ workflow chính.

### 2.2 Output điển hình
- 10–20 domain events cấp business.
- Hotspots/questions (điểm mơ hồ).
- Glossary seed (thuật ngữ bắt đầu được chốt nghĩa).

### 2.3 Ví dụ ADLP (rút gọn)

Workflow premium 48h:

`DataItemUploaded → DataItemNormalized → PrelabelCompleted → BatchCreated → BatchAssigned → BatchSubmitted → QualityEvaluated → (ReviewRequired) → BatchAccepted → DatasetExported`

Bạn chưa cần bàn ai làm gì, chưa cần bàn DB. Mục tiêu là **nhìn ra toàn cảnh**.

### 2.4 Trade-offs
- **Ưu:** nhanh, tạo shared language, ít tranh cãi kỹ thuật.  
- **Nhược:** thiếu chi tiết policy, chưa đủ để code.

> **BEST PRACTICE**  
> Big Picture chỉ cần 2–3 giờ. Nếu dài hơn, bạn đang rơi vào process-level mà không nhận ra.

---

## 3) Process-level Event Storming

### 3.1 Mục tiêu
Process-level trả lời: “Ai khởi tạo sự kiện này? policy nào quyết định? hệ thống bên ngoài nào can thiệp?”  
Đây là nơi bạn gắn **commands, actors, policies, external systems**.

### 3.2 Output điển hình
- Event timeline có gắn actor/command/policy.
- Các policy rõ ràng (ví dụ: “lock TTL 4h”).
- Các external systems xuất hiện (S3, payment gateway, notification…).

### 3.3 Ví dụ ADLP
Event `BatchAssigned` có thể được kích hoạt bởi:
- Actor: Assignment Service
- Policy: “labeler rating > 1200 thì ưu tiên premium”
- External system: Notification Service gửi thông báo

Event `BatchSubmitted`:
- Actor: Labeler
- Policy: “submit trước deadline”
- External: Audit logging

### 3.4 Trade-offs
- **Ưu:** lộ policy và ownership.  
- **Nhược:** dễ trượt sang tranh luận kỹ thuật (REST vs Kafka).

> **WARNING**  
> Nếu bạn bắt đầu tranh luận “dùng Kafka hay REST” ở đây, hãy dừng lại và ghi vào hotspot, không quyết ngay.

---

## 4) Design-level Event Storming

### 4.1 Mục tiêu
Design-level dùng để chuẩn bị cho tactical design: aggregate boundaries, invariants, commands/events chi tiết.

### 4.2 Output điển hình
- Candidate aggregates (Batch, Assignment, QualityEvaluation…).
- Commands cụ thể (AssignBatch, SubmitBatch).
- Domain events cụ thể (BatchAssigned, BatchSubmitted).
- Invariants được ghi rõ.

### 4.3 Ví dụ ADLP
Aggregate `Batch`:
- Invariant: “một batch chỉ assigned cho một labeler tại một thời điểm”.
- Command: `AssignBatch(labeler_id, ttl)`
- Event: `BatchAssigned`

Aggregate `QualityEvaluation`:
- Invariant: “BatchAccepted chỉ khi quality_score >= threshold”.
- Command: `EvaluateQuality(batch_id)`
- Event: `QualityEvaluated`, `BatchAccepted` hoặc `ReviewRequired`

### 4.4 Trade-offs
- **Ưu:** đủ thông tin để code.  
- **Nhược:** nếu làm quá sớm, bạn sẽ thiết kế dựa trên giả định sai.

> **BEST PRACTICE**  
> Chỉ vào design-level khi Big Picture và Process-level đã ổn (hotspots đã được giải quyết hoặc được ghi ADR).

---

## 5) Khi nào dùng cấp độ nào?

### Nếu bạn đang…
- **Khởi động dự án / chưa có shared language** → Big Picture  
- **Cần làm rõ policy và actor** → Process-level  
- **Chuẩn bị coding cho 1 slice** → Design-level

> **EXAMPLE (ADLP)**  
> Tuần 1: Big Picture cho workflow premium 48h.  
> Tuần 2: Process-level để chốt lock TTL và escalation policy.  
> Tuần 3: Design-level cho Task Assignment và Quality context trước khi code.

### 5.1 Transition checklist

**Big Picture → Process-level**
- [ ] Timeline có 10–20 events nghiệp vụ rõ ràng  
- [ ] Glossary seed có 15–30 terms được chốt nghĩa v0  
- [ ] Top 5 hotspots đã ghi và có owner  
- [ ] Stakeholders đồng ý “đây là workflow chính”  

**Process-level → Design-level**
- [ ] Policies quan trọng đã chốt (TTL, thresholds, escalation)  
- [ ] Actors và ownership rõ ràng  
- [ ] External systems đã xác định  
- [ ] Bounded context candidates đã xuất hiện  

---

## 6) Best practices (kèm giải thích)

### 6.1 Làm đúng cấp độ, đừng nhảy
Nếu bạn chưa chốt ngôn ngữ, đừng vào design-level. Bạn sẽ “đúc sai khuôn”.

### 6.2 Luôn ghi lại hotspots
Hotspot là “nợ hiểu biết”. Nếu không ghi, bạn sẽ tranh luận lặp lại.

### 6.3 Domain expert phải có mặt ở Big Picture và Process-level
Design-level có thể do dev/architect chủ đạo, nhưng Big Picture và Process-level cần domain expert thật sự.

---

## 7) Anti-patterns thường gặp

1) **Big Picture bị bỏ qua** → team đi thẳng vào design-level và tạo aggregate sai.  
2) **Process-level thành technical design** → mất shared understanding.  
3) **Design-level làm trước khi hotspots rõ** → code dựa trên giả định sai.

---

## 8) Exercise có hướng dẫn: chạy 3 cấp độ cho workflow ADLP

### Bước 1: Big Picture (15 phút)
Viết timeline 8–12 events cho workflow premium 48h.

### Bước 2: Process-level (20 phút)
Chọn 3 events quan trọng (BatchAssigned, BatchSubmitted, BatchAccepted) và gắn actor/policy/external system.

### Bước 3: Design-level (20 phút)
Chọn 1 aggregate (Batch hoặc QualityEvaluation), viết invariant + command + event.

### Đáp án tham khảo (rút gọn)
- Big Picture: DataItemUploaded → PrelabelCompleted → BatchCreated → BatchAssigned → BatchSubmitted → QualityEvaluated → BatchAccepted → DatasetExported  
- Process-level: BatchAssigned (actor: assignment service; policy: rating>1200; external: notification)  
- Design-level: Aggregate Batch (invariant: one assignment at a time; command AssignBatch; event BatchAssigned)

---

## 9) Artefacts/Deliverables sau chương này

- Một timeline big picture đủ rõ (10–20 events).  
- Process-level map cho 3–5 events quan trọng.  
- 1–2 aggregate candidates với invariant + commands/events.  
- Danh sách hotspots đã được ghi lại.

### 9.1 Output artifacts template (gợi ý)

**Big Picture Output**
- Event timeline (Mermaid/ASCII)  
- Hotspots list (hotspot | owner | priority | resolution plan)  
- Glossary seed (term | definition v0 | context | conflicts)  

**Process-level Output**
- Process cards (Event | Actor | Command | Policy | External System)  
- Policy notes (Policy name | Rule | Exception | Owner)  
- Actor–Context mapping  

**Design-level Output**
- Aggregate candidates (Name | Invariants | Commands | Events)  
- Command–Event flow (sequence diagram)  
- Invariants list  

---

## Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Bạn biết mục tiêu của level đang làm (Big Picture vs Process vs Design-level)  
> - [ ] Big Picture đã đủ rõ trước khi đi sâu (timeline 10–20 events + hotspots top-5)  
> - [ ] Chỉ chọn 1–2 flow “đắt tiền” để làm process-level (đừng làm mọi thứ)  
> - [ ] Chỉ đi design-level khi đã chốt nghĩa thuật ngữ v0 và có owner cho hotspots chính  
> - [ ] Sau mỗi level, bạn có output artefacts và quyết định “dừng hay lên cấp” có lý do  
