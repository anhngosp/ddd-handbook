# Checklist — Tactical Design (Aggregate, Invariants, Events, Consistency, Tests)

Checklist này giúp bạn chốt tactical “đủ để code đúng” cho một bounded context/slice, không over-engineer.  
Mục tiêu: invariants rõ, boundaries rõ, events đúng nghĩa, và edge-cases được nghĩ trước.

---

## 1) Readiness (đủ điều kiện vào tactical chưa?)

- [ ] Có workflow đắt tiền (big picture 10–20 events).
- [ ] Có glossary seed và các từ “đắt tiền” đã chốt nghĩa v0 (Submitted/Accepted/Confidence/QualityScore…).
- [ ] Hotspots top 3–5 có owner và hướng giải (hoặc ADR).
- [ ] Đã chọn 1 bounded context và 1 slice (không scope quá rộng).

---

## 2) Aggregate & Invariants

- [ ] Có 1–2 aggregate roots cho slice (v0).
- [ ] Mỗi aggregate có 2–5 invariants “đắt nhất” (viết được thành câu).
- [ ] State transitions rõ (đặc biệt Submitted vs Accepted, TTL expiry).
- [ ] Không có God Aggregate (nếu mọi thứ đều nằm trong 1 aggregate, scope sai hoặc strategic sai).

---

## 3) Commands & Events (đúng nghĩa)

- [ ] Commands là “hãy làm”; events là “đã xảy ra”.
- [ ] Event names business-readable (tránh updated/changed).
- [ ] Event owner context rõ.
- [ ] Có command/event table cho happy path + failure path.

---

## 4) Consistency & Concurrency

- [ ] Chỗ nào cần strong consistency đã rõ (trong aggregate boundary).
- [ ] Chỗ nào eventual consistency (cross-context) đã rõ.
- [ ] Có idempotency keys cho events/commands đắt tiền (payout/export/accept).
- [ ] Có precedence rule cho các race (TTL expiry vs submit; concurrent assign).

---

## 5) Persistence & Transactions

- [ ] Repository interface chỉ cho aggregate roots (không leak schema).
- [ ] Factory bảo vệ invariants khi create (nếu cần).
- [ ] Unit of Work/transaction boundary rõ cho mỗi use case.
- [ ] Publish event an toàn với transaction (after commit / outbox).

---

## 6) Tests & Observability

- [ ] Unit tests cho invariants (domain tests).
- [ ] Integration tests cho repository/UoW + idempotency.
- [ ] Contract tests cho event schema (nếu cross-context).
- [ ] Correlation_id/traceability cho workflow đắt tiền.

---

## 7) Gate: Tactical Done (đủ để implement slice)

Bạn có thể coi tactical “đủ” khi:
- [ ] Viết được invariants và test được.
- [ ] Có command/event table và edge-cases chính.
- [ ] Có idempotency strategy.
- [ ] Có repository/UoW plan và publish strategy.

