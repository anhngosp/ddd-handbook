# Chương 34 — Kết luận: DDD là hành trình, không phải đích đến

Bạn đã đi một vòng từ “hiểu domain” đến “chốt contracts”, từ “event storming” đến “NFR by design”, từ “tổ chức code” đến “testing”. Nếu có một điều quan trọng nhất cần mang theo sau handbook này, thì đó là:

> DDD không phải là bộ kỹ thuật để làm hệ thống “trông đẹp”.  
> DDD là kỷ luật để làm hệ thống **đúng nghĩa**, **đổi được**, và **vận hành được**.

Trong đời thật, bạn sẽ luôn bị kéo khỏi đường đúng: deadline, pressure, thiếu domain expert, legacy, incident. DDD không loại bỏ những thứ đó. DDD cho bạn một “la bàn” để quay lại.

---

## Bạn sẽ nhận được gì sau chương này?

1) Một cách đóng gói “tinh thần DDD thực dụng” để bạn dùng lâu dài.  
2) Những nguyên tắc tối thiểu để DDD không trượt (và những thứ nên bỏ qua).  
3) Áp dụng vào ADLP: vì sao DDD là lựa chọn hợp lý cho một marketplace có workflows dài và rủi ro sai âm thầm.  
4) Một bài tập nhỏ: viết “DDD charter” 1 trang cho đội của bạn (để giữ kỷ luật).

---

## 1) DDD là công cụ tư duy: đặt câu hỏi đúng trước khi chọn giải pháp

DDD không bắt đầu bằng diagram. Nó bắt đầu bằng câu hỏi:
- “Thứ gì là thật trong business?” (events, decisions, policies)
- “Ngôn ngữ nào phải thống nhất?” (ubiquitous language)
- “Ranh giới nào giúp đổi ít đau?” (bounded context)
- “Coupling thật nằm ở đâu?” (integration contracts)
- “Rule nào đắt nhất?” (invariants, correctness)

Khi bạn đặt câu hỏi đúng, giải pháp kỹ thuật (REST/events, monolith/microservices, cache/read model) tự nhiên trở nên “có lý do”, thay vì “theo trend”.

---

## 2) Giá trị lâu dài của DDD: không phải code, mà là “khả năng tiến hóa”

Trong một hệ thống sống 1–3 năm, điều chắc chắn nhất là:
1) domain sẽ đổi,
2) team sẽ đổi,
3) hạ tầng sẽ đổi,
4) expectation của người dùng sẽ đổi.

DDD giúp bạn giữ ba thứ:

### 2.1 Giữ ngôn ngữ (để team không hiểu sai)

Khi glossary sống, event names business-readable, error codes có semantics, bạn giảm “chi phí hiểu sai”.

### 2.2 Giữ ranh giới (để đổi đúng chỗ)

Khi bounded contexts rõ và contracts có kỷ luật, thay đổi localize và blast radius giảm.

### 2.3 Giữ vận hành (để production không dạy bạn bằng incident)

Khi NFR/SLO, outbox/idempotency, correlation_id, DLQ/runbook có từ đầu, bạn không “mù” trong hệ thống phân tán.

---

## 3) Vai trò của Architect & Dev: DDD sống nhờ nhịp làm việc, không sống nhờ tài liệu

### 3.1 Architect: người giữ “đường ray”

Vai trò quan trọng nhất không phải là “vẽ đúng”. Mà là:
- chọn đúng nơi cần đầu tư (core domain gradient),
- giữ kỷ luật contracts/ownership,
- và giúp team ra quyết định trade-off minh bạch.

Trong ADLP, architect phải đặc biệt canh:
- boundaries giữa Quality ↔ Wallet/Export,
- integration semantics (idempotency/outbox),
- và SLO per context (ADR-007).

### 3.2 Developer: người biến luật thành code có thể tin

Dev giữ DDD bằng hành động nhỏ mỗi ngày:
- đặt tên đúng theo ngôn ngữ domain,
- đưa invariants vào aggregate/VO,
- viết unit tests cho rule đắt tiền,
- không đi tắt qua DB context khác,
- và không “bắn event” trước commit.

> **NOTE**  
> DDD thất bại thường không phải vì “thiếu kiến trúc”. Nó thất bại vì “thiếu kỷ luật nhỏ mỗi PR”.

---

## 4) Áp dụng vào ADLP: vì sao DDD là một khoản đầu tư hợp lý

ADLP là một domain dễ “sai âm thầm”:
- workflow dài (ingest → prelabel → assign → label → quality → export/payout),
- marketplace semantics (fairness, ratings, disputes),
- compliance/privacy (audio/transcript),
- và NFR/SLO khác nhau theo context.

