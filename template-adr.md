# Template: ADR (Architecture Decision Record)

Tài liệu này là mẫu ADR “dùng được ngay” để ghi lại các quyết định đắt tiền (strategic/system/NFR).  
Bạn có thể tham chiếu thêm các quyết định trong ADLP strategic design (`design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`).

> **NOTE**  
> ADR tốt không dài. Nó rõ “vì sao”, “đổi lại cái gì”, và “đo bằng gì”.

---

## Khi nào nên viết ADR?

- Quyết định ảnh hưởng nhiều bounded contexts (integration, contracts, event strategy).
- Quyết định ảnh hưởng vận hành lâu dài (SLO, observability, DR, retention).
- Quyết định “khó đảo ngược” (database ownership, service boundaries).
- Quyết định có trade-off rõ và có rủi ro nếu hiểu sai.

---

## Mẫu ADR (copy/paste)

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

## Best practices (để ADR có giá trị)

- Viết “consequences” thật: chi phí vận hành, coupling, rủi ro compliance, effort của team.
- Chỉ chọn 2–4 alternatives thật sự; đừng liệt kê 10 lựa chọn cho có.
- Nếu decision liên quan contracts/events: ghi rõ versioning/deprecation.
- Nếu decision liên quan SLO/NFR: ghi rõ SLI đo ở đâu và alerting policy ở mức tối thiểu.

