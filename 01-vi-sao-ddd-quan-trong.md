# Chương 1 — Vì sao DDD vẫn quan trọng trong hệ thống hiện đại?

Bạn có thể sở hữu một hệ thống “đúng kỹ thuật”: CI/CD mượt, observability đủ, infra chuẩn, code sạch, test xanh. Nhưng hệ thống vẫn có thể thất bại theo cách… rất khó chịu: **chạy được, nhưng không đúng**. “Đúng” ở đây không phải “không crash”, mà là đúng với thực tế vận hành của business: đúng quy tắc, đúng trạng thái, đúng trách nhiệm, đúng cách giải thích khi có tranh chấp.

Domain-Driven Design (DDD) tồn tại vì một sự thật bền vững: **hệ thống business thường chết vì hiểu sai nghiệp vụ hơn là vì thiếu framework**. Chương này không mở đầu bằng định nghĩa sách giáo khoa. Mình sẽ đi từ một câu chuyện rất thật trong ADLP (AI Data Labeling Platform) và từ đó bạn sẽ thấy DDD có ý nghĩa cụ thể như thế nào, tại sao “nghe có vẻ triết lý” nhưng lại cứu bạn ở những chỗ đắt nhất.

Trong handbook này, ví dụ xuyên suốt là ADLP theo Strategic Design v0.2: `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`.

---

## Bạn sẽ nhận được gì sau chương này?

Sau chương 1, bạn nên đạt 3 thứ:

1) **Hiểu DDD bằng ngôn ngữ “rủi ro và chi phí”.** Bạn sẽ biết DDD giảm rủi ro kiểu gì, và giảm ở những điểm nào là đáng tiền nhất.  
2) **Hiểu 5 khái niệm nền tảng bằng ví dụ ADLP:** Ubiquitous Language, Bounded Context, Context Map, Invariant, Domain Event — không phải thuộc lòng, mà hiểu “nó cứu mình ở đâu”.  
3) **Biết bắt tay vào làm ngay trong tuần 1:** làm gì trước, ai tham gia, output là gì, tránh sai lầm nào.

Cuối chương có một bài tập nhỏ, nhưng lần này không phải “tự bơi”: mình sẽ hướng dẫn cách nghĩ và đưa ví dụ mẫu để bạn có thể làm theo.

---

## 1) Một câu chuyện thật (ADLP): “chúng ta làm xong rồi mà sao vẫn sai?”

Hãy đặt bạn vào một tình huống rất “đời” của ADLP:

Một data consumer mua dataset “premium” và yêu cầu giao trong 48 giờ. Hệ thống cần đi qua một chuỗi bước tưởng như hiển nhiên:

1) Ingest audio (upload/crawl), validate, normalize, dedupe.  
2) Prelabel STT: segmentation + confidence, lưu kết quả, gắn model version.  
3) Task assignment: tách batch, tính priority, lock TTL, routing theo skill/rating/confidence.  
4) Labeling: UI phát audio/segments, autosave, submit.  
5) Quality: tính WER/agreement, automated review nếu bất thường, escalation nếu cần.  
6) Export: chỉ xuất dữ liệu **đã accept**, đóng gói version, signed URL, audit download.

Nếu bạn làm theo kiểu “kỹ thuật trước”, bạn sẽ dễ rơi vào kịch bản này:

Bạn có endpoint upload/assign/submit/export; bạn lưu mọi thứ vào DB với một vài trường `status`; bạn thêm cron job để “dọn dẹp”; và bạn chạy được end-to-end trên demo.

Nhưng khi đi vào vận hành thật, vấn đề sẽ nổ đúng chỗ business quan tâm: labeler submit nhanh bất thường làm chất lượng kém dẫn tới khiếu nại; hai labeler lấy trùng batch dẫn tới tranh chấp “ai được trả tiền”; batch hết 4 giờ không quay lại queue làm vỡ SLA; export lẫn dữ liệu “đã submit” nhưng chưa “đã accept” làm dataset premium bị nhiễm rác; và khi bị hỏi “vì sao batch này được accept?”, hệ thống không giải thích được vì thiếu audit trail/compliance.

Điểm đau ở đây không nằm ở framework. Nó nằm ở **định nghĩa** và **ranh giới**.

