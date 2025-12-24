# Template: Workshop Agenda + Output Artefact Pack (Event Storming)

Tài liệu này giúp bạn vừa “chạy workshop đúng nhịp”, vừa “đóng gói output” để dùng ngay cho Strategic/Tactical/System design.

Tham chiếu:
- Checklist end-to-end: `design/docs/0.ref/DDDPractical/checklist-event-storming.md`
- Toolkit: `design/docs/0.ref/DDDPractical/toolkit-event-storming.md`

---

## 1) Agenda mẫu (2–3 giờ) — Big Picture Event Storming

### 1.1 Kick-off (10 phút)
1) Nhắc one-liner (kịch bản đắt tiền, deadline, hậu quả nếu fail)
2) Nhắc luật chơi: event-centric, không bàn giải pháp, tranh luận → parking lot
3) Chốt decision owners (ai quyết định threshold/TTL/payout trigger…)

### 1.2 Dựng timeline events (40 phút)
1) Viết events theo thứ tự thời gian (past tense, business-readable)
2) Timebox tranh luận “có bước này không?” → ghi hotspot

### 1.3 Gắn actor/command/policy (30 phút)
1) Ai kích hoạt?
2) Command là gì?
3) Policy/rule quyết định ra sao?

### 1.4 Hotspots & Questions (20 phút)
1) Chốt top hotspots (đừng để 30 cái)
2) Mỗi hotspot có owner + due date + next action

### 1.5 Glossary seed (15 phút)
1) Chốt 10–30 terms (v0)
2) Mark ambiguous terms, owner context (nếu đã rõ)

### 1.6 Tổng hợp & next steps (10 phút)
1) Kể lại workflow bằng 8–12 events
2) Chốt “sau workshop làm gì” (process-level/design-level/ADR/strategic)

---

## 2) Output Artefact Pack — cấu trúc “nộp bài” sau workshop

Mục tiêu của pack: bất kỳ ai không tham dự workshop vẫn hiểu và dùng được.

### 2.1 Danh sách artefacts tối thiểu
1) **Timeline tóm tắt** (8–20 events) + scope note  
2) **Hotspots/Questions list** (question, why, owner, due, status)  
3) **Glossary seed v1** (terms, definitions, owner context)  
4) **Event catalog seed** (owner context, consumers, payload tối thiểu, idempotency key gợi ý)  
5) **Decisions list**: cái nào cần ADR, cái nào đã chốt ngay  

> **NOTE**  
> Nếu bạn chạy đến process-level/design-level, hãy thêm: aggregate candidates sheet + invariants + edge cases.

### 2.2 Cấu trúc folder gợi ý (trong repo)

```text
design/docs/0.ref/DDDPractical/workshops/<YYYY-MM-DD>-<scenario>/
  00-scope.md
  01-timeline.md
  02-hotspots.md
  03-glossary.md
  04-event-catalog.md
  05-next-steps.md
  assets/
    board-snapshot.png
```

### 2.3 Nội dung từng file (mẫu ngắn)

**00-scope.md**
- Scenario: …
- In-scope / Out-of-scope: …
- Decision owners: …

**01-timeline.md**
- Timeline (events): …
- Notes: …

**02-hotspots.md**
```md
### Hotspot: <short title>
- Question: <câu hỏi cụ thể, follow-up được>
- Why it matters: <hậu quả nếu sai>
- Owner: <ai chịu trách nhiệm trả lời>
- Due: <ngày>
- Next action: <ADR/workshop/spike>
```

**03-glossary.md**
| Term | Định nghĩa 1–2 câu | Ví dụ đúng | Ví dụ sai | Owner context |
|---|---|---|---|---|
|  |  |  |  |  |

**04-event-catalog.md**
| Event | Owner context | Khi nào phát | Payload tối thiểu | Ai tiêu thụ | Idempotency key gợi ý |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

**05-next-steps.md**
- Next workshop: process-level / design-level (scope + date)
- ADRs cần viết: …
- Tasks: …

### 2.4 Post-workshop report template (khuyến nghị)

**06-post-workshop-report.md**
```md
# Workshop Report — <Scenario> (<Date>)

## Summary (3–5 bullet)
- Workflow đã chốt: …
- Top 3 hotspots: …
- Decisions đã chốt ngay: …

## Decisions & ADRs
- ADR-XXX: <title> (owner, due)
- ADR-YYY: <title> (owner, due)

## Open Questions
1) …
2) …

## Risks & Mitigations
- Risk: …
  - Mitigation: …

## Next Steps
- Process-level workshop: <date>
- Design-level workshop: <date>
- Owner actions: …
```

---

## 3) Definition of Done (DoD) cho “Workshop đã xong”

- [ ] Có timeline tóm tắt + board snapshot link
- [ ] Hotspots top-5 có owner + due + next action
- [ ] Glossary seed tối thiểu 10 terms, không mâu thuẫn rõ rệt
- [ ] Có event catalog seed (ít nhất 8–12 events cho workflow đắt tiền)
- [ ] Có quyết định “đắt tiền” được đưa vào ADR backlog
