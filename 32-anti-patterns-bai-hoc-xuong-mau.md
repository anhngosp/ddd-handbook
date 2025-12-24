# Chương 32 — Anti-patterns & bài học xương máu: vì sao DDD “trượt”, và cách kéo nó về đường đúng

Bạn có thể đọc hết handbook này và vẫn làm sai. Không phải vì bạn không hiểu thuật ngữ. Mà vì trong đời thật, có những lực kéo rất mạnh:
- deadline kéo bạn về đường tắt,
- áp lực “ship nhanh” kéo bạn về CRUD,
- kiến trúc “trông có vẻ enterprise” kéo bạn về over-engineering,
- và sự mơ hồ của domain kéo bạn về “đoán bừa”.

Chương 32 là phần “phòng chống tai nạn”. Nó không nhằm làm bạn sợ DDD. Nó nhằm giúp bạn nhận diện sớm các anti-pattern trước khi chúng hóa thành “văn hóa codebase”.

Ví dụ xuyên suốt vẫn là ADLP. Hãy tưởng tượng một năm sau khi MVP chạy production: lượng labelers tăng, workflows dài hơn, payout/quality bị soi kỹ hơn. Những anti-pattern dưới đây chính là thứ sẽ xuất hiện đầu tiên.

---

## Bạn sẽ nhận được gì sau chương này?

1) Nhận diện các anti-pattern phổ biến nhất khi áp dụng DDD (kèm triệu chứng, hậu quả, cách sửa).  
2) Phân biệt “tối giản thực dụng” với “làm ẩu” (DDD-lite vs DDD nửa mùa).  
3) Một bộ “câu hỏi chặn cửa” để review quyết định kiến trúc trước khi nó khóa bạn vào đường sai.  
4) Áp dụng vào ADLP: các điểm dễ trượt nhất (Assignment/Quality/Wallet/Export).  
5) Exercise có hướng dẫn: audit một module và viết kế hoạch kéo về đúng trong 1 sprint.

---

## 1) Over-engineering: DDD bị biến thành “tôn giáo” thay vì công cụ

### Triệu chứng
Bạn thấy các dấu hiệu:
- dựng microservices cho mọi bounded context ngay từ tuần đầu,
- viết 15 lớp abstraction trước khi có 1 use case chạy,
- tạo “framework nội bộ” cho DDD,
- mọi thứ đều có interface dù chưa có reason.

### Hậu quả
Team chậm, mất niềm tin, và kết luận “DDD là màu mè”.

### Cách tránh (thực dụng)
DDD không bắt bạn “làm nhiều”. DDD bắt bạn “làm đúng chỗ”.

Trong ADLP:
- Core contexts (Task Assignment, Quality Assurance) đáng đầu tư sâu.
- Generic contexts (Identity & Access) dùng giải pháp chuẩn, giữ boundaries và contracts, đừng tự phát minh.

> **BEST PRACTICE**  
> Mọi abstraction phải trả lời 2 câu: “nó giảm rủi ro gì?” và “nó sẽ giúp ta đổi gì trong 3 tháng tới?” Nếu không trả lời được, hãy hoãn.

---

## 2) DDD “nửa mùa”: vẽ đúng nhưng code không phản ánh domain

Đây là anti-pattern nguy hiểm nhất vì nó tạo cảm giác “chúng ta làm DDD rồi”, trong khi thực tế bạn chỉ đổi tên folder.

### Triệu chứng
- Có folder `domain/` nhưng bên trong chỉ có DTO, không có invariants.
- “Service” chứa toàn bộ rule; entity chỉ có getter/setter.
- Domain errors không có, chỉ có `InvalidStateException`.
- Controller/consumer có logic nghiệp vụ (if-else theo trạng thái).

### Hậu quả
Bạn mất cả hai: vừa có boilerplate, vừa không có lợi ích DDD. Bug vẫn rò rỉ, refactor vẫn đau.

### Cách kéo về đúng
Bắt đầu từ một use case đắt tiền (Chương 28), và làm 3 bước:
1) đưa invariants vào aggregate/VO,
2) làm controller mỏng lại,
3) viết 2–5 unit tests cho invariants.

> **NOTE**  
> Không cần “đập lại toàn bộ”. Chỉ cần chọn đúng lát, sửa đúng nơi.

---

## 3) “Data-driven design” quay lại bằng cửa sau: join xuyên context, shared schema, gọi thẳng DB

### Triệu chứng
- Module A query bảng của module B (hoặc service A đọc DB service B).
- Có “database chung” và mọi service cùng viết vào một schema.
- Consumer “thiếu field” nên gọi ngược DB producer để lấy thêm.

