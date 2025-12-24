# Chương 6 — Event Storming là gì và vì sao nó đứng đầu quy trình?

Bạn có thể đọc hàng chục trang về DDD nhưng vẫn không biết “bắt đầu từ đâu”. Đây là lý do Event Storming được đặt ở đầu quy trình: nó là cách nhanh nhất để biến **suy nghĩ mơ hồ** thành **bức tranh chung có thể nhìn thấy** — nơi business và tech nói cùng một ngôn ngữ, trên cùng một timeline.

Trong ADLP, Event Storming giúp bạn tránh những sai lầm đắt tiền như “submit = accepted”, “prelabel confidence = quality”, hay “lock TTL là chi tiết kỹ thuật”. Nó kéo mọi người về đúng câu hỏi: *Điều gì đã xảy ra?* và *Tại sao nó quan trọng?*

---

## Bạn sẽ nhận được gì sau chương này?

1) Hiểu Event Storming là gì (bản chất, không phải format giấy dán).  
2) Biết **tại sao Event Storming phải đứng đầu quy trình**.  
3) Có cách thiết kế workshop để ra được **event timeline, hotspots, glossary seed**.  
4) Biết phân biệt “event đúng nghĩa” với “event trá hình command”.  
5) Có exercise có hướng dẫn để tự chạy một mini Event Storming.

---

## 1) Event Storming là gì — nói ngắn gọn mà đúng

Event Storming là một workshop giúp team mô tả **workflow nghiệp vụ** bằng chuỗi **domain events** (điều đã xảy ra), theo timeline, và từ đó làm lộ:
- quy tắc ẩn (invariants),
- điểm mơ hồ (hotspots),
- ranh giới tự nhiên của domain (bounded contexts),
- ngôn ngữ chung (glossary).

Điểm cốt lõi: Event Storming không bắt đầu bằng “giải pháp”, mà bắt đầu bằng **sự kiện nghiệp vụ đã xảy ra**. Bạn chưa cần bàn REST hay Kafka, chưa cần bàn DB hay schema; bạn chỉ cần trả lời: *Điều gì đã xảy ra trong business?*

> **NOTE**  
> Nếu workshop của bạn đang tranh luận tool, framework, hoặc schema, thì đó không phải Event Storming nữa.

---

## 2) Vì sao Event Storming phải đứng đầu quy trình?

### 2.1 Vì nó làm lộ “điểm sai” trước khi bạn viết code

Event Storming ép domain expert và dev cùng nhìn vào một timeline sự kiện. Bạn sẽ thấy ngay:
- thuật ngữ bị hiểu khác nhau,
- rule nằm rải rác,
- chỗ nào “mơ hồ” hoặc “mâu thuẫn”.

Ví dụ ADLP: nếu timeline không tách `BatchSubmitted` và `BatchAccepted`, bạn sẽ sớm nhận ra “quality gate” chưa được hiểu đúng. Nếu timeline không có `ReviewRequired`, bạn sẽ thấy escalation policy đang bị bỏ quên.

### 2.2 Vì nó tạo “ngôn ngữ chung” nhanh nhất

Glossary seed hình thành tự nhiên khi mọi người phải gọi tên sự kiện.  
Nếu ai đó nói “task”, người khác nói “batch”, workshop sẽ buộc họ chốt lại nghĩa.

### 2.3 Vì nó giúp tìm ranh giới tự nhiên

Khi sự kiện và actor được gắn vào timeline, bạn sẽ thấy cụm events nào thuộc cùng một team/ownership. Đó chính là candidate for bounded context.

Ví dụ ADLP: các events liên quan quality (QualityEvaluated, ReviewRequired, BatchAccepted) tự nhiên cụm lại. Đây là một tín hiệu rõ rằng Quality Assurance là một context riêng.

---

## 3) Event Storming khác gì so với flowchart?

Flowchart thường mô tả **luồng xử lý** theo kỹ thuật (API call, DB update). Event Storming mô tả **sự thật nghiệp vụ** theo thời gian.

So sánh nhanh:
- Flowchart: “System receives request, updates DB, returns response.”  
- Event Storming: “BatchSubmitted → QualityEvaluated → BatchAccepted.”

Flowchart dễ rơi vào bias kỹ thuật. Event Storming kéo bạn về business.

### 3.1 So sánh nhanh các phương pháp

| Phương pháp | Focus | Output | Khi nào dùng |
|---|---|---|---|
| Event Storming | Business events | Timeline + Hotspots | Discovery, shared understanding |
| Use Case Diagram | Actor actions | Use cases | Requirements gathering |
| Flowchart | Process flow | Decision tree | Technical implementation |
| User Story Mapping | User journey | Story backlog | Agile planning |

---

## 4) Event đúng nghĩa là gì? (và cách nhận ra event sai)

### 4.1 Event đúng nghĩa
Event đúng nghĩa là một **điều đã xảy ra**, có ý nghĩa business, và có giá trị để audit hoặc trigger workflow khác.

Ví dụ ADLP:
- `PrelabelCompleted`
- `BatchAssigned`
- `BatchSubmitted`
- `ReviewRequired`
- `BatchAccepted`

### 4.2 Event trá hình command
Nếu event nghe như “hãy làm”, thì đó là command.  
Ví dụ:
- `RunPrelabeling` → command  
→ nên đổi thành `PrelabelCompleted` (event)

> **CHECKLIST**  
> Để kiểm tra event đúng nghĩa, hãy hỏi: “Nếu event này xuất hiện trong log audit, người business có hiểu nó không?”

### 4.3 Sticky notes color coding (gợi ý)