Ví dụ: “Submit” là gì? Là labeler bấm nút? Hay là batch đạt chất lượng? Nếu bạn gọi “submit = done” thì payout sẽ sai. “Lock TTL” là chi tiết kỹ thuật? Không. Nó là rule của marketplace về fairness và SLA. “Quality gate” là vài con số? Không. Nó là định nghĩa “premium” và là ranh giới giữa dữ liệu rác và dữ liệu bán được.

> **NOTE**  
> DDD không làm giảm số thứ bạn cần nghĩ. DDD làm giảm sự hỗn loạn bằng cách: đặt tên đúng, tách ranh giới đúng, và bảo vệ quy tắc đúng.

---

## 2) DDD giải quyết loại bug nào?

Bug kỹ thuật thường “đau” theo kiểu: crash, timeout, race-condition. Bug domain “đau” theo kiểu: chạy được nhưng sai business. Và bug domain là loại bug đắt nhất vì nó thường lộ muộn.

Để thấy rõ, bạn hãy tưởng tượng hai kiểu lỗi:

### 2.1 Lỗi kỹ thuật (thường lộ sớm)

Ví dụ: prelabel service timeout khi gọi SageMaker. Bạn thêm retry/backoff/circuit breaker, và vấn đề giảm. Đây là lỗi kỹ thuật — không xem nhẹ, nhưng “có đường” để xử.

### 2.2 Lỗi khái niệm (lộ muộn, rất đắt)

Ví dụ: bạn trả tiền ngay khi labeler submit. Ban đầu có vẻ hợp lý, vì “submit là xong rồi”. Nhưng với ADLP, Strategic Design v0.2 coi Quality Assurance là core domain — nghĩa là business value nằm ở chất lượng, không nằm ở tốc độ submit.

Vài tuần sau:
- gian lận xuất hiện (submit nhanh, chất lượng thấp),
- consumer phàn nàn,
- bạn phải thêm review/escalation,
- bạn phải sửa payout và xử lý tranh chấp,
- bạn phải giải thích “vì sao batch này được trả/không trả”, nhưng không có audit trail.

Lỗi ở đây không phải bug code. Nó là **lỗi định nghĩa**: “completed” bị hiểu sai.

DDD được tạo ra để giảm loại lỗi này. Nhưng DDD không “chỉ cho bạn một pattern”. Nó thay đổi cách bạn bắt đầu dự án: bạn bắt đầu từ **ngôn ngữ và ranh giới của business**, chứ không bắt đầu từ service/database/UI.

---

## 3) 5 khái niệm nền tảng (giải thích bằng ADLP, không nói suông)

Nhiều người nghe DDD rồi thấy mơ hồ vì gặp một loạt thuật ngữ. Ta làm ngược lại: hiểu khái niệm bằng một câu hỏi thực dụng: “nó giúp mình tránh tai nạn nào?”

### 3.1 Ubiquitous Language: “nói cùng một thứ bằng cùng một từ”

Ubiquitous Language (ngôn ngữ chung) là một thỏa thuận: domain expert và dev dùng **cùng từ**, và code phản ánh đúng từ đó.

Trong ADLP, nếu bạn không chốt ngôn ngữ, bạn sẽ nhanh chóng bị “đánh nhau nghĩa”. Cùng một từ như “task” có thể bị hiểu là “một segment” hoặc “một batch”; “approved” có thể là “reviewer approve” hoặc “quality gate passed”; “confidence” có thể là “model confidence” hoặc “label quality confidence”.

Thảm họa ở đây là: bạn viết code rất đúng với từ bạn dùng, nhưng từ bạn dùng không đúng với business.

**Best practice (rất cụ thể):** ngay tuần đầu, bạn lập một glossary seed 10–30 thuật ngữ. Mỗi thuật ngữ tối thiểu có: định nghĩa 1–2 câu, một ví dụ đúng/sai, và “owner context” (thuộc bounded context nào).

Khi bạn bắt đầu làm Event Storming (chương sau), glossary sẽ tự nhiên “được lấp đầy” và mâu thuẫn nghĩa sẽ lộ ra sớm.

Để bạn không phải “tự nghĩ format”, đây là template tối thiểu (và cũng là best practice nên dùng ngay từ đầu):