### Hậu quả
Bounded context trở thành khẩu hiệu. Coupling thật nằm ở DB. Khi schema đổi, mọi thứ vỡ. Khi incident xảy ra, không ai biết ai là owner dữ liệu.

### Cách tránh
- Database per service theo nghĩa **ownership**, không nhất thiết per-instance (Chương 25).
- Read models/CQRS-lite để phục vụ query mà không phá boundaries.
- Event/API payload “tối thiểu nhưng đủ” để consumer không phải gọi ngược (Chương 24, checklist integration).

> **WARNING**  
> “Cho nhanh” là lý do phổ biến nhất để phá boundary — và cũng là lý do phổ biến nhất để bạn phải rewrite.

---

## 4) Event-driven sai cách: event như log kỹ thuật, thiếu contract, thiếu versioning, thiếu idempotency

Event-driven không tự động làm bạn loose coupling. Nếu làm sai, nó làm bạn khó debug gấp 10 lần.

### Triệu chứng
- event type mơ hồ: `Updated`, `Changed`, `Processed`.
- payload kiểu “full row” hoặc “full aggregate dump”.
- không có envelope chuẩn (correlation/causation/version).
- consumer không idempotent; duplicates gây double side effects.
- retry vô tội vạ; DLQ không owner.

### Hậu quả
Workflow vỡ âm thầm: payout double, export double, quality chạy lại, backlog tăng mà không ai biết nguyên nhân.

### Cách tránh (thực dụng)
- Event phải business-readable và có owner context (Chương 21).
- Published language + semver + consumer tolerant (templates + Chương 24).
- Idempotency theo business identity, không theo event_id (Chương 21/24).
- Có SLO và runbook cho event processing (Chương 26).

> **EXAMPLE (ADLP)**  
> `BatchAccepted` là event đắt tiền vì kéo Wallet/Export. Nếu bạn không idempotent, bạn mất tiền và mất uy tín — không phải “bug nhẹ”.

---

## 5) “Chia microservices trước, rồi mới tìm domain”: distributed monolith ra đời

### Triệu chứng
- Service boundaries theo UI, theo bảng, theo team cũ, hoặc theo “đoán”.
- Request chain A→B→C→D cho 1 action user.
- Tất cả services share cùng một “common domain library”.

### Hậu quả
Bạn có distributed monolith: vận hành khó, deploy khó, nhưng đổi vẫn đau như monolith (thậm chí đau hơn).

### Cách tránh
DDD nói rõ thứ tự:
1) discovery + event storming,
2) strategic design (bounded contexts + context map),
3) rồi mới mapping sang kiến trúc (Chương 23).

Nếu bạn đã lỡ tách sai:
- đừng cố “chữa bằng thêm message broker”.
- hãy quay lại strategic: định nghĩa lại boundaries và contracts, rồi refactor theo strangler.

---

## 6) “Một model cho tất cả”: không chấp nhận ngữ nghĩa thay đổi theo context

### Triệu chứng
Bạn cố dùng cùng một class `Batch` cho Assignment/Labeling/Quality/Export/Wallet.

### Hậu quả
`Batch` trở thành “mọi thứ”. Field phình to. Rule mâu thuẫn. Thay đổi ở Quality làm vỡ Assignment.

### Cách tránh
Chấp nhận sự thật của DDD: cùng một từ có thể khác nghĩa ở context khác.

Trong ADLP:
- `Batch` trong Assignment nói về lock, routing, priority.
- `Batch` trong Quality nói về evaluation, decision, audit.
- `Batch` trong Wallet nói về earning/credit reference.

Đó là lý do bạn cần bounded contexts và published language.

---

## 7) “Để NFR cho ops”: không có observability, không có SLO, không có runbook

### Triệu chứng
- không có correlation_id, debug workflow bằng grep log thủ công,
- alert theo CPU, không theo SLO,
- DLQ tăng mà không ai chịu trách nhiệm.

### Hậu quả
Bạn có “hệ thống chạy bằng cầu nguyện”. Incident sẽ dạy bạn bằng downtime và mất niềm tin.

### Cách tránh
NFR by design (Chương 26) không phải tài liệu cho ops. Nó là driver cho thiết kế và testing:
- correlation_id là một phần contract,
- event processing SLO là một phần workflow,
- runbook là một phần “definition of done”.

---

## 8) “Policy rải rác”: rule ở config, ở code, ở SQL, ở UI, không có owner

### Triệu chứng
Rule như TTL, threshold, payout formula:
- có 3 bản khác nhau ở 3 nơi,
- thay đổi rule phải sửa nhiều repo,
- không ai chắc “đúng là cái nào”.

### Hậu quả
Sai business và sai fairness (đặc biệt ở marketplace như ADLP).

