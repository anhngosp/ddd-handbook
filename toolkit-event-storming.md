# Event Storming Toolkit — checklist + templates + prompts (dùng được ngay)

Toolkit này gom các “đồ nghề” thực dụng để bạn chạy Event Storming và ra artefacts có thể dùng cho Strategic/Tactical design.

Mục tiêu của toolkit:
- giúp bạn không bị mơ hồ khi bắt đầu,
- giúp workshop không trượt sang kỹ thuật,
- và giúp bạn kết thúc với output có owner + next steps.

Ví dụ xuyên suốt: ADLP (premium order 48h).

---

## 1) Checklist trước workshop (15 phút)

### 1.1 Scope & mục tiêu
- [ ] Chọn **một** workflow “đắt tiền” (VD: premium order 48h).
- [ ] Viết 1 câu mô tả workflow: “Ai muốn gì, deadline gì, quality gì”.
- [ ] Chốt mục tiêu workshop: timeline + hotspots + glossary seed (không chốt giải pháp).

### 1.2 People
- [ ] Có facilitator (người kéo nhịp).
- [ ] Có domain expert thật sự (có quyền quyết định).
- [ ] Có dev leads sẽ implement.

### 1.3 Rules
- [ ] Luật cứng: chỉ nói “điều đã xảy ra”.
- [ ] Tranh luận kỹ thuật → parking lot.
- [ ] Hotspot phải có owner + due date.

### 1.4 Board setup (Miro/FigJam/whiteboard)
- [ ] Lane Timeline
- [ ] Lane Hotspots/Questions
- [ ] Lane Glossary seed
- [ ] Parking lot (tech debates)

---

## 2) Checklist trong workshop (facilitator cheat-sheet)

### 2.1 Câu hỏi kéo đúng hướng
- “Trong thực tế, chuyện gì đã xảy ra tiếp theo?”
- “Ai là người quyết định ở bước này?”
- “Nếu bước này sai thì hậu quả gì?”
- “Cái này là event hay command?”

### 2.2 Dấu hiệu workshop đang trượt
- Ai đó nói “Kafka/REST/DB schema”.
- Event tên `updated/changed/processed`.
- Không ai ghi hotspot.

### 2.3 Cách kéo lại
- Dừng tranh luận, ghi vào parking lot.
- Quay lại timeline, hỏi “đã xảy ra điều gì?”.
- Time-box hotspot (2–3 phút), gán owner.

---

## 3) Checklist kết thúc workshop (15 phút)

- [ ] Tóm tắt workflow bằng 8–12 events (nói thành câu chuyện).
- [ ] Chốt top 5 hotspots quan trọng nhất.
- [ ] Gán owner + due date cho mỗi hotspot.
- [ ] Chốt glossary seed 10–30 terms.
- [ ] Chốt next step: process-level hay strategic design?
- [ ] Lưu board link/ảnh + action items.

---

## 4) Templates (copy/paste)

### 4.1 Hotspot card template

```md
### Hotspot: <short title>
- Question: <câu hỏi cụ thể, follow-up được>
- Why it matters: <hậu quả nếu sai>
- Owner: <ai chịu trách nhiệm trả lời>
- Due: <ngày>
- Notes: <đính kèm link/refs>
```

### 4.2 Event naming rules
- Ở thì quá khứ: `XxxCreated`, `XxxSubmitted`, `XxxAccepted`.
- Tránh: `updated`, `changed`, `processed` (trừ khi định nghĩa rõ).
- Event phải “business-readable”: business đọc hiểu được.

### 4.3 Glossary seed template

| Term | Định nghĩa 1–2 câu | Ví dụ đúng | Ví dụ sai | Owner context |
|---|---|---|---|---|
|  |  |  |  |  |

### 4.4 Process card template (process-level)

```md
### Event: <EventName>
- Actor:
- Command:
- Policy:
- Time boundaries:
- External systems:
- Failure modes / hotspots:
```

### 4.5 Design-level sheet template

```md
## Aggregate candidate: <Name>
- Responsibilities:
- Invariants:
- Commands:
- Events:
- Idempotency keys:
- Edge cases:
```

---

## 5) Prompts nhanh (dùng trong workshop)

### 5.1 Prompt để đặt event
- “Điều gì đã xảy ra tiếp theo?”
- “Khi business nói ‘xong’, điều gì đã xảy ra?”

### 5.2 Prompt để phát hiện policy
- “Theo rule nào mà quyết định bước này?”
- “Nếu tier khác thì rule khác không?”

### 5.3 Prompt để phát hiện time boundary
- “Hết bao lâu thì coi là timeout?”
- “Khi TTL hết hạn, hệ quả gì xảy ra?”

---

## 6) Ví dụ ADLP: artefacts mẫu (rút gọn)

### 6.1 Big Picture timeline (gợi ý)
`DataItemUploaded → DataItemNormalized → PrelabelCompleted → BatchCreated → BatchAssigned → BatchSubmitted → QualityEvaluated → ReviewRequired → BatchAccepted → DatasetExported`

### 6.2 Hotspots mẫu
- Accepted definition (threshold theo tier, policy versioning)
- Lock TTL & reassign policy
- Review SLA + escalation levels
- Payout trigger + idempotency

### 6.3 Glossary seed 5 terms (gợi ý)
- Batch, Segment, Submitted, Accepted, Confidence