| Term | Định nghĩa 1–2 câu | Ví dụ đúng | Ví dụ sai | Owner context |
|---|---|---|---|---|
| Batch | Một nhóm segments được phân công như một đơn vị công việc; có lock TTL và lifecycle riêng. | “Batch A được assign cho labeler X đến 14:00.” | “Batch = một segment.” | Task Assignment |
| Segment | Một đoạn audio có start/end; là đơn vị nhỏ để transcribe và tính quality. | “Segment 10s có confidence 0.82.” | “Segment = batch.” | Prelabeling/Labeling |
| Submitted | Labeler đã nộp kết quả chỉnh sửa; **chưa** đảm bảo đạt chất lượng. | “BatchSubmitted → QualityEvaluated.” | “Submitted = Accepted.” | Labeling |
| Accepted | Batch đã qua quality gate (WER/agreement/…); được phép export/payout. | “BatchAccepted → Exportable.” | “Accepted do UI bấm submit.” | Quality Assurance |
| Confidence | Mức chắc chắn của model prelabel; không đồng nghĩa chất lượng labeler. | “prelabel_confidence_avg=0.9.” | “confidence = quality_score.” | Prelabeling |

Bạn sẽ thấy ngay tác dụng: chỉ cần điền được 5–10 dòng như vậy, team đã giảm mạnh tranh luận “từ này nghĩa gì” và giảm khả năng viết nhầm model.

### 3.2 Bounded Context: “một model không thể đúng cho mọi nơi”

Bounded Context là ranh giới nơi một thuật ngữ và mô hình của nó có nghĩa nhất quán. Điều này nghe trừu tượng cho tới khi bạn nhìn ADLP: trong Labeling context, “Transcript” là thứ labeler đang sửa (có version, autosave); còn trong Quality Assurance context, “Transcript” là đối tượng để tính WER/agreement và ra quyết định accept/reject.

Nếu bạn cố dùng “một model Transcript” cho cả hai, bạn sẽ kéo theo coupling và tranh cãi: labeler sửa gì, reviewer sửa gì, quality tính gì, ai sở hữu trạng thái nào?

Strategic Design v0.2 của ADLP có 9 bounded contexts. Bạn không cần nhớ hết ngay, nhưng bạn cần hiểu ý nghĩa: tách ranh giới để giảm “mơ hồ” và giảm coupling.

> **NOTE**  
> Bounded Context không phải “một database” hay “một service”. Nó là ranh giới của **ngôn ngữ và trách nhiệm**. Service/DB là quyết định triển khai — đến sau.

### 3.3 Context Map: “coupling thật sự nằm ở integration”

Bạn có thể chia bounded contexts rất đẹp, nhưng nếu bạn tích hợp sai, hệ thống vẫn thành distributed monolith.

Context Map trả lời: context nào upstream/downstream, quan hệ là partnership hay customer/supplier, chỗ nào cần ACL, chỗ nào conformist.

Trong ADLP, Task Assignment cần skills/rating từ User Profile (customer/supplier); Quality quyết định payout (liên quan Wallet & Payment); Prelabeling phát event để Assignment tạo batch.

Nếu bạn không vẽ context map, bạn sẽ “tự nhiên” làm những thứ cực nguy hiểm: gọi thẳng DB của context khác “cho nhanh”, chia sẻ bảng user/batch giữa nhiều service, hoặc để event schema thay đổi tùy hứng.

Context Map là bản đồ để bạn chấp nhận coupling có chủ đích và tránh coupling vô thức.

### 3.4 Invariant: “quy tắc phải luôn đúng”

Invariant là rule mà nếu sai thì hệ thống không còn đúng với business. Đây là thứ mà bạn **phải bảo vệ bằng thiết kế**, chứ không thể hy vọng “ai đó nhớ”.

Ví dụ ADLP (rất đời): một batch chỉ được assigned cho một labeler tại một thời điểm; signed URL chỉ phát cho user có quyền và phải hết hạn; export chỉ bao gồm dữ liệu đã accepted; rating chỉ cập nhật dựa trên quality evaluation, không dựa trên cảm tính.

