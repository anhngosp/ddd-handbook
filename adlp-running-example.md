# Running Example — ADLP “Premium 48h” từ Discovery đến Production (một câu chuyện xuyên suốt)

Tài liệu này là “xương sống” để bạn đọc handbook theo kiểu thực hành: một kịch bản chạy xuyên từ domain discovery → event storming → strategic/tactical → system design → implementation → productionize.

Kịch bản được chọn: **ADLP Premium 48h** (AI Data Labeling Platform).  
Thiết kế ADLP tham chiếu nguồn mới nhất: `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`.

---

## 1) Câu chuyện business (1 phút để hiểu “đắt tiền” ở đâu)

Một khách hàng mua dataset tier **Premium** và yêu cầu giao trong **48 giờ**.  
Họ không quan tâm bạn dùng Kafka hay REST. Họ quan tâm:
- dataset đúng chất lượng (quality gate),
- giao đúng hạn (SLA),
- audit được (vì sao accepted/rejected),
- và payout đúng (không double pay, không trả sai người).

Bạn có một workflow dài (và nhiều failure modes):
1) Ingestion nhận audio + metadata
2) Prelabeling chạy model, tạo segments + confidence
3) Task Assignment tạo batch, route theo skill/rating, lock TTL
4) Labeling UI edit transcript, autosave, submit
5) Quality evaluate/review, quyết định accept/rework/reject
6) Export đóng gói dataset version, cấp quyền download (signed URL), audit
7) Wallet credit earning (và/hoặc payout workflow)

---

## 2) DDD artefacts bạn cần tạo theo thứ tự (tối thiểu nhưng đủ)

### 2.1 Discovery (đầu vào)
Mục tiêu: “hiểu đúng business reality” trước khi chọn kiến trúc.

Output tối thiểu:
- Glossary seed (10–20 terms): `design/docs/0.ref/DDDPractical/glossary.md`
- Discovery checklist để phỏng vấn đúng: `design/docs/0.ref/DDDPractical/checklist-discovery.md`
- Hotspots/Questions list (premium definition, TTL, payout trigger, audit…)

### 2.2 Event Storming (shared understanding)
Mục tiêu: biến câu chuyện thành timeline events + hotspots.

Output tối thiểu:
- Checklist chạy workshop: `design/docs/0.ref/DDDPractical/checklist-event-storming.md`
- Toolkit hỗ trợ tổng hợp: `design/docs/0.ref/DDDPractical/toolkit-event-storming.md`

Một timeline mẫu (rút gọn):
`DataItemUploaded → DataItemNormalized → PrelabelCompleted → BatchCreated → BatchAssigned → TranscriptSubmitted → QualityEvaluated → BatchAccepted → DatasetExported → EarningCredited`

### 2.3 Strategic Design (boundaries + coupling thật)
Mục tiêu: chốt bounded contexts và quan hệ tích hợp.

Output tối thiểu:
- Context catalog + context map (v0)
- Strategic checklist: `design/docs/0.ref/DDDPractical/checklist-strategic-design.md`
- ADRs cho quyết định đắt tiền (sync/async, SLO, retention, event schema…): `design/docs/0.ref/DDDPractical/template-adr.md`

Gợi ý contexts liên quan trực tiếp Premium 48h:
- Data Ingestion
- Prelabeling
- Task Assignment
- Labeling
- Quality Assurance
- Dataset Export
- Wallet & Payment

### 2.4 Tactical Design (invariants + commands/events)
Mục tiêu: chọn một slice để model đủ sâu và code đúng.

Slice gợi ý (đủ “đắt tiền” nhưng vẫn gọn):
`BatchAssigned → TranscriptSubmitted → BatchAccepted`

Output tối thiểu:
- Invariants (2–5 câu) + aggregate boundary
- Command/Event table + edge cases (duplicates, TTL expiry, race)
- Tactical checklist: `design/docs/0.ref/DDDPractical/checklist-tactical-design.md`

### 2.5 System/Solution Design (architecture + integration + data)
Mục tiêu: biến model thành hệ thống chạy được, không distributed monolith.

