# Chương 33 — Checklist bắt đầu dự án mới với DDD: từ “zero” đến productionize (đi đúng ngay từ ngày đầu)

Nếu bạn chỉ nhớ một chương để áp dụng ngay, hãy nhớ chương này.

Nhiều dự án thất bại không phải vì thiếu người giỏi, mà vì bắt đầu sai nhịp:
- nhảy vào code quá sớm,
- chưa có ngôn ngữ chung,
- chưa chốt ownership,
- chưa chốt contracts,
- và đến lúc production thì mới vỡ ra rằng NFR/observability không hề tồn tại.

Chương 33 là một checklist end-to-end để bạn:
1) khởi động dự án đúng thứ tự,
2) tạo artefacts tối thiểu nhưng “dùng được ngay”,
3) biết rõ “đến đâu thì đủ để code”, và “đến đâu thì đủ để production”.

Ví dụ xuyên suốt: ADLP. Bạn có thể thay ADLP bằng domain khác, nhưng checklist vẫn dùng được.

---

## Bạn sẽ nhận được gì sau chương này?

1) Một checklist đầy đủ theo 5 chặng: Discovery → Event Storming → Strategic Design → Trước khi code → Trước khi productionize.  
2) Mỗi chặng có output/artefacts rõ (tài liệu, bảng, quyết định).  
3) Những câu hỏi quan trọng để tránh hiểu sai domain.  
4) Trade-offs: khi nào làm “lite”, khi nào phải làm “đủ bài”.  
5) Exercise có hướng dẫn: áp checklist vào một kịch bản ADLP (Premium 48h) và tự kiểm.

---

## 1) Nguyên tắc dùng checklist: đừng biến nó thành “thủ tục”

Checklist chỉ có giá trị nếu:
- mỗi item có owner,
- mỗi item tạo ra artefact thật,
- và checklist phục vụ một kịch bản “đắt tiền”.

> **NOTE**  
> Nếu bạn tick được 80% mà vẫn không ai nói cùng một ngôn ngữ, coi như bạn chưa làm gì.

---

## 2) Checklist — Discovery (khám phá domain)

Mục tiêu: hiểu “business reality” đủ để không code sai hướng.

> **CHECKLIST**
> - [ ] Chọn 1–2 kịch bản đắt tiền nhất (ví dụ ADLP: Premium 48h, Payout/Dispute)  
> - [ ] Chốt domain experts thật sự (người ra quyết định/đánh đổi)  
> - [ ] Tạo glossary seed (10–20 thuật ngữ) + owner context (v0)  
> - [ ] Viết 5–10 câu hỏi “đinh” để phỏng vấn (dùng `design/docs/0.ref/DDDPractical/checklist-discovery.md`)  
> - [ ] Ghi hotspots/questions (mâu thuẫn/ambiguous/điểm rủi ro) + owner + deadline trả lời  
> - [ ] Xác định Core/Supporting/Generic (đầu tư theo gradient)  

**Artefacts tối thiểu**
- Glossary seed (bảng thuật ngữ v0)
- Hotspots & Questions list
- Core/Supporting/Generic notes

**Đừng làm**
- đừng bắt đầu bằng ERD hoặc schema.
- đừng bắt đầu bằng microservices diagram.

---

## 3) Checklist — Event Storming (tạo shared understanding)

Mục tiêu: biến câu chuyện nghiệp vụ thành timeline events + ngôn ngữ chung.

> **CHECKLIST**
> - [ ] Chạy Big Picture Event Storming cho kịch bản đắt tiền (2–3 giờ)  
> - [ ] Chốt timeline events (10–30 events) và actor/command ở điểm nóng  
> - [ ] Đánh dấu hotspots/questions ngay trên board và chốt follow-up  
> - [ ] Nếu cần: đi Process-level cho 1–2 flow quan trọng  
> - [ ] Nếu sẵn sàng: đi Design-level cho 1 context core để chốt invariants v0  
> - [ ] Tổng hợp artefacts sau workshop (dùng `design/docs/0.ref/DDDPractical/toolkit-event-storming.md`)  

**Artefacts tối thiểu**
- Event timeline (ảnh/markdown)
- Hotspots list + action items
- Glossary update (v1)
- Domain event catalog seed (nếu đã rõ owner/consumers)

**Trade-off**
- Nếu deadline gấp, bạn vẫn phải làm Big Picture tối thiểu. Đó là “đầu tư bắt buộc”.

---

## 4) Checklist — Strategic Design (Bounded Context + Context Map)

Mục tiêu: chốt ranh giới ngôn ngữ/ownership và quan hệ tích hợp (coupling thật).

> **CHECKLIST**
> - [ ] Chốt bounded contexts (v0) theo capability + ownership + data lifecycle  
> - [ ] Viết context map (ai là upstream/downstream, pattern: ACL/OHS/Conformist/…)  
> - [ ] Chốt ownership: context nào sở hữu sự thật của dữ liệu nào (database ownership)  
> - [ ] Chốt integration style theo mối quan hệ (sync/async, contracts)  
> - [ ] Viết ADR cho các quyết định đắt tiền (integration, SLO, retention, event strategy)  
> - [ ] Chốt SLO theo context (tham chiếu ADR-007 trong ADLP)  
> - [ ] Rà bằng checklist: `design/docs/0.ref/DDDPractical/checklist-strategic-design.md`  

**Artefacts tối thiểu**
- Bounded context catalog (tên, mô tả, owner)
- Context map (diagram + notes)
- Danh sách ADRs (ít nhất 3–7 quyết định đắt tiền)
- SLO table per context (v0)

