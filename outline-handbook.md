Tài liệu này là **handbook outline**: chắc, có logic, dùng được để **khởi động bất kỳ dự án mới nào** (startup đến enterprise).

Ở bước này **chỉ xây outline**, chưa viết nội dung chi tiết.

---

# HANDBOOK OUTLINE

## Domain-Driven Design – Thực hành từ Zero đến Production

**Prompting (dùng lại cho các session viết tiếp)**  
- `design/docs/0.ref/DDDPractical/prompting-handbook.md`
- Templates: `design/docs/0.ref/DDDPractical/templates.md`

**Running Example (đọc xuyên handbook)**  
- ADLP Premium 48h: `design/docs/0.ref/DDDPractical/adlp-running-example.md`

**V1 Notes**
- Scope freeze + next ideas: `design/docs/0.ref/DDDPractical/v1-notes.md`

**Đối tượng**
- Solution/Software Architect
- Senior/Lead Developer
- Tech Lead chuẩn bị khởi động dự án

**Mục tiêu**
- Không dạy DDD kiểu học thuật; tập trung “làm gì trước – làm như thế nào – vì sao”.
- Kết nối thực hành: **Domain Discovery → Event Storming → Strategic Design → Tactical Design → System Design → Implementation → Evolution**.

---

# PHẦN I — TƯ DUY NỀN TẢNG

## 1. Vì sao DDD vẫn quan trọng trong hệ thống hiện đại?

- Nội dung: `design/docs/0.ref/DDDPractical/01-vi-sao-ddd-quan-trong.md`
- Vấn đề thực tế khi **không dùng DDD**: hệ thống “đúng kỹ thuật” nhưng vẫn thất bại vì hiểu sai business.
- Khoảng cách giữa business và code: khi ngôn ngữ/khái niệm không khớp, chi phí thay đổi tăng theo thời gian.
- DDD không phải “pattern” hay “framework”, mà là **cách tư duy thiết kế**.
- Phân biệt:
  - Code-driven design
  - Data-driven design
  - Domain-driven design
- DDD trong bối cảnh hiện đại:
  - Microservices / Modular monolith
  - Event-driven architecture
  - AI / Data platforms

---

## 2. DDD không phải là gì (Myths & Anti-patterns)

- Nội dung: `design/docs/0.ref/DDDPractical/02-ddd-khong-phai-la-gi.md`
- “DDD = phức tạp / nhiều class”
- “Chỉ dùng cho hệ thống lớn”
- “DDD = microservices”
- Khi nào **không nên dùng DDD** (độ phức tạp thấp, domain ổn định, CRUD thuần, time-to-market cực gấp)

---

## 3. Bản đồ toàn cảnh quy trình DDD thực hành

 - Nội dung: `design/docs/0.ref/DDDPractical/03-ban-do-quy-trinh-ddd.md`

> Chương này là **kim chỉ nam xuyên suốt handbook**.

Pipeline tổng thể:

```
Business Problem
   ↓
Domain Discovery
   ↓
Event Storming
   ↓
Strategic Design (Bounded Context, Context Map)
   ↓
Tactical Design (Aggregates, Domain Model)
   ↓
System Design (Architecture, Integration)
   ↓
Detailed Design (API, DB, Contracts)
   ↓
Implementation & Evolution
```

- Vai trò của từng artefact
- Ai tham gia ở mỗi bước
- Output của bước trước = input của bước sau

---

# PHẦN II — KHÁM PHÁ MIỀN (DOMAIN DISCOVERY)

## 4. Hiểu miền nghiệp vụ trước khi vẽ kiến trúc

- Nội dung: `design/docs/0.ref/DDDPractical/04-hieu-domain-truoc-kien-truc.md`
- Checklist Discovery: `design/docs/0.ref/DDDPractical/checklist-discovery.md`
- “Domain” là gì? Phân biệt **miền nghiệp vụ** và **hệ thống phần mềm**.
- Xác định **Core / Supporting / Generic**:
  - Core: nơi tạo lợi thế cạnh tranh (đáng đầu tư sâu)
  - Supporting: đặc thù business nhưng không tạo khác biệt lớn
  - Generic: commodity, ưu tiên dùng giải pháp chuẩn
- Sai lầm phổ biến:
  - Gọi nhầm “core” theo cảm tính/kỹ thuật
  - Đẩy phần core sang vendor/framework và giữ lại phần generic để tự viết

---

## 5. Domain Expert: vai trò và anti-pattern

