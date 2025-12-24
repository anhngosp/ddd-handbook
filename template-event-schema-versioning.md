# Template: Event Schema + Versioning Rules (Published Language)

Tài liệu này là mẫu “published language” cho integration events: có envelope chuẩn, rule versioning, và nguyên tắc compatibility.  
Mục tiêu: events tiến hóa mà không phá consumers (và không tạo distributed monolith).

Tham chiếu liên quan:
- Template box tổng hợp: `design/docs/0.ref/DDDPractical/templates.md`
- Chương 21/24 (events, contracts, idempotency)

---

## 1) Envelope chuẩn (khuyến nghị)

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

### Ý nghĩa các field (thực dụng)
- `event_id`: định danh duy nhất cho bản ghi event (phục vụ dedup/trace).
- `event_type`: business-readable (tránh `updated/changed`).
- `occurred_at`: thời điểm “sự kiện xảy ra” theo producer.
- `version`: version của schema (semver).
- `producer`: bounded context/service phát event.
- `correlation_id`: trace end-to-end workflow.
- `causation_id`: link tới event/command gây ra event này (hữu ích khi debug).
- `payload`: dữ liệu business tối thiểu nhưng đủ cho consumers.

---

## 2) Versioning rules (semver)

- **PATCH**: thêm field optional, không đổi nghĩa.
- **MINOR**: thêm event type mới hoặc thêm field có default/optional.
- **MAJOR**: đổi/bỏ field bắt buộc, đổi nghĩa field, đổi semantics xử lý.

> **NOTE**  
> Khi cần breaking change, ưu tiên publish **event type mới** (hoặc bump major và chạy song song) thay vì “đổi nghĩa lén”.

---

## 3) Compatibility rules (consumer-driven, thực dụng)

- Producer không remove field ở minor/patch.
- Consumer phải tolerate unknown fields.
- Không đổi semantics của field ở minor/patch.
- Khi deprecate: publish song song đủ lâu, đo adoption, rồi mới tắt.

---

## 4) Migration example: BatchAccepted V1 → V2 (song song an toàn)

**Bài toán:** cần thêm `payout_currency` vào payload để Wallet xử lý multi-currency.

**Cách làm (không breaking):**
1) Thêm field optional → bump MINOR (`1.1.0`).  
2) Publish song song `1.0.0` và `1.1.0` trong 1–2 sprint.  
3) Consumer nhận `1.1.0` trước, fallback `1.0.0`.  
4) Đo adoption → deprecate `1.0.0` sau khi tỷ lệ < 5%.  

```json
// v1.0.0 payload
{
  "batch_id": "b-123",
  "labeler_id": "l-999",
  "policy_version": "2.1.0",
  "decision_version": 3
}
```

```json
// v1.1.0 payload (additive)
{
  "batch_id": "b-123",
  "labeler_id": "l-999",
  "policy_version": "2.1.0",
  "decision_version": 3,
  "payout_currency": "USD"
}
```

**Breaking change (ví dụ cần MAJOR):**
- Đổi nghĩa `decision_version` từ “số lần quyết định” sang “policy decision id”.  
Khi đó: publish event type mới (BatchAcceptedV2) hoặc bump major và chạy song song.

---

## 5) Payload rules: “tối thiểu nhưng đủ”

Payload nên đủ để consumer chạy workflow mà **không** cần gọi ngược DB nội bộ của producer.

Nếu bạn thiếu field và consumer phải gọi back:
- bạn phá boundary,
- tăng coupling thời gian,
- tăng failure modes (và thường tạo call chain sync dài).

> **BEST PRACTICE**  
> Với events “đắt tiền” (payout/export/accept), hãy ghi rõ payload tối thiểu + idempotency key gợi ý trong event catalog.

---

## 6) Idempotency key gợi ý (cho consumers)

Nguyên tắc:
- dựa trên **business identity**, không dựa trên `event_id`.
- ví dụ: `batch_id + decision_version`, `export_id`, `payout_id`.

> **WARNING**  
> Nếu consumer không idempotent, at-least-once delivery sẽ tạo double side effects (double credit, double export).

---

## 7) Mẫu “Event Catalog entry” đi kèm (khuyến nghị)

| Event | Owner context | Khi nào phát | Payload tối thiểu | Ai tiêu thụ | Idempotency key gợi ý |
|---|---|---|---|---|---|
| BatchAccepted | Quality Assurance | Khi batch được accept | batch_id, labeler_id, policy_version, decision_version, … | Wallet, Export | batch_id + decision_version |
