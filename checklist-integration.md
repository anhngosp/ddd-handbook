## Checklist: Integration Design (Sync/Async, Contracts, Reliability)

Checklist này dùng khi bạn chốt integration giữa các bounded contexts (REST/gRPC hoặc events). Mục tiêu: **tránh distributed monolith**, tránh “vỡ âm thầm”, và làm cho integration **có thể vận hành được**.

> **NOTE**  
> Nếu một item không có owner (ai chịu trách nhiệm) thì coi như chưa làm.

---

### 1) Scope & Ownership
- [ ] Đã xác định rõ producer/consumer và **owner context** của dữ liệu/sự thật nghiệp vụ
- [ ] Đã chốt “source of truth” (consumer không được gọi ngược DB nội bộ của producer)
- [ ] Đã ghi rõ integration phục vụ use case nào (kịch bản cụ thể, không mơ hồ)
- [ ] Đã thống nhất “định nghĩa đúng” (ubiquitous language) cho các từ khóa trong contract

---

### 2) Chọn Sync vs Async (không theo sở thích)
- [ ] Nếu dùng **sync**: có lý do (user-facing, cần response ngay, chấp nhận time coupling)
- [ ] Nếu dùng **async**: đã xác định SLA “end-to-end latency” và cơ chế eventual consistency
- [ ] Đã nêu rõ failure mode: fail-fast vs degrade vs retry (không “để production quyết định”)
- [ ] Đã cân nhắc “blast radius” (lỗi một context có kéo context khác chết dây chuyền không)

---

### 3) API Contract (REST/gRPC)
- [ ] Endpoint/operation names phản ánh domain (không CRUD bừa)
- [ ] Có error model rõ (mã lỗi, semantics, retryable vs non-retryable)
- [ ] Có timeout budget và retry policy ở client (không retry vô hạn)
- [ ] Có rate limiting/quotas nếu public hoặc shared dependency
- [ ] Có versioning strategy (path/header/semantic) và deprecation plan

---

### 4) Event Contract (Published Language)
- [ ] Event có **envelope chuẩn**: `event_id`, `event_type`, `occurred_at`, `version`, `producer`, `correlation_id`, `causation_id`
- [ ] Schema versioning theo semver + consumer tolerant unknown fields
- [ ] Payload “tối thiểu nhưng đủ”: consumer xử lý workflow mà không gọi ngược DB producer
- [ ] Đã ghi rõ ordering expectations (nếu có) và “out-of-order handling”
- [ ] Đã ghi rõ retention của topic/stream và replay policy (nếu cần)

---

### 5) Idempotency & Duplicates (bắt buộc cho async, khuyến nghị cho sync)
- [ ] Có idempotency key theo business identity (không phụ thuộc event_id)
- [ ] Consumer xử lý duplicate an toàn (at-least-once semantics)
- [ ] Có dedup store hoặc check “already applied” theo entity/version
- [ ] Side effects quan trọng (payout/export) có cơ chế chống double-spend/double-export

---

### 6) Outbox / Delivery Semantics
- [ ] Nếu publish event từ DB change: có outbox hoặc cơ chế tương đương để tránh “write succeed, publish fail”
- [ ] Đã xác định delivery semantics (at-least-once) và system behavior khi publish/consume fail
- [ ] Có cơ chế backpressure khi consumer lag (pause, scale, shed load)

---

### 7) Retry, DLQ, Poison Messages
- [ ] Retry có exponential backoff + jitter + max attempts
- [ ] Có DLQ với threshold rõ + owner xử lý + SLA xử lý DLQ
- [ ] Có phân loại lỗi: transient vs permanent (không retry lỗi dữ liệu mãi mãi)
- [ ] Có runbook: “DLQ tăng đột biến” (bước kiểm tra + quyết định requeue/skip/fix-forward)

---

### 8) Observability & Traceability (để debug end-to-end)
- [ ] Correlation ID đi xuyên request + events (propagate qua headers/envelope)
- [ ] Có metrics tối thiểu: success/fail, latency, retry count, DLQ size, consumer lag
- [ ] Có structured logs chứa domain identifiers (batch_id, dataset_id, export_id…)
- [ ] Có dashboard/alert theo SLO (không chỉ theo CPU/memory)

---

### 9) Security & Access Boundaries
- [ ] Service-to-service auth (mTLS/JWT) và authorization theo context
- [ ] Sensitive fields được mã hóa/mask trong logs và payload nếu cần
- [ ] Secrets management (không hardcode), rotation policy
- [ ] External integrations đi qua ACL (anti-corruption layer) nếu cần

---

### 10) Backward Compatibility & Migration
- [ ] Có kế hoạch rollout: canary/dual-write/dual-consume (nếu breaking change)
- [ ] Có thời gian “chạy song song” cho schema/event version mới
- [ ] Có tiêu chí dừng version cũ (deprecation) và cách đo “còn ai dùng không”

---

### 11) Anti-patterns (tự kiểm tra nhanh)
- [ ] Không join/read xuyên DB context khác để “cho nhanh”
- [ ] Không publish “full row/full aggregate” làm payload mặc định
- [ ] Không dùng retry để che giấu lỗi domain (thiếu idempotency, thiếu validation)
- [ ] Không dùng sync call chain dài (N+1 services) cho một request user
- [ ] Không để “event như log kỹ thuật” (event names không business-readable)

