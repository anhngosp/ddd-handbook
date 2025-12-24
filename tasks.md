# Task List — Viết nội dung Handbook DDD Practical

Mục tiêu: hoàn thiện nội dung cho `design/docs/0.ref/DDDPractical/outline-handbook.md` theo đúng outline, văn phong handbook, ưu tiên tính “dùng được ngay” (templates, checklist, ví dụ, warning box).

---

## 0) Chuẩn bị & nguyên tắc biên soạn
- [x] Chốt đối tượng độc giả (Architect/Lead/Tech Lead) và mức độ “thực hành” (ít lý thuyết, nhiều bước làm)
- [x] Chốt quy ước trình bày (heading, callout blocks: NOTE/WARNING/EXAMPLE/CHECKLIST)
- [x] Chuẩn hóa thuật ngữ (Việt/Anh) + quy tắc dùng thuật ngữ DDD (Bounded Context, Aggregate, Domain Event…)
- [x] Soạn “prompting guide” để viết handbook xuyên session
- [x] Tạo “template box” dùng lại: Workshop agenda, ADR template, Event schema template, Checklist template
- [x] Định nghĩa tiêu chí hoàn thành (DoD) cho mỗi chương: có mục tiêu, bước làm, output artefact, lỗi thường gặp

---

## PHẦN I — TƯ DUY NỀN TẢNG
- [x] Viết Chương 1: Vì sao DDD vẫn quan trọng (problem framing, chi phí hiểu sai domain, ví dụ ngắn)
- [x] Viết Chương 2: Myths & Anti-patterns (khi nào KHÔNG dùng DDD + dấu hiệu cảnh báo over-engineering)
- [x] Viết Chương 3: Bản đồ toàn cảnh quy trình (pipeline, ai tham gia, artefacts, “output→input”)

---

## PHẦN II — KHÁM PHÁ MIỀN (DOMAIN DISCOVERY)
- [x] Viết Chương 4: Hiểu domain trước kiến trúc (Core/Supporting/Generic + cách nhận diện + ví dụ)
- [x] Viết Chương 5: Domain Expert (ai là expert, cách làm việc, anti-pattern suy diễn nghiệp vụ)
- [x] Soạn checklist Discovery (câu hỏi phỏng vấn, artefacts tối thiểu, rủi ro thường gặp)

---

## PHẦN III — EVENT STORMING
- [x] Viết Chương 6: Event Storming là gì (event-centric, khi dùng, kết quả mong đợi)
- [x] Viết Chương 7: Các cấp độ (Big Picture / Process / Design-level) + khi nào dừng/tiến cấp
- [x] Viết Chương 8: Chuẩn bị workshop (vai trò, luật chơi, công cụ, agenda mẫu 2–3 giờ)
- [x] Viết Chương 9: Big Picture thực hành (cách đặt câu hỏi, xử lý tranh cãi, hotspots/questions)
- [x] Viết Chương 10: Process-level thực hành (commands/policies/actors/external systems, time boundaries)
- [x] Viết Chương 11: Design-level (aggregate candidates, invariants, transaction boundary, edge-cases)
- [x] Viết Chương 12: Chốt Ubiquitous Language (glossary, đồng nghĩa/đa nghĩa, tách theo context)
- [x] Tạo “Event Storming toolkit” (checklist + mẫu bảng thuật ngữ + mẫu tổng hợp hotspots)

---

## PHẦN IV — STRATEGIC DESIGN
- [x] Viết Chương 13: Strategic vs Tactical (điểm dừng hợp lý, quyết định đắt tiền)
- [x] Viết Chương 14: Bounded Context (tiêu chí, dấu hiệu tốt/xấu, anti-pattern chia theo DB/UI/CRUD)
- [x] Viết Chương 15: Context Map (patterns + cách chọn + trade-off REST vs events vs ACL)
- [x] Viết Chương 16: Strategic Design → kiến trúc/team/roadmap (team topology, đầu tư theo core domain)
- [x] Viết Chương 17: ADRs (khi nào cần, template ADR, ví dụ quyết định chiến lược)
- [x] Soạn checklist Strategic Design (BC catalog, context map, ownership, contracts, risks)

---

## PHẦN V — TACTICAL DESIGN
- [x] Viết Chương 18: Khi nào bắt đầu tactical (sẵn sàng, làm theo từng BC)
- [x] Viết Chương 19: Aggregate (invariants, boundary, sai lầm God Aggregate, mapping theo DB)
- [x] Viết Chương 20: Entity/VO/Domain Service (phân biệt + ví dụ thực tế)
- [x] Viết Chương 21: Domain Events & Consistency (Domain vs Integration, eventual consistency, saga/process manager, versioning)
- [x] Viết Chương 22: Repository & Factory (persistence ignorance, thiết kế interface, khi nào cần)
- [x] Soạn checklist Tactical Design (invariants, commands/events, consistency model, tests)

---

## PHẦN VI — SYSTEM & SOLUTION DESIGN
- [x] Viết Chương 23: Mapping Domain Model → kiến trúc (BC ↔ deployment unit, microservices vs modular monolith)
- [x] Viết Chương 24: Integration design (sync/async, contracts, schema versioning, idempotency/retries, choreography vs orchestration)
- [x] Viết Chương 25: Data & infrastructure (database per service, read models, event store khi cần, caching)
- [x] Viết Chương 26: NFR (scalability, observability, security, compliance, performance, resilience) + “NFR by design”
- [x] Soạn “integration checklist” (contract tests, DLQ/retry policy, tracing/correlation IDs)

---

## PHẦN VII — DETAILED DESIGN & IMPLEMENTATION
- [x] Viết Chương 27: API design theo domain (không leak domain, versioning, error model)
- [x] Viết Chương 28: Use case → code (command handling, validation, transaction, publish events)
- [x] Viết Chương 29: Tổ chức code theo DDD (layered/hexagonal, dependency rules, anti-pattern)
- [x] Viết Chương 30: Testing (unit domain, integration context, contract test, event-driven testing)
- [x] Soạn “code review checklist” cho DDD (domain purity, boundaries, invariants, coupling)

---

## PHẦN VIII — VẬN HÀNH & TIẾN HÓA
- [x] Viết Chương 31: DDD trong thực tế (business change, continuous discovery, refactor BC, dealing with legacy)
- [x] Viết Chương 32: Anti-patterns & bài học (over-engineering, DDD nửa mùa, event-driven sai cách)
- [x] Viết Chương 33: Checklist bắt đầu dự án mới (end-to-end: discovery → productionize)
- [x] Viết Chương 34: Kết luận (DDD là hành trình, kỷ luật, vai trò architect/dev)

---

## PHỤ LỤC (Dùng được ngay)
- [x] Event Storming checklist (chuẩn bị, chạy, tổng hợp, follow-up)
- [x] Strategic Design checklist (BC, context map, contracts, risks)
- [x] Tactical Design checklist (aggregate, invariants, events, consistency)
- [x] Glossary (thuật ngữ DDD + mapping Việt/Anh)
- [x] Template: ADR
- [x] Template: Event schema + versioning rules
- [x] Template: Workshop agenda + output artefact pack

---

## Kiểm tra chất lượng & hoàn thiện
- [x] Rà soát tính nhất quán thuật ngữ và cấu trúc chương/mục
- [x] Rà soát “actionability”: mỗi chương có steps + output + checklist
- [x] Bổ sung ví dụ xuyên suốt (1 domain mẫu chạy xuyên từ discovery → production)
- [x] Proofread (chính tả, định dạng markdown, link nội bộ)
- [x] Chốt phiên bản v1 (đóng scope) và ghi “Next iteration ideas”