### Cách tránh
- Policy phải có owner context.
- Policy changes phải có version và audit (policy_version).
- Rule đắt tiền phải có unit tests (table-driven).

---

## 9) Detection tools/metrics: phát hiện anti-pattern sớm

Những anti-pattern nguy hiểm thường “bò vào” rất sớm. Dưới đây là các tín hiệu định lượng dễ theo dõi:

**Coupling & ownership**
- Số PR/commit chạm 3+ bounded contexts cho một use case (tín hiệu boundary mơ hồ).  
- Số query/read model “đọc thẳng DB context khác” (tín hiệu data-driven quay lại).  

**Event-driven health**
- Tỉ lệ retry/DLQ cho các event đắt tiền (BatchAccepted, ExportCreated).  
- Số consumer không idempotent (phát hiện qua “double side effects”).  

**Domain quality**
- Tỉ lệ unit tests cho domain invariants (aggregate/VO) so với test tổng.  
- Số lỗi production gắn với “rule sai/ngữ nghĩa sai” (so với lỗi hạ tầng).  

**Change dynamics**
- Lead time thay đổi rule/policy (từ yêu cầu đến deploy).  
- Số lần “breaking change” contract không có versioning/deprecation plan.  

**Operational signals**
- Số sự cố không trace được correlation_id end-to-end.  
- Backlog outbox tăng đều (dấu hiệu publish/consume đang “kẹt”).  

Nếu 2–3 chỉ số xấu cùng lúc, bạn đang trượt khỏi DDD mà không biết.

---

## 10) Best practices: “câu hỏi chặn cửa” trước khi merge quyết định sai

Khi review thiết kế/PR, hãy hỏi:
1) Invariant nào được bảo vệ ở đâu? Có test không?
2) Contract nào bị ảnh hưởng? Có versioning/deprecation plan không?
3) Consumer có phải gọi ngược producer DB không?
4) Retry/DLQ/idempotency đã rõ chưa? Owner là ai?
5) Correlation_id có đi xuyên workflow không?
6) Thay đổi này localize trong một context hay lan ra nhiều?

Nếu không trả lời được, bạn đang đi vào vùng rủi ro.

---

## 11) Artefacts/Deliverables

Sau chương này, bạn nên có:
1) Một danh sách anti-pattern “đặc hữu” của team (top 5) và policy phòng tránh.  
2) Một checklist review (bạn có thể dùng `design/docs/0.ref/DDDPractical/checklist-code-review-ddd.md`).  
3) Một “stop-the-line rule”: khi nào phải dừng và quay lại discovery/strategic.  

---

## 12) Exercise (có hướng dẫn) — Audit một module và viết kế hoạch “kéo về đúng” trong 1 sprint

### Bài toán
Chọn một context trong ADLP mà team hay làm nhanh (ví dụ Assignment hoặc Export). Giả sử code hiện tại có dấu hiệu:
- controller có logic nghiệp vụ,
- publish event trước commit,
- consumer không idempotent.

Bạn cần viết kế hoạch sửa trong 1 sprint, ưu tiên “đúng và an toàn”.

### Cách làm (gợi ý từng bước)

**Bước 1 — Chọn 1 use case đắt tiền**  
Ví dụ: `ClaimBatch` hoặc `CreateExport`.

**Bước 2 — Liệt kê 3 anti-pattern đang có**  
Ghi ngắn nhưng cụ thể: file nào, hành vi nào.

**Bước 3 — Thiết kế “spine” đúng (Chương 28)**  
Map: controller → handler → domain → repository/UoW → outbox.

**Bước 4 — Chọn 2 tests bắt buộc**  
Gợi ý:
- unit test cho invariant chính,
- integration test cho outbox/idempotency.

**Bước 5 — Chọn 1 contract guardrail**  
Event schema hoặc API error model, và viết contract test tối thiểu.

### Output mong đợi
1) 3 anti-pattern + hậu quả nếu để nguyên.  
2) Plan 5–7 bước (theo thứ tự deploy-safe).  
3) 2 tests + 1 contract test mô tả rõ.  

---

## Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Anti-pattern nào đang xuất hiện? (DDD nửa mùa / shared DB / event như log / sync chain dài)  
> - [ ] Có “stop-the-line rule” khi chạm payout/export/compliance (bắt buộc idempotency/outbox/contracts)  
> - [ ] Có checklist code review DDD để chặn lỗi lặp lại (`design/docs/0.ref/DDDPractical/checklist-code-review-ddd.md`)  
> - [ ] Có contract tests cho edges đắt tiền (Quality ↔ Wallet/Export)  
> - [ ] Có owner cho DLQ/runbook/SLO, không để sự cố “vô chủ”  