**Trade-off**
- Bạn không cần “perfect context map”. Bạn cần “đủ đúng để không tách sai”.

---

## 5) Checklist — Trước khi code (Tactical + Implementation Spine)

Mục tiêu: chọn 1 lát triển khai, model đủ để code đúng và test được.

> **CHECKLIST**
> - [ ] Chọn 1 bounded context core + 1 slice workflow (đừng chọn cả hệ thống)  
> - [ ] Chốt 2–5 invariants đắt tiền cho slice đó  
> - [ ] Chọn aggregate boundary v0 và state machine v0  
> - [ ] Chốt commands → events (domain/integration) + error model  
> - [ ] Chốt transaction boundary + concurrency strategy (optimistic locking, idempotency)  
> - [ ] Chốt event schema envelope + versioning rules (theo `design/docs/0.ref/DDDPractical/templates.md`)  
> - [ ] Chốt code organization + dependency rules (Chương 29)  
> - [ ] Chốt test strategy tối thiểu (Chương 30)  
> - [ ] Rà bằng checklist tactical: `design/docs/0.ref/DDDPractical/checklist-tactical-design.md`  

**Artefacts tối thiểu**
- Aggregate sketch (commands, events, invariants)
- Error taxonomy v0
- Event schema v0 (published language)
- “Implementation spine” cho 1–2 use cases (Chương 28)

**Đừng làm**
- đừng code UI trước khi chốt error semantics và state transitions cho use cases core.

---

## 6) Checklist — Trước khi productionize (NFR + Reliability + Governance)

Mục tiêu: đảm bảo hệ thống không chỉ “chạy”, mà còn “vận hành được”.

> **CHECKLIST**
> - [ ] Chốt NFR pack theo context: scalability/observability/security/compliance/performance/resilience (Chương 26)  
> - [ ] Có correlation_id end-to-end + tracing/logging conventions  
> - [ ] Có retry/DLQ policy + owner + runbook cho events quan trọng  
> - [ ] Có outbox/inbox semantics (nếu event-driven) + tests bảo vệ  
> - [ ] Có contract tests cho API/events + deprecation plan  
> - [ ] Có DR plan (RPO/RTO) tối thiểu cho contexts critical  
> - [ ] Có data retention & deletion policy cho dữ liệu nhạy cảm  
> - [ ] Rà integration bằng `design/docs/0.ref/DDDPractical/checklist-integration.md`  

**Artefacts tối thiểu**
- Dashboard + alerts theo SLO
- Runbooks (DLQ spike, backlog spike, payout mismatch)
- Retention matrix + audit plan
- DR plan + cadence diễn tập

---

## 7) Trade-offs: khi nào “DDD-lite” là đủ?

DDD-lite hợp lý khi:
- domain đơn giản, ổn định,
- thay đổi ít,
- rủi ro sai nghiệp vụ thấp,
- SLA/NFR không nghiêm ngặt.

Nhưng trong các miền như marketplace (ADLP), payout/quality/compliance:
- hiểu sai domain = mất tiền/mất uy tín,
- eventual consistency + event-driven = dễ “sai âm thầm”,

…nên bạn cần đầu tư đủ bài ở core contexts.

> **BEST PRACTICE**  
> Dùng checklist như “core domain gradient”: core làm đủ, generic làm tối giản nhưng vẫn giữ boundaries và contracts.

---

## 8) Timeline ước lượng (tham khảo)

Timeline dưới đây mang tính “điểm xuất phát” cho dự án mới (team 5–8 người):

| Phase | Mục tiêu chính | Thời lượng gợi ý |
|---|---|---|
| Discovery | Chốt problem statement + core domains | 1–2 tuần |
| Event Storming | Timeline events + hotspots + glossary seed | 2–3 ngày |
| Strategic Design | Bounded contexts + context map v0 | 1–2 tuần |
| Tactical Slice | 1–2 aggregates + invariants + events | 1–2 tuần |
| System/Integration | Contracts + outbox + SLOs | 1–2 tuần |
| Productionize | Observability + runbooks + alerts | 1 tuần |

Nếu domain rất mới hoặc có nhiều stakeholders, hãy cộng thêm 30–50% thời gian cho discovery.

---

## 9) Exercise (có hướng dẫn) — Áp checklist vào ADLP “Premium 48h”

### Bài toán
Bạn chuẩn bị khởi động ADLP với cam kết Premium 48h. Hãy dùng checklist chương này để tạo “starter pack” trong 1–2 ngày workshop + 1 ngày tổng hợp.

### Cách làm (gợi ý từng bước)

**Bước 1 — Discovery (60 phút)**  
Chốt 10 thuật ngữ + 5 hotspots + core contexts liên quan Premium 48h.

**Bước 2 — Big Picture Event Storming (2 giờ)**  
Vẽ timeline events end-to-end (ingest → prelabel → assign → label → quality → export/payout).

**Bước 3 — Strategic (90 phút)**  
Chốt boundaries v0 cho 4 contexts: Prelabeling, Assignment, Quality, Export/Wallet.  
Vẽ context map + integration style sơ bộ.

**Bước 4 — Trước khi code (90 phút)**  
Chọn slice: `BatchAssigned → BatchSubmitted → BatchAccepted`.  
Chốt invariants + commands/events + error model + idempotency keys.

**Bước 5 — Productionize (60 phút)**  
Chốt 5 metrics/alerts + DLQ policy + runbook “Premium backlog”.

### Output mong đợi
1) 1 trang context map (v0) + ownership notes.  
2) 1 domain event catalog seed (10 events).  
3) 2 ADRs: integration choice + SLO/NFR pack.  
4) 1 checklist productionize (runbooks + alerts) cho Premium 48h.