Output tối thiểu:
- Mapping BC → deployment units (phase-based): `design/docs/0.ref/DDDPractical/23-domain-to-architecture.md`
- Integration decisions + contracts + idempotency/DLQ: `design/docs/0.ref/DDDPractical/24-integration-design.md`
- Data ownership + read models + caching: `design/docs/0.ref/DDDPractical/25-data-infrastructure-design.md`
- Integration checklist: `design/docs/0.ref/DDDPractical/checklist-integration.md`

### 2.6 Implementation (use case → code)
Mục tiêu: giữ invariants, transaction, publish semantics đúng.

Output tối thiểu:
- API contracts đúng ngôn ngữ domain: `design/docs/0.ref/DDDPractical/27-api-design-theo-domain.md`
- Implementation spine (handlers, UoW, outbox): `design/docs/0.ref/DDDPractical/28-tu-use-case-den-code.md`
- Code organization + dependency rules: `design/docs/0.ref/DDDPractical/29-to-chuc-code-theo-ddd.md`
- Testing strategy + contract tests + idempotency tests: `design/docs/0.ref/DDDPractical/30-testing-trong-ddd.md`
- Code review checklist: `design/docs/0.ref/DDDPractical/checklist-code-review-ddd.md`

### 2.7 Productionize (NFR by design)
Mục tiêu: không “mù” khi hệ thống phân tán chạy thật.

Output tối thiểu:
- NFR pack theo context + SLO + DR/retention: `design/docs/0.ref/DDDPractical/26-nfr-by-design.md`
- Dashboard/alerts + DLQ/runbook + correlation_id end-to-end

---

### 2.8 End-to-end trace example (correlation_id xuyên workflow)

**Scenario:** labeler nhận batch premium và submit, quality accept, export + payout.

```text
correlation_id = 8f52-...-a1

Assignment Service
  Command: ClaimBatch(batch_id=b-123, labeler_id=l-88)
  Event: BatchAssigned { batch_id=b-123, labeler_id=l-88, correlation_id }

Labeling Service
  Command: SubmitTranscript(batch_id=b-123, idempotency_key=b-123:v1)
  Event: BatchSubmitted { batch_id=b-123, correlation_id }

Quality Service
  Command: EvaluateBatch(batch_id=b-123, policy_version=2.1)
  Event: BatchAccepted { batch_id=b-123, decision_version=3, correlation_id }

Wallet Service
  Consume: BatchAccepted
  Side effect: CreditEarning(earning_id=e-456, correlation_id)

Export Service
  Consume: BatchAccepted
  Side effect: DatasetExported(export_id=ex-789, correlation_id)
```

Nếu bạn không truy được chuỗi này bằng một `correlation_id`, hệ thống sẽ “mù” khi incident.

---

## 3) “Starter Pack” 1 ngày: nếu bạn cần làm rất nhanh

Nếu bạn cần một kế hoạch 1 ngày để “đủ để code đúng slice”:
1) 60’ discovery: glossary seed + 5 hotspots (premium definition, TTL, accept criteria, payout trigger, audit)
2) 120’ big picture event storming: timeline + hotspots + glossary update
3) 90’ strategic mini: contexts liên quan + context map v0 + 2 ADRs (integration + SLO)
4) 90’ tactical mini: chọn slice + invariants + command/event + idempotency keys

Kết thúc 1 ngày, bạn phải có “đầu vào đủ” cho implementation spine của 1–2 use cases.

---

## 4) Câu hỏi tự kiểm: bạn có đang làm đúng nhịp không?

- Bạn có thể kể lại workflow bằng 8–12 events không? Nếu không, chưa đủ shared understanding.
- Bạn có “định nghĩa đắt tiền” (Submitted/Accepted/TTL/Score/Payout) được chốt nghĩa chưa?
- Consumer có phải gọi ngược DB producer không? Nếu có, boundary/contract đang sai.
- Nếu nhận duplicate `BatchAccepted`, hệ thống có double credit/double export không?
- Bạn có trace được 1 premium batch end-to-end bằng correlation_id không?
