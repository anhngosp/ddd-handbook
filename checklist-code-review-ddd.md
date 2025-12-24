## Checklist: DDD Code Review (Domain Purity, Boundaries, Invariants, Coupling)

Checklist này dùng khi review PR trong codebase theo DDD. Mục tiêu: giữ cho hệ thống **không trượt** về “DDD nửa mùa” sau vài tháng.

> **NOTE**  
> Checklist không thay thế cho hiểu domain. Nó là hàng rào để phát hiện sớm các lỗi kiến trúc thường gặp.

---

### 1) Boundaries & Dependency Rules
- [ ] Thay đổi nằm đúng bounded context/module (không “tiện tay” sửa chéo context)
- [ ] `domain` không import framework/ORM/HTTP/broker (không dependency ngược)
- [ ] `application` chỉ phụ thuộc `domain` + ports/interfaces, không phụ thuộc implementation infra
- [ ] Controllers/consumers mỏng (mapping + gọi handler), không chứa invariants
- [ ] Không có “shared domain objects” giữa contexts (shared kernel có lý do và governance rõ)

---

### 2) Domain Model & Invariants
- [ ] Quy tắc nghiệp vụ (invariants) nằm trong aggregate/VO/domain service, không rải if-else ở controller
- [ ] Command dẫn tới state transition rõ + domain event (nếu phù hợp)
- [ ] Domain errors/rejections có ý nghĩa nghiệp vụ (không chỉ `InvalidState`)
- [ ] Value Objects có validation/normalization đúng chỗ (không để primitives trôi nổi)
- [ ] Không tạo “God Aggregate” hoặc aggregate boundary bị lộ qua API (client phải cập nhật nhiều nơi mới xong)

---

### 3) Use Case Implementation (Application Layer)
- [ ] Use case có “implementation spine” nhất quán (map → validate → load → domain → save → publish → return)
- [ ] Transaction boundary rõ (một use case = một transaction, trừ khi có lý do)
- [ ] Concurrency strategy rõ (optimistic locking/conflict behavior)
- [ ] Command “đắt tiền” có idempotency (tránh double submit/payout/export)
- [ ] Validation tách bạch: input shape vs domain meaning (không lẫn tầng)

---

### 4) Integration & Messaging (Sync/Async)
- [ ] Không có call chain sync dài gây distributed monolith (N services cho một request)
- [ ] Event/API contracts giữ “payload tối thiểu nhưng đủ” (consumer không gọi ngược DB producer)
- [ ] Correlation ID được propagate (request → domain/integration events) cho workflow dài
- [ ] Retry policy có backoff + jitter + max attempts; lỗi permanent không retry vô hạn
- [ ] Có DLQ + owner + runbook (đặc biệt cho payout/export/quality events)

---

### 5) Event Publishing Semantics
- [ ] Không publish event trước khi commit
- [ ] Nếu workflow critical: có outbox hoặc cơ chế tương đương (write state + write outbox trong cùng transaction)
- [ ] Consumer xử lý duplicate/out-of-order (idempotency key theo business identity)
- [ ] Schema versioning có kỷ luật (semver rules, consumer tolerant unknown fields)

---

### 6) Data Access & Read Models
- [ ] Không join/read xuyên DB của context khác để “cho nhanh”
- [ ] Query phục vụ UI/report có read model/CQRS-lite nếu cần (không ép domain model phục vụ mọi query)
- [ ] Cache không phá invariants (cache reads, không cache writes theo cách “lệch nghĩa”)
- [ ] Migration/data retention có xem xét ảnh hưởng audit/compliance (nếu dữ liệu nhạy cảm)

---

### 7) API Design & Error Model
- [ ] Endpoints phản ánh use case/domain language (không CRUD bừa cho core domain)
- [ ] Error model có `error_code` semantics + phân loại retryable vs non-retryable
- [ ] Versioning/deprecation plan được cập nhật nếu có breaking change
- [ ] Không leak persistence details/IDs nội bộ qua contract

---

### 8) Testing & Observability (đủ để ngủ ngon)
- [ ] Domain invariants có unit tests (đặc biệt các rule đắt tiền)
- [ ] Có integration tests cho transaction/outbox/idempotency nếu PR chạm vào các phần này
- [ ] Contract tests được cập nhật khi thay đổi schema/API/events
- [ ] Có logs/metrics/traces tối thiểu cho workflow chạm business-critical path (theo SLO/NFR)
- [ ] PR không làm giảm khả năng debug (missing correlation_id, log thiếu domain identifiers)

---

### 9) Security & Compliance (nếu có chạm)
- [ ] Authorization theo domain/context (không chỉ “đã login”)
- [ ] Secrets không hardcode; không log dữ liệu nhạy cảm
- [ ] Retention/deletion/audit trail được giữ đúng nếu PR chạm dữ liệu nhạy cảm/financial

