# Quick Reference Card — DDD Practical (1 page)

Mục tiêu: một trang nhắc nhanh để team dùng trong workshop, review, và design session.

---

## 1) Strategic Design (quick checks)

- **Core/Supporting/Generic**: core = lợi thế cạnh tranh; generic = buy/integrate.
- **Bounded Context**: ranh giới ngôn ngữ + ownership + lifecycle dữ liệu.
- **Context Map patterns**:
  - Partnership: đồng hành, cùng roadmap.
  - Customer/Supplier: upstream giữ contract.
  - Conformist: downstream chấp nhận model upstream.
  - ACL: dịch/che giữa domain và external.
  - OHS: mở API có published language.

---

## 2) Tactical Design (quick checks)

- **Aggregate**: bảo vệ invariants trong 1 transaction.
- **Entity**: có identity + lifecycle.
- **Value Object**: immutable, so sánh theo value.
- **Domain Event**: “điều đã xảy ra” trong context.
- **Rule of thumb**: aggregate nhỏ, invariants đắt tiền nằm trong aggregate.

---

## 3) Integration (quick checks)

- **Sync vs Async**:
  - Sync cho query/short command.
  - Async cho workflow dài + eventual consistency.
- **Outbox**: publish sau commit.
- **Idempotency**: key theo business identity.
- **Contracts**: OpenAPI/events versioned + deprecation plan.
- **Observability**: correlation_id + DLQ + runbook.

---

## 4) “Stop-the-line” questions (review nhanh)

1) Invariant nào được bảo vệ ở đâu? Có test không?
2) Contract nào bị ảnh hưởng? Versioning/deprecation plan?
3) Consumer có phải gọi ngược DB producer không?
4) Event publish trước commit không?
5) Nếu event duplicate thì side effect có bị double không?

---

## 5) Minimum artefacts (để không trượt)

- Glossary seed + owner
- Event timeline + hotspots
- Context map v0
- 1 slice tactical (aggregate + invariants)
- Contract definitions (API/events)
- Test plan cho invariants + outbox + idempotency