- Nội dung: `design/docs/0.ref/DDDPractical/05-domain-expert.md`
- Ai là **domain expert thật sự**? (người ra quyết định/đánh đổi trong vận hành)
- Vì sao BA/PO đôi khi **không đủ** để thay thế domain expert
- Anti-pattern: developer tự “suy diễn nghiệp vụ” → sai mô hình → sai hệ thống
- Cách làm việc hiệu quả:
  - Cơ chế review từ domain expert theo vòng lặp ngắn
  - Chốt thuật ngữ/định nghĩa: “nói cùng một thứ bằng cùng một từ”

---

# PHẦN III — EVENT STORMING (KHỞI ĐỘNG DỰ ÁN ĐÚNG CÁCH)

## 6. Event Storming là gì và vì sao đứng đầu quy trình?

- Nội dung: `design/docs/0.ref/DDDPractical/06-event-storming-la-gi.md`
- Event Storming vs “requirements doc”/flowchart
- Tư duy **event-centric**: khám phá sự thật nghiệp vụ (business reality)
- Vì sao Event Storming thường hiệu quả hơn flowchart: kéo đúng người, đúng ngôn ngữ, vào cùng một bức tranh
- Kết quả mong đợi:
  - Shared understanding
  - Business language
  - Ranh giới tự nhiên của miền

---

## 7. Các cấp độ Event Storming

- Nội dung: `design/docs/0.ref/DDDPractical/07-cac-cap-do-event-storming.md`
- Big Picture: toàn cảnh end-to-end
- Process-level: đi sâu theo luồng/actor/policy
- Design-level: đi sâu mô hình (aggregate/command/event) để chuẩn bị tactical design

---

## 8. Chuẩn bị Event Storming Workshop

- Nội dung: `design/docs/0.ref/DDDPractical/08-chuan-bi-workshop.md`
- Stakeholders cần có
- Chuẩn bị mindset: ưu tiên business talk, hạn chế technical talk
- Không gian, công cụ, luật chơi
- Vai trò:
  - Facilitator
  - Domain Expert
  - Architect
  - Observer

---

## 9. Big Picture Event Storming

- Nội dung: `design/docs/0.ref/DDDPractical/09-big-picture-thuc-hanh.md`
- Business Event là gì?
- Cách đặt câu hỏi đúng
- Tránh technical talk
- Kết quả mong đợi:
  - Business flow end-to-end
  - Điểm nghẽn, mâu thuẫn, ambiguity
  - Hotspots & Questions (điểm nóng cần làm rõ)

---

## 10. Process-level Event Storming

- Nội dung: `design/docs/0.ref/DDDPractical/10-process-level-thuc-hanh.md`
- Commands
- Policies
- Actors
- External Systems
- Time & consistency boundaries

---

## 11. Design-level Event Storming (cầu nối sang tactical)

- Nội dung: `design/docs/0.ref/DDDPractical/11-design-level-event-storming.md`
- Từ command/event sang phác thảo Aggregate candidate
- Xác định invariants và transaction boundary ban đầu
- Mapping các tình huống edge-case (retries, idempotency, race conditions)

---

## 12. Từ Event Storming đến ngôn ngữ miền (Ubiquitous Language)

- Nội dung: `design/docs/0.ref/DDDPractical/12-chot-ubiquitous-language.md`
- Cách “chốt” thuật ngữ
- Phát hiện từ đồng nghĩa/đa nghĩa
- Tài liệu hóa ngôn ngữ miền (glossary)
- Khi nào cần tách ngôn ngữ theo ngữ cảnh

**Toolkit (dùng được ngay)**
- `design/docs/0.ref/DDDPractical/toolkit-event-storming.md`

---

# PHẦN IV — STRATEGIC DESIGN (THIẾT KẾ CHIẾN LƯỢC)

## 13. Strategic Design giải quyết vấn đề gì?

- Nội dung: `design/docs/0.ref/DDDPractical/13-strategic-vs-tactical.md`
- Strategic vs Tactical
- Khi nào nên dừng Event Storming để chốt chiến lược
- Strategic design là **quyết định đắt tiền** (ảnh hưởng dài hạn đến team, deploy, data ownership)

---

## 14. Bounded Context – chia hệ thống đúng cách

- Nội dung: `design/docs/0.ref/DDDPractical/14-bounded-context.md`
- Vì sao không thể có “một model cho tất cả” (mâu thuẫn ngữ nghĩa, overloaded model)
- Tiêu chí xác định BC:
  - Năng lực nghiệp vụ
  - Ownership & team
  - Vòng đời dữ liệu
  - Tốc độ thay đổi