DDD giúp ADLP ở 3 điểm:
1) **Đúng nghĩa**: quality decision, payout, lock TTL là “luật”, không phải “code tùy hứng”.  
2) **Đổi được**: thêm modality, thêm tier, đổi policy scoring mà không rewrite.  
3) **Vận hành được**: event processing SLO, correlation_id, DLQ/runbook, retention/DR.

Nếu bạn làm ADLP bằng CRUD thuần và shared DB, hệ thống có thể chạy nhanh ở MVP, nhưng chi phí sai/đổi/điều tra sẽ bùng lên khi scale.

---

## 5) Những thứ bạn nên giữ (và những thứ bạn nên bỏ qua)

### 5.1 Giữ (bắt buộc nếu muốn DDD “sống”)

- 1–2 kịch bản đắt tiền làm trục xuyên suốt (ví dụ Premium 48h)
- Glossary + event catalog sống (owner rõ)
- Bounded contexts + context map (kỷ luật contracts)
- Implementation spine cho use cases (Chương 28)
- Outbox/idempotency cho workflow đắt tiền
- SLO/NFR theo context + observability conventions
- Test strategy “đúng điểm đau”

### 5.2 Bỏ qua (đừng lãng phí)

- Tạo framework nội bộ khi chưa cần
- Microservices hóa mọi thứ sớm
- Abstraction/Interface mọi nơi
- UML/diagram cho “đẹp slide” nhưng không gắn artefacts/thực thi

> **BEST PRACTICE**  
> DDD tốt là DDD giúp bạn ship đều, ít incident, và refactor không sợ. Không phải DDD có nhiều class.

---

## 6) Artefacts/Deliverables: “DDD starter pack” 1 trang cho đội của bạn

Bạn có thể dùng danh sách này như “hợp đồng nội bộ”:
1) 1 kịch bản đắt tiền + timeline events
2) glossary v1 + owner
3) context map v0 + integration style
4) 2–3 ADRs đắt tiền (SLO, integration, retention)
5) implementation spine + code organization rules
6) testing strategy + contract tests
7) NFR by design pack + runbooks

---

## 7) Exercise (có hướng dẫn) — Viết “DDD Charter” 1 trang cho team của bạn

### Bài toán
Bạn muốn DDD không trượt sau 6 tháng. Hãy viết một “DDD Charter” 1 trang: những luật tối thiểu và nhịp làm việc để giữ kỷ luật.

### Cách làm (gợi ý từng bước)

**Bước 1 — Chọn 1 kịch bản đắt tiền làm trục**  
Ví dụ ADLP: Premium 48h.

**Bước 2 — Chọn 5 luật kỹ thuật tối thiểu**  
Gợi ý:
- domain không import framework
- publish events không trước commit (outbox cho workflow đắt tiền)
- consumer phải idempotent
- correlation_id end-to-end
- không join/read DB xuyên context

**Bước 3 — Chọn nhịp continuous discovery**  
Ví dụ: mỗi 2 tuần 60–90 phút mini workshop + cập nhật glossary/event catalog.

**Bước 4 — Chọn 3 artefacts bắt buộc cập nhật**  
Glossary, event catalog, context map/ADR.

**Bước 5 — Chọn cách enforce trong code review**  
Dùng checklist: `design/docs/0.ref/DDDPractical/checklist-code-review-ddd.md`.

### Output mong đợi
1) Charter 1 trang (markdown) gồm: mục tiêu, luật, cadence, artefacts, enforcement.  
2) Một đoạn “why”: vì sao charter này phù hợp với domain của bạn (hoặc ADLP).

---

## 8) Further reading & cộng đồng (đi sâu sau handbook)

**Books (nền tảng)**
- “Domain-Driven Design” — Eric Evans (Blue Book)  
- “Domain-Driven Design Distilled” — Vaughn Vernon  
- “Implementing Domain-Driven Design” — Vaughn Vernon  
- “Domain Modeling Made Functional” — Scott Wlaschin  
- “Event Storming” — Alberto Brandolini  

**Blogs/Essays (thực dụng)**
- Martin Fowler (DDD, microservices, event-driven)  
- Vaughn Vernon blog (DDD tactical/strategic)  
- Greg Young (event sourcing, CQRS)  

**Advanced topics (khi đã vững)**
- CQRS + Event Sourcing (khi audit/replay là core)  
- Saga/Process Manager patterns  
- Outbox/Inbox + exactly-once semantics (thực dụng)  
- Schema evolution + contract testing (events/API)  

---

## Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Bạn có 1 kịch bản đắt tiền làm “trục” để ra quyết định (VD: Premium 48h)  
> - [ ] Bạn giữ glossary/event catalog/context map như tài sản sống (có owner)  
> - [ ] Bạn có kỷ luật contracts/versioning/idempotency cho workflow đắt tiền  
> - [ ] Bạn có checklist code review DDD để giữ hệ thống không trượt  
> - [ ] Bạn có “DDD charter” 1 trang để truyền lại cho người mới và giữ nhịp  
