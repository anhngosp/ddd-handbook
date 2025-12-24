## Checklist: Event Storming (Chuẩn bị → Chạy → Tổng hợp → Follow-up)

Tài liệu này là checklist “dùng được ngay” để chạy Event Storming theo phong cách DDD Practical.  
Mục tiêu: workshop không trượt sang kỹ thuật, và kết thúc bằng artefacts đủ tốt để đi Strategic/Tactical/Implementation.

Ví dụ xuyên suốt: ADLP (kịch bản “Premium 48h”).

> **NOTE**  
> Nếu bạn đã có board và artefacts, bạn vẫn nên chạy checklist này như “pre-flight check” để tránh bỏ sót owner, decision, và follow-up.

---

## 1) Trước workshop (30–60 phút)

### 1.1 Scope & “kịch bản đắt tiền”
- [ ] Chọn **một** kịch bản đắt tiền nhất (VD: Premium 48h, Payout/Dispute, Export Audit)
- [ ] Viết “one-liner”: *Ai* muốn gì, trong *bao lâu*, chất lượng/điều kiện gì, nếu fail thì hậu quả gì
- [ ] Xác định “điểm kết thúc”: thế nào là “xong” trong ngôn ngữ business (không phải “status = done”)

### 1.2 People (đúng người, đúng quyền)
- [ ] Có **domain expert thật** (người có quyền quyết định trade-off)
- [ ] Có **facilitator** (giữ nhịp, giữ luật)
- [ ] Có đại diện implement (tech lead/senior) và người vận hành/QA (nếu workflow dài, compliance)
- [ ] Nếu có external dependencies (payment/model provider): có người hiểu ràng buộc hoặc có quyền “commit follow-up”

### 1.3 Luật chơi (để không trượt)
- [ ] Luật 1: ưu tiên “điều đã xảy ra” (events) trước “giải pháp”
- [ ] Luật 2: tranh luận kỹ thuật → đưa vào parking lot + timebox
- [ ] Luật 3: hotspot phải có **owner + due date** (không để câu hỏi trôi)
- [ ] Luật 4: tôn trọng ngôn ngữ business; thuật ngữ mơ hồ phải được đưa vào glossary

### 1.4 Chuẩn bị board (Miro/FigJam/whiteboard)
- [ ] Lane Timeline (events theo thời gian)
- [ ] Lane Actors/Commands (đặt ở gần events)
- [ ] Lane Policies/Rules (các quyết định “vì sao”)
- [ ] Lane Hotspots/Questions
- [ ] Lane Glossary seed
- [ ] Parking lot (tech debates)

### 1.5 Chuẩn bị “output pack” (để sau workshop không mất dữ liệu)
- [ ] Tạo sẵn file/khung để ghi: timeline tóm tắt, hotspots, glossary, event catalog seed
- [ ] Chốt nơi lưu artefacts (repo path) và người tổng hợp

---

## 2) Trong workshop (2–3 giờ) — Facilitator Cheat-Sheet

### 2.1 Khởi động (10 phút)
- [ ] Nhắc lại one-liner + “definition of done”
- [ ] Nhắc luật chơi + giới hạn: hôm nay không chốt giải pháp kỹ thuật chi tiết
- [ ] Xác nhận vai trò: ai là decision owner cho các policy (deadline, quality threshold, payout trigger, …)

### 2.2 Big Picture: dựng timeline events (30–60 phút)
- [ ] Viết events ở **thì quá khứ**, business-readable (`BatchAssigned`, `BatchSubmitted`, `BatchAccepted`…)
- [ ] Không dùng event mơ hồ (`Updated/Changed/Processed`) trừ khi định nghĩa rõ
- [ ] Khi tranh luận “có tồn tại bước này không?” → đó là dấu hiệu hotspot, ghi lại

> **EXAMPLE (ADLP)**  
> `PrelabelCompleted → BatchCreated → BatchAssigned → BatchSubmitted → QualityEvaluated → BatchAccepted → DatasetExported → EarningCredited`

### 2.3 Gắn actor/command/policy (30–45 phút)
- [ ] Ở mỗi event quan trọng: ai kích hoạt? command là gì? policy quyết định ra sao?
- [ ] Đặt câu hỏi: “Ai có quyền quyết định bước này?” → ghi owner
- [ ] Đặt câu hỏi: “Nếu fail ở đây, chuyện gì xảy ra tiếp theo?” → ghi failure modes

### 2.4 Đào hotspot (20–30 phút) — timebox
- [ ] Mỗi hotspot phải viết thành câu hỏi follow-up được
- [ ] Gán owner + due date + cách trả lời (mini workshop, ADR, spike, …)
- [ ] Nếu tranh luận kéo dài → timebox 2–3 phút, đưa vào follow-up

### 2.5 Chốt glossary seed (15–20 phút)
- [ ] Liệt kê 10–30 thuật ngữ “đắt tiền”
- [ ] Mỗi thuật ngữ có định nghĩa 1–2 câu (v0) + owner context (nếu đã rõ)
- [ ] Các từ đồng nghĩa/đa nghĩa phải được đánh dấu (để tách theo context)

---

## 3) Kết thúc workshop (15–30 phút)

- [ ] Kể lại workflow bằng 8–12 events như một câu chuyện (để kiểm “shared understanding”)
- [ ] Chốt top 5 hotspots quan trọng nhất (đừng để 30 items không ai làm)
- [ ] Chốt next step: process-level / design-level / strategic design / ADR workshop
- [ ] Chốt artefacts sẽ commit vào repo (ai tổng hợp, deadline)
- [ ] Lưu board link + snapshot (và đảm bảo quyền truy cập)

---

## 4) Sau workshop (Follow-up trong 24–72 giờ)

### 4.1 Tổng hợp artefacts (bắt buộc)
- [ ] Timeline tóm tắt (8–20 events) + scope note
- [ ] Hotspots list (question, why, owner, due, status)
- [ ] Glossary seed v1 (terms, definitions, owner context)
- [ ] Domain event catalog seed (owner context, consumers, payload tối thiểu, idempotency key gợi ý)

### 4.2 Quyết định “đắt tiền” (đưa vào ADR)
- [ ] Những lựa chọn sync/async quan trọng
- [ ] Event schema envelope + versioning rules
- [ ] SLO/NFR cho workflow đắt tiền (theo Chương 26)
- [ ] Retention/audit/compliance constraints (nếu liên quan)

### 4.3 “Gate” để biết workshop có đạt không
- [ ] Mọi hotspot top-5 đều có owner + due + next action
- [ ] Glossary có tối thiểu 10 thuật ngữ và không mâu thuẫn rõ rệt
- [ ] Có ít nhất một context map phác thảo cho workflow đắt tiền (nếu đã vào strategic)

---

## 5) Anti-patterns khi chạy Event Storming (tự kiểm)
- [ ] Workshop biến thành “họp giải pháp kỹ thuật” ngay từ đầu
- [ ] Không có domain expert thật → mọi thứ là suy diễn
- [ ] Không ghi hotspots → mơ hồ bị “nuốt” và sẽ nổ ở implementation
- [ ] Không tổng hợp artefacts → workshop thành “buổi brainstorm vui”

---

## 6) Tham chiếu liên quan trong handbook
- Toolkit: `design/docs/0.ref/DDDPractical/toolkit-event-storming.md`
- Checklist Discovery: `design/docs/0.ref/DDDPractical/checklist-discovery.md`
- Integration checklist: `design/docs/0.ref/DDDPractical/checklist-integration.md`