- Dấu hiệu BC tốt / BC xấu
- Logical BC vs Physical Service (BC không luôn = microservice)
- Anti-pattern khi chia BC:
  - Chia theo database
  - Chia theo UI
  - Chia theo CRUD

---

## 15. Context Map – kiến trúc ở mức hệ thống

- Nội dung: `design/docs/0.ref/DDDPractical/15-context-map.md`
- Vì sao quan hệ giữa BC quan trọng không kém bản thân BC (coupling thật sự nằm ở integration)
- Các strategic patterns:
  - Partnership
  - Customer/Supplier
  - Conformist
  - Shared Kernel
  - Open Host Service
  - Anti-Corruption Layer
- Quyết định & trade-off:
  - Khi nào dùng events, khi nào dùng REST
  - Khi nào chấp nhận coupling, khi nào cần ACL

---

## 16. Strategic Design như nền móng kiến trúc

- Nội dung: `design/docs/0.ref/DDDPractical/16-strategic-design-to-architecture.md`
- Strategic Design → ảnh hưởng chọn microservices vs modular monolith
- Strategic Design → team topology (ai sở hữu cái gì)
- Strategic Design → roadmap (đầu tư theo core domain)

---

## 17. Strategic Decisions & ADRs

- Nội dung: `design/docs/0.ref/DDDPractical/17-adrs.md`
- Vì sao cần ADR
- Strategic decision nào cần ghi lại
- Ví dụ ADR thực tế
- ADR như “bộ nhớ tổ chức” giúp tránh lặp sai lầm

- Checklist Strategic Design: `design/docs/0.ref/DDDPractical/checklist-strategic-design.md`

---

# PHẦN V — TACTICAL DESIGN (THIẾT KẾ CHIẾN THUẬT)

## 18. Khi nào bắt đầu Tactical Design?

- Nội dung: `design/docs/0.ref/DDDPractical/18-khi-nao-bat-dau-tactical.md`
- Dấu hiệu sẵn sàng
- Không làm tactical khi strategic chưa ổn
- Làm theo từng Bounded Context (không cố thiết kế toàn hệ thống một lần)

---

## 19. Aggregate – ranh giới nhất quán

- Nội dung: `design/docs/0.ref/DDDPractical/19-aggregate-ranh-gioi-nhat-quan.md`
- Aggregate là gì (và không phải là gì)
- Aggregate Root & invariants
- Transaction boundary
- Sai lầm phổ biến:
  - God Aggregate
  - Đưa mọi thứ vào 1 transaction
  - Thiết kế aggregate theo schema DB

---

## 20. Entity, Value Object, Domain Service

- Nội dung: `design/docs/0.ref/DDDPractical/20-entity-vo-domain-service.md`
- Entity vs Value Object
- Domain Service vs Application Service
- Khi nào cần Factory & Repository

---

## 21. Domain Events & Consistency

- Nội dung: `design/docs/0.ref/DDDPractical/21-domain-events-consistency.md`
- Domain Event vs Integration Event
- Event là kết quả của domain change
- Eventual consistency
- Saga / Process Manager (khi workflow chạy qua nhiều aggregate/context)
- Versioning event

---

## 22. Repositories & Factories

- Nội dung: `design/docs/0.ref/DDDPractical/22-repository-factory.md`
- Repository không phải DAO
- Persistence ignorance
- Factory để đảm bảo business invariants

- Checklist Tactical Design: `design/docs/0.ref/DDDPractical/checklist-tactical-design.md`

---

# PHẦN VI — SYSTEM & SOLUTION DESIGN

## 23. Từ Domain Model sang kiến trúc hệ thống

- Nội dung: `design/docs/0.ref/DDDPractical/23-domain-to-architecture.md`
- Bounded Context ↔ Deployment Unit (khi nào nên/không nên)
- Microservices vs Modular Monolith
- Khi nào gộp/tách service (trade-off vận hành vs tốc độ phát triển)
- Conway’s Law trong thực tế

---

## 24. Integration Design trong hệ thống phân tán

- Nội dung: `design/docs/0.ref/DDDPractical/24-integration-design.md`
- Sync vs Async
- REST vs Events
- API contracts
- Event schema & versioning
- Idempotency & retries
- Saga: choreography vs orchestration

---

## 25. Data & Infrastructure Design