- Orange: Domain Events (điều đã xảy ra)  
- Blue: Commands (ý định)  
- Yellow: Actors (ai làm)  
- Pink: Hotspots/Questions (điểm mơ hồ)  
- Green: Policies (quy tắc)  
- Purple: External Systems (hệ thống ngoài)  

---

## 5) Ba output quan trọng nhất từ Event Storming

### 5.1 Event timeline
Đây là “xương sống” của workflow. Bạn sẽ dùng nó để:
- kiểm tra completeness,
- tìm missing steps,
- nối với bounded contexts.

### 5.2 Hotspots & Questions
Hotspot là chỗ mơ hồ hoặc tranh cãi.  
Ví dụ ADLP:
- “Accepted là gì? có cần review hay auto-accept?”  
- “Lock TTL bao lâu là hợp lý?”  
- “Payout trigger ở đâu?”

Hotspots là thứ bạn phải giải quyết trước khi code.

### 5.3 Glossary seed
Những thuật ngữ xuất hiện trong workshop tạo thành glossary seed (chương 4 và 5 đã nói). Đây là tài sản quan trọng để giảm tranh luận.

### 5.4 Flow workshop 90 phút (gợi ý)

- Minute 0–10: giới thiệu rules, mục tiêu, workflow scope  
- Minute 10–40: silent brainstorming events (viết nhanh, dán)  
- Minute 40–60: sắp timeline, gộp trùng, chốt tên event  
- Minute 60–80: gắn hotspots + owner  
- Minute 80–90: chốt glossary seed + next steps  

---

## 6) Event Storming trong ADLP: ví dụ rút gọn

Dưới đây là một timeline tối giản cho workflow “premium order 48h”:

1) `DataItemUploaded`  
2) `DataItemNormalized`  
3) `PrelabelCompleted`  
4) `BatchCreated`  
5) `BatchAssigned`  
6) `BatchSubmitted`  
7) `QualityEvaluated`  
8) `ReviewRequired` (nếu cần)  
9) `BatchAccepted`  
10) `DatasetExported`  

Chỉ cần timeline này, bạn đã có 3 thứ:
- Hình dung rõ pipeline end-to-end.
- Thấy ngay chỗ cần policy (review, escalation).
- Có input để vẽ context map sơ bộ.

### 6.1 Timeline visualization (ASCII)

```text
[DataItemUploaded] → [DataItemNormalized] → [PrelabelCompleted]
        → [BatchCreated] → [BatchAssigned] → [BatchSubmitted]
        → [QualityEvaluated] → [ReviewRequired?] → [BatchAccepted]
        → [DatasetExported]
```

---

## 7) Best practices (kèm giải thích)

### 7.1 “Event trước, giải pháp sau”
Nếu team nhảy vào bàn database hay queue, bạn sẽ mất shared understanding.  
Event Storming là về **điều đã xảy ra**, không phải “làm thế nào”.

### 7.2 Chỉ chọn một workflow đắt tiền
Chọn 1 workflow để tránh loãng. ADLP: premium order 48h.  
Khi làm tốt 1 workflow, bạn có thể mở rộng.

### 7.3 Luôn có domain expert thật sự
Nếu người tham gia không ra quyết định được, bạn sẽ thu được “giả định”, không phải domain truth.

### 7.4 Ghi lại hotspots ngay lập tức
Hotspots là nơi quyết định. Nếu không ghi, bạn sẽ quay lại tranh luận từ đầu.

---

## 8) Anti-patterns (triệu chứng → hậu quả → cách tránh)

### 8.1 Workshop thành họp kỹ thuật
**Triệu chứng:** 10 phút đầu event, 50 phút sau tranh luận Kafka vs REST.  
**Hậu quả:** không có shared understanding.  
**Cách tránh:** facilitator nhắc “điều đã xảy ra” và đưa về timeline.

### 8.2 Event list chỉ là “CRUD log”
**Triệu chứng:** `RecordUpdated`, `DataChanged`, `RowInserted`.  
**Hậu quả:** consumer không hiểu ý nghĩa business, khó audit.  
**Cách tránh:** đặt event bằng ngôn ngữ business (BatchAccepted).

### 8.3 Không có domain expert
**Triệu chứng:** dev tự quyết nghĩa của event.  
**Hậu quả:** sai domain, lộ muộn.  
**Cách tránh:** domain expert bắt buộc tham gia.

---

## 9) Exercise có hướng dẫn: mini Event Storming (30 phút)

### Bước 1: Chọn workflow
Chọn “premium order 48h” của ADLP.

### Bước 2: Viết 10 câu “điều đã xảy ra”
Ví dụ: “Audio được upload”, “Audio được chuẩn hóa”, “Batch được assign”, …

### Bước 3: Chuyển thành event names
Đổi sang thì quá khứ: `DataItemUploaded`, `BatchAssigned`.

### Bước 4: Gắn 3 hotspots
Ví dụ: “Submit vs Accept”, “Lock TTL”, “Review policy”.

### Đáp án tham khảo
Timeline rút gọn:  
`DataItemUploaded → DataItemNormalized → PrelabelCompleted → BatchCreated → BatchAssigned → BatchSubmitted → QualityEvaluated → BatchAccepted → DatasetExported`

**Câu hỏi tự kiểm**
1) Event nào nghe như command?  
2) Event nào không có ý nghĩa business?  
3) Hotspot nào nếu bỏ qua sẽ gây tranh chấp lớn nhất?

---

## 10) Artefacts/Deliverables sau chương này

Sau một Event Storming cơ bản, bạn nên có:
- Event timeline (10–20 events).  
- Glossary seed (10–30 terms).  
- Hotspots/questions list.  
- Đề xuất bounded context candidates.  

Chỉ cần có những artefacts này, chương tiếp theo (các cấp độ Event Storming) sẽ đi vào chiều sâu mà không bị mơ hồ.