Khi bạn bắt đầu thiết kế tactical (aggregate boundaries), invariant sẽ là “điểm neo” để bạn quyết định transaction boundary: cái gì cần nhất quán ngay, cái gì chấp nhận eventual consistency.

### 3.5 Domain Event: “điều đã xảy ra” (không phải “hãy làm”)

Domain Event là sự kiện đã xảy ra trong domain. Nó quan trọng vì nó giúp:
thể hiện audit trail, dựng workflow event-driven, và giảm coupling giữa contexts.

Trong ADLP, bạn sẽ sớm thấy sự khác biệt giữa event đúng và event sai:

- `BatchSubmitted` (đúng): một điều đã xảy ra, có ý nghĩa nghiệp vụ.  
- `batch.updated` (sai): một thông báo kỹ thuật, consumer phải đoán “đổi gì”.

Domain Event không phải là command. Command là “hãy làm”; event là “đã xảy ra”. Nếu bạn lẫn lộn, hệ thống event-driven của bạn sẽ rất nhanh biến thành “message queue của CRUD”.

Để event không mơ hồ, bạn có thể dùng template sau (tối giản nhưng đủ để dùng trong workshop và review):

| Event | Owner context | Khi nào phát | Payload tối thiểu | Ai tiêu thụ | Ghi chú an toàn |
|---|---|---|---|---|---|
| PrelabelCompleted | Prelabeling | Sau khi inference + segmentation xong | `item_id`, `segments[]`, `model_id`, `model_version`, `avg_confidence` | Task Assignment | Idempotent theo `item_id + model_version` |
| BatchAccepted | Quality Assurance | Sau quality gate + (review nếu cần) | `batch_id`, `quality_score`, `accepted_at`, `policy_version` | Export, Wallet/Payment | Idempotent theo `batch_id` |

Template này giúp bạn “đứng vững” khi triển khai event-driven: bạn biết ai là owner, payload tối thiểu là gì, và idempotency key nên dựa trên cái gì.

---

## 4) Khi nào DDD tạo giá trị nhất? (và trade-off thật)

DDD không phải bắt buộc cho mọi dự án. Giá trị của DDD tăng khi:
- domain phức tạp và thay đổi,
- sai domain gây thiệt hại lớn,
- workflow dài, cần audit/trace,
- nhiều team/role tham gia.

Trong ADLP, strategic design đã chỉ ra core domains (nơi đáng đầu tư DDD sâu):
- Prelabeling orchestration,
- Task assignment & routing,
- Quality assurance.

Nhưng “đầu tư DDD sâu” cũng có giá:
- bạn phải dành thời gian discovery/workshop,
- bạn phải chấp nhận thiết kế có ranh giới rõ (khó “đi đường tắt”),
- bạn phải quản trị contracts/schema.

> **NOTE**  
> DDD không làm bạn ship chậm. DDD làm bạn **ship đúng**, và nhờ đó ship nhanh hơn về dài hạn vì ít rework.

---

## 5) “Bắt tay vào làm” trong tuần 1: làm gì trước để không mơ hồ?

Đây là phần nhiều người thiếu nhất: nghe khái niệm hay nhưng không biết bắt đầu thế nào. Dưới đây là một “kịch bản tuần 1” rất thực dụng cho dự án như ADLP.

### Ngày 1: Chốt người và phạm vi workshop

Bạn cần tối thiểu:
- 1–2 domain experts thật (người chịu trách nhiệm quy tắc vận hành/payout/quality),
- 1 facilitator (có thể là architect/tech lead),
- 2–4 dev leads (backend/frontend/ML nếu có),
- 1 người vận hành/QA nếu có.

Mục tiêu ngày 1 không phải “vẽ kiến trúc”, mà là chốt:
- workflow end-to-end mà business coi là quan trọng nhất (ví dụ “premium order 48h”),
- 3–5 rủi ro domain lớn nhất (nếu sai là toang),
- và lịch workshop.

### Ngày 2–3: Big Picture Event Storming (2–3 giờ)

Đầu ra mong muốn là:
- 10–20 domain events cấp business,
- các điểm nóng (hotspots) gây tranh cãi,
- và bản nháp glossary.

Bạn chưa cần Bounded Context “đẹp”. Bạn cần shared understanding.

