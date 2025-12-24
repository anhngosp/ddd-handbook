# DDD Practical Handbook — V1 Notes (Scope freeze + Next iteration ideas)

Tài liệu này chốt phạm vi v1 của handbook và ghi các ý tưởng cải tiến cho vòng tiếp theo.

---

## 1) Scope v1 (đã có trong repo)

**Handbook (Chương 01–34)**  
- Mỗi chương nằm trong một file riêng trong `design/docs/0.ref/DDDPractical/`.
- Outline đã link đủ tới từng chương trong `design/docs/0.ref/DDDPractical/outline-handbook.md`.

**Running Example**
- ADLP Premium 48h end-to-end: `design/docs/0.ref/DDDPractical/adlp-running-example.md`

**Checklists/Toolkits/Templates (dùng được ngay)**
- Event Storming checklist: `design/docs/0.ref/DDDPractical/checklist-event-storming.md`
- Discovery checklist: `design/docs/0.ref/DDDPractical/checklist-discovery.md`
- Strategic checklist: `design/docs/0.ref/DDDPractical/checklist-strategic-design.md`
- Tactical checklist: `design/docs/0.ref/DDDPractical/checklist-tactical-design.md`
- Integration checklist: `design/docs/0.ref/DDDPractical/checklist-integration.md`
- Code review checklist: `design/docs/0.ref/DDDPractical/checklist-code-review-ddd.md`
- Toolkit Event Storming: `design/docs/0.ref/DDDPractical/toolkit-event-storming.md`
- Template ADR: `design/docs/0.ref/DDDPractical/template-adr.md`
- Template Event schema + versioning: `design/docs/0.ref/DDDPractical/template-event-schema-versioning.md`
- Template Workshop agenda + output pack: `design/docs/0.ref/DDDPractical/template-workshop-artefact-pack.md`
- Templates tổng hợp: `design/docs/0.ref/DDDPractical/templates.md`

**Glossary**
- DDD & ADLP glossary: `design/docs/0.ref/DDDPractical/glossary.md`

**Quality gates v1**
- Tất cả chapter 01–34 có: mục tiêu/chuyện mở đầu (phần lớn), best practices + anti-patterns + exercise + checklist.
- Internal links trong outline đều trỏ tới file tồn tại.

---

## 2) Những thứ cố ý chưa làm trong v1 (để đóng scope)

- Viết “case study” có artefacts thật (ảnh board, context map mermaid, ADR thật) cho 1 workshop cụ thể.
- Thêm nhiều mermaid diagrams cho mọi chương (v1 ưu tiên narrative + templates).
- Chuẩn hóa style lint/markdown formatter (v1 ưu tiên nội dung).
- Tự động tạo “index” hoặc site docs.

---

## 3) Next iteration ideas (v2+)

### 3.1 Case study thực chiến (giá trị cao nhất)
- Tạo 1 thư mục workshop thật theo template: `design/docs/0.ref/DDDPractical/workshops/<date>-premium-48h/` và điền artefacts thật (timeline/hotspots/glossary/event catalog).
- Viết 3–5 ADR thật cho ADLP (integration, outbox, SLO, retention, export access control).

### 3.2 Governance & consistency
- Bổ sung “event catalog” chuẩn hóa cho ADLP (1 file tổng hợp) và link từ glossary.
- Bổ sung “contract testing playbook” (tool-agnostic) cho REST/events.

### 3.3 Implementation accelerators
- Thêm skeleton code minh họa (pseudo hoặc repo-specific) cho 2 use cases: `ClaimBatch`, `AcceptBatch`.
- Thêm mẫu “runbook” cho DLQ spike/backlog spike/payout mismatch.

### 3.4 UX cho người đọc
- Thêm “reading path”: nếu là Architect/Tech Lead/Dev thì nên đọc chương nào trước.
- Thêm “quickstart 2 giờ” (chỉ đủ để chạy 1 workshop + chốt slice).

