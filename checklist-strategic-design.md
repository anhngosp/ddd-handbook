# Checklist — Strategic Design (Bounded Contexts, Context Map, Contracts, Ownership, Risks)

Checklist này dùng để “chốt strategic đủ để đi tiếp” — tức là đủ để vào tactical/implement một slice mà không mơ hồ.  
Nó không nhằm làm tài liệu dày. Nó nhằm đảm bảo bạn không bỏ sót những thứ “đắt tiền”.

Ví dụ tham chiếu: ADLP Strategic Design v0.2 (`design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`).

---

## 1) Bounded Contexts (BC) — đã chốt đủ rõ chưa?

- [ ] Có danh sách BC (v0) theo capability (không theo CRUD entity).
- [ ] Mỗi BC có mô tả responsibilities (in/out of scope).
- [ ] Mỗi BC có owner team (hoặc decision owner).
- [ ] Các thuật ngữ “đắt tiền” có owner context (glossary seed).
- [ ] Không có shared DB/join xuyên BC (nếu có, ghi hotspot/plan read model).

---

## 2) Context Map — quan hệ và hướng phụ thuộc đã rõ chưa?

- [ ] Có context map cho workflow đắt tiền (premium 48h hoặc workflow tương đương).
- [ ] Mỗi edge có direction (upstream/downstream).
- [ ] Mỗi edge có pattern (customer/supplier, partnership, conformist, OHS, ACL, shared kernel).
- [ ] Đã chỉ ra edge “rủi ro cao” (schema evolution, SLA, payout, compliance).

---

## 3) Contracts — edge nào nói chuyện bằng gì?

### 3.1 API contracts (sync)
- [ ] Edge nào cần sync đã được liệt kê (REST/gRPC).
- [ ] Đã xác định payload tối thiểu (tránh consumer gọi ngược DB nội bộ).
- [ ] Đã thống nhất error model cơ bản + timeout/retry policy.

### 3.2 Event contracts (async)
- [ ] Danh sách event types cho workflow đắt tiền (10–20 events v0).
- [ ] Với mỗi event “đắt tiền” có: owner context, payload tối thiểu, consumers.
- [ ] Có envelope chuẩn (event_id, version, correlation_id…).
- [ ] Có versioning rules (semver) + compatibility rules.
- [ ] Có idempotency key strategy (ít nhất cho payout/export/accept).

---

## 4) Hotspots & Decision Ownership

- [ ] Top 5 hotspots đã được ghi thành câu hỏi cụ thể.
- [ ] Mỗi hotspot có owner + due date.
- [ ] Hotspot nào là “decision đắt tiền” đã có ADR (hoặc draft).

---

## 5) Core/Supporting/Generic — đầu tư đúng chỗ chưa?

- [ ] Core domains đã xác định rõ (đầu tư tactical sâu).
- [ ] Generic domains đã có strategy “buy/standardize”.
- [ ] Team giỏi nhất đang tập trung vào core (không bị hút vào generic).

---

## 6) NFR/SLO ở mức strategic (tối thiểu)

- [ ] SLOs tối thiểu theo context hoặc theo nhóm context (API vs worker).
- [ ] Các ràng buộc compliance/audit được ghi rõ (PII, retention, export audit).
- [ ] Observability baseline: correlation_id, tracing/logging principle cho cross-context flows.

---

## 7) Gate để “đi tiếp” (Strategic Done)

Bạn có thể coi strategic “đủ” để đi tiếp khi:
- [ ] Có 1 workflow đắt tiền được mô tả bằng events và có context map tương ứng.
- [ ] Có 3–5 hotspots đắt tiền có owner và kế hoạch giải.
- [ ] Có contracts tối thiểu cho 2–3 edges quan trọng nhất.
- [ ] Có quyết định sơ bộ sync/async + idempotency ở chỗ payout/export.