Nếu bạn cần một agenda mẫu (2 giờ) để ngày mai làm ngay, dùng khung này:
1) 10 phút: thống nhất kịch bản (premium order 48h) và mục tiêu workshop.  
2) 40 phút: viết events theo timeline (chỉ “đã xảy ra”, không bàn giải pháp).  
3) 20 phút: gắn commands/actors ở vài điểm nóng (nơi tranh cãi).  
4) 30 phút: đánh dấu hotspots/questions (điểm mơ hồ), và chốt glossary seed.  
5) 20 phút: tổng hợp output và phân công ai làm rõ câu hỏi nào trước workshop tiếp theo.

### Ngày 4: Chốt sơ bộ bounded contexts và “đường đi” integration

Bạn đối chiếu với Strategic Design v0.2 của ADLP:
- 9 contexts đã có; hãy dùng nó như baseline.
- xác định 2–3 contexts cần làm sâu nhất trước (core).

Đầu ra mong muốn:
- một context map thô (ai upstream/downstream),
- quyết định sơ bộ: sync vs async ở các điểm chính.

### Ngày 5: Chốt 3–5 invariants “đắt nhất” và ADR cho trade-off lớn

Ví dụ invariants đắt trong ADLP:
- “Export chỉ lấy accepted, không lấy submitted.”
- “Batch chỉ thuộc về một labeler tại một thời điểm, lock TTL rõ.”
- “Quality quyết định payout.”

Mỗi tranh luận lớn nên có ADR: bạn không cần viết dài, nhưng cần ghi lý do để tuần sau không tranh luận lại từ đầu.

---

## 6) Best practices (và vì sao) — để DDD không thành nghi thức

### 6.1 Đừng bắt đầu bằng “chia service”

Nếu bạn chia service trước khi chốt language/boundaries, bạn sẽ rất nhanh có distributed monolith. DDD bắt đầu từ bounded contexts như ranh giới ngôn ngữ, sau đó mới quyết định deploy topology (microservices vs modular monolith).

### 6.2 Glossary phải “sống”, không phải tài liệu chết

Glossary không phải file để tick cho đủ. Nó là hợp đồng ngôn ngữ. Mỗi khi có tranh luận “từ này nghĩa gì”, bạn cập nhật glossary. Đến tháng thứ 2, glossary giúp onboarding và giảm tranh luận.

### 6.3 Event schema là API public: phải có versioning

Trong ADLP, events như `PrelabelCompleted`, `BatchSubmitted`, `BatchAccepted` sẽ được nhiều context tiêu thụ. Nếu bạn đổi schema tùy hứng, bạn tạo ra downtime logic (consumer xử lý sai) mà không ai thấy ngay. Vì vậy event schema phải có versioning và compatibility rules.

### 6.4 Tập trung vào core domain, đừng “DDD hóa” mọi thứ

Không phải context nào cũng cần domain model phong phú. Identity/Auth có thể dùng giải pháp chuẩn. Nhưng Task Assignment và Quality Assurance phải được đầu tư vì đó là nơi tạo giá trị và rủi ro.

---

## 7) Bài tập có hướng dẫn: viết 12 domain events cho workflow ADLP

Bạn nói đúng: “viết 12 domain events” mà không chỉ cách nghĩ thì rất mơ hồ. Vì vậy đây là cách làm.

### Bước 1: Chọn một kịch bản duy nhất (đừng chọn tất cả)

Chọn kịch bản: “premium order 48h” cho audio STT.

Bạn chỉ cần một kịch bản để luyện tư duy event-centric; các kịch bản khác sẽ làm sau.

### Bước 2: Viết lại workflow thành câu chuyện, không dùng thuật ngữ kỹ thuật

Hãy viết 6–10 câu dạng: “X xảy ra, rồi Y xảy ra”.
Ví dụ: “Người dùng upload audio” → “hệ thống chuẩn hóa audio” → “hệ thống chạy prelabel” → “hệ thống tạo batch” → “batch được gán cho labeler” → “labeler submit” → “quality đánh giá” → “batch được accept” → “dataset được export”.

Điểm quan trọng: đây là **điều đã xảy ra**, không phải “hãy làm”.

### Bước 3: Chuyển mỗi câu thành “event name” ở thì quá khứ