- Nội dung: `design/docs/0.ref/DDDPractical/25-data-infrastructure-design.md`
- Database per service
- Read models (CQRS mức vừa đủ)
- Event store (khi nào cần, khi nào không)
- Caching & performance

---

## 26. Non-Functional Requirements trong DDD

- Nội dung: `design/docs/0.ref/DDDPractical/26-nfr-by-design.md`
- Scalability
- Observability
- Security
- Compliance
- Performance
- Resilience

**Checklist**
- Integration checklist: `design/docs/0.ref/DDDPractical/checklist-integration.md`

---

# PHẦN VII — DETAILED DESIGN & IMPLEMENTATION

## 27. API Design theo Domain

- Nội dung: `design/docs/0.ref/DDDPractical/27-api-design-theo-domain.md`
- API là hợp đồng domain
- Tránh “leak” domain nội bộ
- Versioning strategy

---

## 28. Từ Use Case đến Code

- Nội dung: `design/docs/0.ref/DDDPractical/28-tu-use-case-den-code.md`
- Command handling
- Validation
- Transaction management
- Publishing events

---

## 29. Tổ chức code theo DDD

- Nội dung: `design/docs/0.ref/DDDPractical/29-to-chuc-code-theo-ddd.md`
- Layered architecture (Domain / Application / Infrastructure / Interfaces)
- Hexagonal / Clean Architecture
- Dependency rules
- Anti-pattern khi code DDD (anemic model, domain phụ thuộc framework, transaction rải rác)

---

## 30. Testing trong DDD

- Nội dung: `design/docs/0.ref/DDDPractical/30-testing-trong-ddd.md`
- Unit test domain
- Integration test aggregate/context
- Contract test giữa các context/service
- Event-driven testing

**Checklist**
- Code review checklist (DDD): `design/docs/0.ref/DDDPractical/checklist-code-review-ddd.md`

---

# PHẦN VIII — VẬN HÀNH & TIẾN HÓA

## 31. DDD trong môi trường thực tế

- Nội dung: `design/docs/0.ref/DDDPractical/31-ddd-trong-thuc-te.md`
- Business thay đổi: domain model phải tiến hóa
- Refactor bounded context (khi nào và làm thế nào)
- Dealing with legacy

---

## 32. Anti-patterns & bài học xương máu

- Nội dung: `design/docs/0.ref/DDDPractical/32-anti-patterns-bai-hoc-xuong-mau.md`
- Over-engineering
- DDD “nửa mùa” (vẽ đẹp nhưng code không phản ánh domain)
- Event-driven sai cách (event như log, thiếu schema/versioning)
- “Chia microservices trước, rồi mới tìm domain”

---

## 33. Checklist bắt đầu dự án mới với DDD

- Nội dung: `design/docs/0.ref/DDDPractical/33-checklist-bat-dau-du-an-voi-ddd.md`
- Discovery
- Event Storming
- Chốt Strategic Design
- Trước khi code
- Trước khi productionize

---

## 34. Kết luận: DDD là hành trình, không phải đích đến

- Nội dung: `design/docs/0.ref/DDDPractical/34-ket-luan-ddd-la-hanh-trinh.md`
- DDD là công cụ tư duy
- Giá trị lâu dài
- Vai trò của Architect & Dev

---

# PHỤ LỤC (để handbook “dùng được ngay”)

**Event Storming**
- Event Storming checklist: `design/docs/0.ref/DDDPractical/checklist-event-storming.md`
- Event Storming toolkit: `design/docs/0.ref/DDDPractical/toolkit-event-storming.md`

**Checklists**
- Strategic Design checklist: `design/docs/0.ref/DDDPractical/checklist-strategic-design.md`
- Tactical Design checklist: `design/docs/0.ref/DDDPractical/checklist-tactical-design.md`
- Integration checklist: `design/docs/0.ref/DDDPractical/checklist-integration.md`
- Code review checklist (DDD): `design/docs/0.ref/DDDPractical/checklist-code-review-ddd.md`

**Glossary**
- Glossary (DDD & ADLP): `design/docs/0.ref/DDDPractical/glossary.md`

**Templates**
- Template ADR: `design/docs/0.ref/DDDPractical/template-adr.md`
- Template Event schema + versioning: `design/docs/0.ref/DDDPractical/template-event-schema-versioning.md`
- Template Workshop agenda + output pack: `design/docs/0.ref/DDDPractical/template-workshop-artefact-pack.md`
- Templates (tổng hợp): `design/docs/0.ref/DDDPractical/templates.md`
