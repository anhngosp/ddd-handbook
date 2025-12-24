# Template Box — DDD Practical Handbook (dùng lại)

Tài liệu này chứa các template “dùng được ngay” khi:
- viết các chương trong handbook,
- chạy workshop (Event Storming),
- ghi ADR,
- thiết kế event schema và versioning,
- tạo checklist theo từng phần.

---

## 1) Chapter DoD (Definition of Done)

Một chương được xem là “đạt” khi có đủ:
1) **Story mở đầu** (gắn ADLP, có hậu quả thật).
2) **Giải thích khái niệm** (diễn giải + ví dụ đúng/sai).
3) **Khi nào dùng / khi nào tránh** (heuristics).
4) **Trade-offs** (được/mất, chi phí ẩn, điều kiện áp dụng).
5) **Best practices** (kèm “vì sao”).
6) **Anti-patterns** (triệu chứng → hậu quả → cách tránh).
7) **Áp dụng vào ADLP** (mapping rõ: context nào, event nào, invariant nào).
8) **Artefacts/Deliverables** (đầu ra cụ thể).
9) **Exercise có hướng dẫn** (bước nghĩ + ví dụ mẫu).

---

## 2) Callout Blocks (dùng thống nhất)

Dùng định dạng:

> **NOTE** — ghi chú quan trọng  
> **WARNING** — cảnh báo rủi ro/anti-pattern  
> **EXAMPLE** — ví dụ cụ thể  
> **CHECKLIST** — danh sách làm ngay  

---

## 3) Glossary Seed Template (Ubiquitous Language)

| Term | Định nghĩa 1–2 câu | Ví dụ đúng | Ví dụ sai | Owner context |
|---|---|---|---|---|
|  |  |  |  |  |

Gợi ý:
- Owner context giúp “đóng nghĩa”: cùng từ nhưng khác nghĩa ở context khác → tách entry.
- Mỗi định nghĩa nên có “ranh giới”: nó *không* bao gồm cái gì.

---

## 4) Domain Event Catalog Template

| Event | Owner context | Khi nào phát | Payload tối thiểu | Ai tiêu thụ | Idempotency key gợi ý |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

Gợi ý:
- “Payload tối thiểu” là để consumer chạy workflow mà **không** cần gọi ngược DB nội bộ của producer.
- Idempotency key: dùng entity id + event type + version (tuỳ policy).

---

## 5) Event Schema Template (Published Language)

### 5.1 Envelope chuẩn (khuyến nghị)

```json
{
  "event_id": "uuid",
  "event_type": "BatchAccepted",
  "occurred_at": "2025-12-20T10:20:30Z",
  "version": "1.0.0",
  "producer": "quality-assurance",
  "correlation_id": "uuid",
  "causation_id": "uuid",
  "payload": {}
}
```

### 5.2 Versioning rules (semver)
- **PATCH**: thêm field optional, không đổi nghĩa.
- **MINOR**: thêm event type mới hoặc thêm field có default/optional.
- **MAJOR**: đổi/bỏ field bắt buộc, đổi nghĩa field, đổi semantics xử lý.

### 5.3 Compatibility rules (thực dụng)
- Producer không remove field ở minor/patch.
- Consumer phải tolerate unknown fields.
- Khi cần breaking change: publish event type mới hoặc bump major + chạy song song một thời gian.

---

## 6) ADR Template (Architecture Decision Record)

```md
# ADR-XXX: <Decision Title>

**Status:** Proposed | Accepted | Superseded | Rejected  
**Date:** YYYY-MM-DD  
**Owners:** <team/person>  

## Context
Mô tả bối cảnh, constraint, và vấn đề cần quyết.

## Decision
Quyết định rõ ràng (1–3 câu). Nếu có scope, ghi scope.

## Alternatives Considered
- A: ...
- B: ...

## Consequences
### Positive
- ...

### Negative / Risks
- ...

### Mitigations
- ...

## How to Measure (optional)
KPI/SLO/metric nào cho thấy quyết định này đúng/sai.
```

---

## 7) Workshop Agenda Template

### 7.1 Kick-off (30–45 phút)
Mục tiêu: chốt kịch bản workshop + người tham gia + rủi ro domain lớn nhất.
1) 5’ — scope + mục tiêu
2) 10’ — chọn 1 kịch bản “đắt tiền nhất” (ví dụ premium 48h)
3) 15’ — liệt kê top 3–5 rủi ro domain (nếu sai là toang)
4) 10’ — chốt lịch workshop + artefacts cần chuẩn bị

### 7.2 Big Picture Event Storming (2 giờ)
1) 10’ — thống nhất kịch bản và luật chơi (event-centric, không bàn giải pháp)
2) 40’ — viết events theo timeline
3) 20’ — gắn actors/commands ở điểm nóng
4) 30’ — đánh dấu hotspots/questions + chốt glossary seed
5) 20’ — tổng hợp output + action items

---

## 8) Checklist Template (dùng để “dùng được ngay”)

```md
## Checklist: <Topic>
- [ ] <Action 1>
- [ ] <Action 2>
- [ ] <Action 3>
```

Gợi ý:
- Checklist nên ngắn, tối đa ~10–15 items.
- Mỗi item phải là hành động cụ thể, không phải khái niệm.