Quy tắc đặt tên event:
- Ở thì quá khứ: `XxxCreated`, `XxxSubmitted`, `XxxCompleted`, `XxxAssigned`.
- Tránh từ kỹ thuật mơ hồ: `updated`, `changed`, `processed` (trừ khi bạn định nghĩa rất rõ).
- Mỗi event phải trả lời: “đã xảy ra điều gì trong business?”

### Bước 4: Kiểm tra event có phải command trá hình không

Nếu event nghe như “hãy làm”: `RunPrelabeling`, `AssignBatchToLabeler` → đó là command.  
Event đúng phải là: `PrelabelCompleted`, `BatchAssigned`.

### Bước 5: Gắn event với bounded context (để bớt mơ hồ)

Mỗi event nên thuộc một context “phát ra” (owner). Ví dụ:
- `DataItemIngested` thuộc Data Ingestion.
- `PrelabelCompleted` thuộc Prelabeling.
- `BatchAssigned` thuộc Task Assignment.
- `BatchAccepted` thuộc Quality Assurance.

### Ví dụ mẫu (12 events) cho ADLP

Bạn có thể dùng luôn danh sách này như đáp án tham khảo. Khi bạn làm, bạn không cần giống y hệt; quan trọng là “tư duy đúng”.

1) `DataItemUploaded` (Data Ingestion) — có upload xảy ra.  
2) `DataItemNormalized` (Data Ingestion) — audio đã được chuẩn hóa/dedupe xong.  
3) `PrelabelJobCreated` (Prelabeling) — hệ thống tạo job prelabel (có thể từ event #2).  
4) `PrelabelCompleted` (Prelabeling) — có transcript/segments/confidence và model version.  
5) `BatchCreated` (Task Assignment) — segments được gom thành batch theo policy.  
6) `BatchPrioritized` (Task Assignment) — batch được tính priority (tier/deadline/difficulty).  
7) `BatchAssigned` (Task Assignment) — batch được lock/gán cho labeler (kèm TTL).  
8) `LabelingStarted` (Labeling) — labeler bắt đầu phiên làm việc (có thể optional).  
9) `BatchSubmitted` (Labeling) — labeler submit transcript.  
10) `QualityEvaluated` (Quality Assurance) — WER/agreement/anomaly/bias computed.  
11) `ReviewRequired` (Quality Assurance) — nếu dưới threshold hoặc anomaly (có thể thay bằng `BatchAccepted`).  
12) `BatchAccepted` (Quality Assurance) — đạt quality gate, trở thành exportable (và có thể kích hoạt payout/export).

Bạn sẽ nhận ra: chỉ riêng việc viết events đã khiến bạn đặt câu hỏi đúng:
- #6 có thật cần event không hay chỉ là thuộc tính của batch? (trade-off)  
- #8 có cần không hay chỉ là telemetry? (phân biệt domain vs analytics)  
- `ReviewRequired` có phải luôn xảy ra hay chỉ khi điều kiện? (policy)  

Đó chính là “tư duy domain”: bạn bắt đầu thấy những điểm cần làm rõ.

> **CHECKLIST**  
> Khi bạn tự viết events, hãy tự hỏi:  
> 1) Event này có ý nghĩa business không, hay chỉ là kỹ thuật?  
> 2) Nếu event này bị duplicate, hệ thống có an toàn không? (gợi mở idempotency)  
> 3) Event này thuộc context nào? ai là owner?  
> 4) Event này có giúp audit trail không?  

---

## 8) Artefacts bạn nên có sau chương này

Không cần “đủ bộ DDD” ngay. Sau chương 1 (và tuần 1), bạn chỉ cần vài artefacts nhỏ nhưng đúng chỗ:

1) 1 trang “Project framing”: mục tiêu, rủi ro domain, core domains.  
2) Glossary seed 10–30 thuật ngữ (có owner context).  
3) Danh sách 10–20 business events (phiên bản thô).  
4) Danh sách 3–5 invariants “đắt nhất”.  
5) 1–3 ADRs cho trade-off lớn (sync/async, quality gate, lock TTL, payout trigger).

Nếu bạn có 5 thứ này, chương sau (Event Storming) sẽ không còn mơ hồ — bạn đã có “điểm bám” để workshop đi vào việc thật.
