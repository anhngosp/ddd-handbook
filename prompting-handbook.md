# Prompting Guide — Viết “DDD Practical Handbook” (tái sử dụng cho nhiều session)

Tài liệu này là “prompt chuẩn” để dùng lại khi viết tiếp handbook ở các session khác mà không mất ngữ cảnh, không mất quy tắc soạn thảo.

Mục tiêu: mỗi chương đọc xong phải **wow** và “ngấm” — người đọc hiểu ngay **khái niệm có ý nghĩa gì**, **khi nào dùng**, **trade-off ra sao**, **best practices thế nào**, và **gắn được với hệ thống ADLP đang thiết kế** (tài liệu trong `design/docs/archive`).

---

## 1) Phạm vi handbook và độc giả mục tiêu

**Handbook:** Domain-Driven Design – Thực hành từ Zero đến Production  
**Độc giả:** Solution/Software Architect, Tech Lead, Senior/Lead Developer (chuẩn bị khởi động hoặc tái cấu trúc hệ thống)  
**Mức độ:** “book-like” nhưng vẫn là handbook: dẫn dắt, có ví dụ thật, có template/checklist dùng được ngay; tránh giáo điều học thuật.

**Tuyên bố giá trị:**  
Handbook giúp người đọc **bắt đầu dự án đúng cách**, giảm rủi ro hiểu sai domain, và thiết kế bền vững khi hệ thống tiến hóa.

---

## 2) Running Example xuyên suốt: ADLP (AI Data Labeling Platform)

Mọi khái niệm quan trọng nên được minh họa bằng **một câu chuyện xuyên suốt**: ADLP/labeling platform (audio STT MVP).  
Trong handbook này, **nguồn thiết kế mới nhất (authoritative)** của ADLP là:
- `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`

Thư mục `design/docs/archive` chỉ dùng để **lấy và tóm tắt bối cảnh dự án** (context/background) và các tài liệu bổ trợ (infra/runbook/đào sâu 1 phần), không dùng làm “nguồn chốt” nếu có khác biệt với bản Strategic Design v0.2.

### 2.1 Tóm tắt ADLP (để nhắc lại đầu mỗi chương, ngắn)

### 2.1.1 Bối cảnh dự án (tóm tắt sẵn để dùng xuyên session)
ADLP (AI Data Labeling Platform) là nền tảng enterprise để thu thập, prelabeling, gán nhãn và phân phối dữ liệu huấn luyện AI. Hệ thống vận hành theo mô hình marketplace: labeler làm việc theo batch và được trả công dựa trên chất lượng; data consumer chọn tier chất lượng dataset và trả phí tương ứng.

MVP tập trung vào audio speech-to-text (Vi/En), nhưng thiết kế cho phép mở rộng sang ảnh/text/3D. Các “điểm đau” domain chủ đạo (đáng đầu tư DDD) nằm ở:
- **Task assignment & routing**: ưu tiên, deadline, skill-matching, lock TTL, reassign.
- **Quality assurance**: đo chất lượng (WER/agreement), overlap sampling, automated review, escalation workflow, audit trail.
- **Prelabeling orchestration**: segmentation, confidence scoring, model routing/versioning.

Theo Strategic Design v0.2, ADLP có **9 Bounded Contexts** (mỗi context là một ranh giới ngôn ngữ + ownership):
1) Identity & Access  
2) User Profile  
3) Wallet & Payment  
4) Data Ingestion  
5) Prelabeling  
6) Task Assignment  
7) Labeling  
8) Quality Assurance  
9) Dataset Export

Workflow “xương sống” (áp dụng cho mọi ví dụ trong handbook):
- Ingest & Normalize: upload/crawl, validate/normalize, dedupe, lưu S3, metadata Postgres.
- Prelabel: tạo segments + confidence, lưu kết quả, phát event hoàn thành.
- Task Assignment/Queue: tạo batch, routing theo skill/rating/confidence, lock TTL, reassign.
- Labeling: UI edit transcript, autosave, submit.
- Quality/Review/Escalation: WER/agreement, overlap sampling, automated review, bias checks, escalation, quyết định accept/reject/rework.
- Export: đóng gói dataset (versioned), signed URL, audit download.

Mục tiêu của handbook: biến các khái niệm DDD (Ubiquitous Language, Bounded Context, Context Map, Aggregate, Domain Event, Consistency, ACL, ADR/SLO…) thành “đồ nghề” áp vào ADLP — để người đọc thấy ngay ý nghĩa, trade-off, và best practices.

### 2.2 Nguồn tham chiếu trong repo (không copy nguyên văn; tóm tắt và trích ý)
Khi cần chứng minh “thực tế”, **ưu tiên bám thiết kế mới nhất**:
- Strategic Design (authoritative): `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`

Thư mục archive dùng để lấy bối cảnh, ví dụ bổ trợ, và các chi tiết triển khai/ops (khi không mâu thuẫn với Strategic Design):
- Tổng quan: `design/docs/archive/system_overview.md`, `design/docs/archive/project_brief.md`
- Service inventory & CI: `design/docs/archive/service_design_and_ci.md`, `design/docs/archive/implementation/service_design.md`
- Ingest/Prelabel chi tiết: `design/docs/archive/production/ingest_prelabel_design.md`
- Chuẩn production: `design/docs/archive/production/microservice_production_standards.md`
- Task queue: `design/docs/archive/task_queue_management.md`
- Automated review/bias/escalation: `design/docs/archive/automated_review.md`, `design/docs/archive/escalation_workflow.md`
- AI integration: `design/docs/archive/AI_integration.md`

**Quy ước ví dụ xuyên suốt:**  
Giữ nhất quán các thuật ngữ và artefacts: `Batch`, `Segment`, `Prelabel`, `Confidence`, `Submission`, `Review`, `QualityScore`, `Escalation`, `DatasetExport`.

---

## 3) Văn phong và tiêu chuẩn “wow”

### 3.1 Văn phong “book-like”
- Viết theo mạch kể: mở bằng tình huống thật → nêu vấn đề → giải thích khái niệm → áp dụng vào ADLP → trade-off → best practices → checklist.
- Hạn chế bullet-only. Bullet dùng cho checklist/tóm tắt, không dùng thay cho diễn giải.
- Dùng ví dụ thực tế thường xuyên, ưu tiên tình huống có hậu quả (tiền, SLA, compliance, dữ liệu sai).

### 3.2 Tiêu chuẩn “wow” (Definition of Done cho mỗi chương)
Một chương đạt yêu cầu khi có đủ:
1) **Vì sao khái niệm quan trọng** (tác hại nếu làm sai).
2) **Khái niệm là gì** (giải thích rõ, tránh định nghĩa mơ hồ).
3) **Dấu hiệu khi nào dùng / không dùng** (heuristics thực tế).
4) **Trade-offs** (được/mất, rủi ro, độ phức tạp).
5) **Best practices** (và giải thích vì sao).
6) **Anti-patterns** (triệu chứng + hậu quả + cách tránh).
7) **Áp dụng vào ADLP** (mapping cụ thể).
8) **Output/Artefacts** tạo ra sau chương (glossary, context map, ADR, event schema, v.v.).
9) **Checklist** dùng ngay.

---

## 4) Quy ước định dạng (Markdown)

### 4.1 Cấu trúc chương bắt buộc (template)
Mỗi file chương nên theo khung:
1) Tiêu đề chương
2) “Bạn sẽ nhận được gì sau chương này” (3–5 dòng)
3) Tình huống mở đầu (story) — gắn ADLP
4) Giải thích khái niệm (dài, rõ)
5) Khi nào dùng / khi nào tránh
6) Trade-offs
7) Best practices (kèm giải thích)
8) Anti-patterns
9) “Áp dụng vào ADLP” (mapping rõ ràng)
10) Checklist
11) Artefacts/Deliverables
12) Bài tập nhỏ (tuỳ chương, 5–15 phút)

> **NOTE**  
> Template box dùng lại (DoD, callouts, glossary seed, event catalog, ADR, event schema, workshop agenda) nằm tại:  
> `design/docs/0.ref/DDDPractical/templates.md`

### 4.2 Callout blocks dùng thống nhất
Dùng đúng cú pháp Markdown sau (phù hợp GitHub):
> **NOTE** — ghi chú quan trọng  
> **WARNING** — cảnh báo rủi ro/anti-pattern  
> **EXAMPLE** — ví dụ cụ thể  
> **CHECKLIST** — danh sách làm ngay  

### 4.3 Sơ đồ
- Ưu tiên Mermaid cho các sơ đồ đơn giản (context map, flow).
- Không lạm dụng; chỉ dùng khi giúp hiểu nhanh hơn 3 đoạn văn.

---

## 5) Quy ước thuật ngữ (Việt/Anh)

### 5.1 Nguyên tắc
- Giữ thuật ngữ DDD chuẩn bằng tiếng Anh khi cần độ chính xác: `Bounded Context`, `Aggregate`, `Domain Event`, `Ubiquitous Language`, `Context Map`.
- Khi dùng tiếng Việt, viết kèm tiếng Anh lần đầu trong chương: ví dụ “Ngữ cảnh giới hạn (Bounded Context)”.
- Không dịch tùy tiện làm méo nghĩa (ví dụ “Aggregate” không chỉ là “tổng hợp”; nó là ranh giới nhất quán).

### 5.2 Mini-glossary dùng xuyên suốt handbook (có thể mở rộng)
- **Domain:** miền nghiệp vụ
- **Core domain:** phần tạo lợi thế cạnh tranh
- **Invariant:** quy tắc phải luôn đúng
- **Boundary:** ranh giới trách nhiệm/nhất quán
- **Eventual consistency:** nhất quán cuối
- **Choreography vs Orchestration:** điều phối phân tán vs điều phối tập trung

---

## 6) “Prompt chuẩn” để viết một chương trong session mới

Copy khối dưới đây, thay `CHAPTER_TITLE`, `CHAPTER_OUTLINE_SECTION`, và danh sách tài liệu tham chiếu cần đọc.

### 6.1 Prompt
```
Bạn là tác giả handbook DDD Practical (văn phong sách, thực dụng, có ví dụ).

Mục tiêu chương: (1) giải thích rõ khái niệm, (2) khi nào dùng/không dùng, (3) trade-offs, (4) best practices + anti-patterns,
(5) áp dụng vào ADLP (AI Data Labeling Platform) theo tài liệu trong design/docs/archive.

Yêu cầu định dạng:
- Viết theo mạch kể, hạn chế bullet-only.
- Có các block NOTE/WARNING/EXAMPLE/CHECKLIST.
- Có mục “Áp dụng vào ADLP” với mapping cụ thể tới: ingest, prelabel, assignment, labeling, quality/review, export.
- Kết thúc bằng checklist và artefacts/deliverables.
- Dùng các template trong design/docs/0.ref/DDDPractical/templates.md (glossary seed, domain event catalog, ADR, event schema, workshop agenda) khi phù hợp.

Chương cần viết:
- Tiêu đề: CHAPTER_TITLE
- Thuộc phần: CHAPTER_OUTLINE_SECTION

Tài liệu tham chiếu (tóm tắt ý chính, không copy nguyên văn):
- design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md
- design/docs/archive/system_overview.md
- design/docs/archive/...

Hãy viết chương này thật chi tiết, dễ hiểu, có ví dụ thực tế và giải thích “vì sao” cho các best practices.
```

---

## 6.2 Hướng dẫn viết “exercise” để người đọc không bị mơ hồ

Nguyên tắc: không giao bài tập kiểu “tự bơi”. Mỗi exercise phải có:
1) **Chọn 1 kịch bản duy nhất** (không chọn nhiều).
2) **Bước nghĩ** (step-by-step) trước khi ra đáp án.
3) **Đáp án tham khảo** (không bắt người đọc giống 100%).
4) **Câu hỏi tự kiểm** (để biết mình làm đúng hướng).

Ví dụ khuôn mẫu:
- Bước 1: chọn kịch bản…
- Bước 2: viết lại thành câu chuyện…
- Bước 3: chuyển thành events/terms…
- Bước 4: gắn owner context…
- Đáp án tham khảo: …

--- 

## 7) Quy trình làm việc (để không mất context)

Mỗi session viết tiếp handbook:
1) Mở `design/docs/0.ref/DDDPractical/outline-handbook.md` để biết outline hiện tại.
2) Mở `design/docs/0.ref/DDDPractical/tasks.md` để chọn chương tiếp theo.
3) Đọc đúng tài liệu trong `design/docs/archive` liên quan tới chương (chỉ đọc phần cần).
4) Viết chương vào file riêng trong `design/docs/0.ref/DDDPractical/` theo quy ước tên file.
5) Cập nhật `design/docs/0.ref/DDDPractical/outline-handbook.md` để chèn link tới file chương.
6) Tick task tương ứng trong `design/docs/0.ref/DDDPractical/tasks.md`.

---

## 7.1 Quy tắc “nguồn sự thật” (source of truth) cho ADLP

- **Authoritative:** `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md` (bounded contexts, context map, ADRs/SLOs, thuật ngữ chính).
- **Archive:** chỉ để lấy bối cảnh và chi tiết bổ trợ; nếu mâu thuẫn thì ưu tiên bản Strategic Design v0.2.

---

## 8) Naming conventions cho file chương

Đề xuất: `NN-ten-chuong-ngan-gon.md`, ví dụ:
- `01-vi-sao-ddd-quan-trong.md`
- `02-ddd-khong-phai-la-gi.md`
- `03-ban-do-quy-trinh-ddd.md`

---

## 9) Quality bar: best practices phải có “lý do”

Khi đưa best practice, luôn kèm:
- Vì sao đúng trong thực tế (hậu quả nếu làm sai).
- Chi phí/đánh đổi.
- Điều kiện áp dụng (không có one-size-fits-all).

Ví dụ khuôn mẫu:
- **Best practice:** X  
  **Vì sao:** …  
  **Trade-off:** …  
  **Khi dùng:** …  
  **Khi tránh:** …  
