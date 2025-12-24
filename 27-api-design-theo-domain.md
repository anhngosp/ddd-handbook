# Chương 27 — API Design theo Domain: viết “hợp đồng” phản ánh ngôn ngữ nghiệp vụ (và không tự phá ranh giới)

Một dự án DDD thường chết theo hai cách:
1) Strategic/Tactical làm đúng, nhưng API làm sai → domain model bị “kéo méo” theo REST/CRUD, rồi team bắt đầu nói bằng tên bảng thay vì nói bằng nghiệp vụ.
2) API “đẹp kỹ thuật” nhưng không khớp ngôn ngữ business → người dùng/PO/DE không hiểu, integration bẻ cong, và cuối cùng mọi thứ quay về “gọi thẳng DB cho nhanh”.

Trong ADLP, điều này xảy ra rất tự nhiên. Vì ADLP có nhiều bounded contexts (Assignment, Labeling, Quality, Export, Wallet…), nên nếu API không phản ánh đúng ngôn ngữ của từng context, bạn sẽ:
- lộ domain nội bộ ra ngoài (rồi không thể đổi),
- tạo coupling ngầm (request chain dài, retry storm),
- hoặc “đi tắt” bằng cách cho consumer đọc DB của producer.

Chương này hướng dẫn bạn thiết kế API như **một phần của DDD**:
- API là **Published Language** cho thế giới bên ngoài context.
- API phải bảo vệ **invariants** và **boundaries**.
- API phải kể được câu chuyện nghiệp vụ, không kể câu chuyện của ORM.

Ví dụ xuyên suốt: ADLP theo thiết kế mới nhất trong `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`.

---

## Bạn sẽ nhận được gì sau chương này?

1) Cách thiết kế API phản ánh ngôn ngữ domain (không biến thành CRUD theo bảng).  
2) Heuristics “khi nào API nên là query, khi nào nên là command”.  
3) Versioning strategy và error model để API tiến hóa mà không phá integration.  
4) Best practices + trade-offs + anti-patterns (những lỗi thường khiến hệ thống bị “khóa tay”).  
5) Áp dụng vào ADLP: phác thảo API cho 3 điểm nóng: Assignment, Labeling, Quality/Export.  
6) Exercise có hướng dẫn: viết API contract cho workflow “Premium 48h” sao cho không leak domain nội bộ.

---

## 1) API trong DDD: nó là “ngôn ngữ công bố”, không phải “cổng dữ liệu”

Nếu bounded context là một quốc gia, thì API là “hiến pháp + luật giao thông” của quốc gia đó đối với bên ngoài. Bên ngoài không có quyền biết:
- bạn lưu dữ liệu thế nào,
- bạn chia aggregate ra sao,
- bạn dùng transaction hay event sourcing,

…nhưng bên ngoài phải biết:
- họ được làm gì,
- điều gì bị cấm,
- và nếu làm sai thì lỗi nghĩa là gì.

Nói cách khác: **API là contract**, và contract phải nói bằng **ubiquitous language** của context.

> **NOTE**  
> Một API tốt giúp người đọc “đoán đúng” hành vi nghiệp vụ. Một API tệ khiến người đọc phải mở code/DB để hiểu.

### 1.1 Domain API khác gì Data API?

- Data API: “cho tôi bản ghi”, “cập nhật field”, “lọc theo cột”.
- Domain API: “tôi muốn thực hiện một hành động nghiệp vụ”, “hãy chấp nhận batch”, “hãy gán batch cho labeler”, “hãy submit transcript”.

Không có nghĩa là Domain API không có query. Nhưng query phục vụ **use cases** (màn hình, báo cáo, decision), không phục vụ “đọc bảng cho đủ”.

---

## 2) Nguyên tắc vàng: bắt đầu từ Use Case, không bắt đầu từ Resource

Một cách viết API dễ sai là: nhìn vào entity rồi tạo endpoint CRUD. Cách đúng là: nhìn vào workflow/use case và hỏi:
- Ai (actor) làm gì?
- Họ kỳ vọng điều gì xảy ra ngay lập tức?
- Điều gì xảy ra “sau đó” (async) và ai chịu trách nhiệm?

Trong ADLP, hãy thử kể câu chuyện của một labeler:
1) “Tôi cần được nhận việc” (assignment)
2) “Tôi cần mở batch, xem segments, sửa transcript” (labeling)
3) “Tôi cần submit để được chấm” (quality)
4) “Tôi muốn biết trạng thái: đã accepted chưa, payout khi nào?” (status)

API nên giúp câu chuyện này đi thẳng, thay vì bắt họ ghép 12 API CRUD.

> **EXAMPLE**  
> Domain story: “Claim a batch”  
> Nếu API là `POST /batches/{id}/claim` thì người đọc hiểu ngay: có lock, có quyền, có conflict.  
> Nếu API là `PATCH /batches/{id} { "status": "claimed", "assignee_id": "..." }` thì bạn đã biến nghiệp vụ thành “cập nhật bảng”.

---

## 3) Query vs Command: chọn đúng để bảo vệ invariants

DDD luôn có hai loại tương tác:
- **Query**: hỏi trạng thái (không thay đổi sự thật).
- **Command**: yêu cầu thay đổi (có thể bị từ chối vì invariant/authorization).

### 3.1 Command nên “được phép thất bại” theo ngôn ngữ nghiệp vụ

Trong ADLP, `SubmitTranscript` có thể bị từ chối vì:
- batch đã hết hạn,
- lock thuộc người khác,
- audio bị revoked,
- transcript không đạt chuẩn tối thiểu.

API cần biểu đạt được những lý do này để client xử lý đúng, thay vì chỉ trả `400`.

### 3.2 Query nên tối ưu cho trải nghiệm và hiệu năng (không kéo domain vào client)

Labeling UI cần dữ liệu tổng hợp: segments + signed URLs + trạng thái autosave. Nếu bạn bắt client gọi 6 endpoints và tự join, bạn đã “đẩy context map lên frontend”.

Trade-off thường gặp:
- Query endpoint “đậm” giúp UI nhanh, ít round-trip.
- Nhưng nếu query “đậm” phụ thuộc domain rules, bạn phải cẩn thận để không biến query thành “command trá hình”.

> **BEST PRACTICE**  
> Query endpoint có thể phục vụ “view model” (CQRS-lite), miễn là nó không cho phép “viết ngược” vào invariants.

---

## 4) Tránh “leak domain nội bộ”: những thứ không nên lộ qua API

Bạn sẽ rất dễ lộ các chi tiết sau:

### 4.1 Leaking persistence model

Ví dụ tệ:
- trả về `created_at`, `updated_at` khắp nơi như “đặc tính nghiệp vụ”
- trả về `row_version`, `internal_status_code`, `db_id`
- để client truyền `foreign_key_id` không có nghĩa business

Không phải cấm tuyệt đối. Nhưng hãy hỏi: “field này có ý nghĩa nghiệp vụ không?”. Nếu không, nó nên nằm trong header/metadata hoặc bị ẩn.

### 4.2 Leaking aggregate boundary

Nếu API bắt client cập nhật 3 resource để hoàn thành 1 hành động nghiệp vụ, có thể aggregate boundary đang bị lộ.

Trong ADLP, “submit” thường là một hành động atomic về mặt nghiệp vụ:
- labeler submit transcript cho batch
- hệ thống đóng snapshot, đẩy sang Quality

Nếu API yêu cầu:
1) PATCH transcript
2) PATCH batch status
3) POST event “submitted”

…thì invariants đã bị phân mảnh ra client.

### 4.3 Leaking orchestration

Client không nên phải biết “gọi service A trước, rồi service B, rồi service C”.
Nếu cần orchestration, hãy đặt nó ở application layer (process manager) hoặc facade phù hợp, không đặt lên client.

> **WARNING**  
> Leaking orchestration là con đường nhanh nhất tạo distributed monolith: UI trở thành orchestrator, còn backend chỉ là “data pipes”.

---

## 5) Versioning strategy: “đổi mà không phá”, và phá khi cần phá

Bạn không thể hứa “API mãi mãi không đổi”. Nhưng bạn có thể hứa: **API thay đổi có kỷ luật**.

### 5.1 Thay đổi tương thích (backward-compatible)

Ví dụ:
- thêm field optional,
- thêm endpoint mới,
- thêm enum value (nếu client tolerant),
- thêm filter/sort.

Những thay đổi này thường không cần bump major version, nhưng cần:
- documentation,
- contract tests,
- deprecation policy cho behavior cũ (nếu có).

### 5.2 Breaking change: chấp nhận nó là “quyết định đắt tiền”

Breaking change có thể là:
- đổi semantics của field,
- xóa field bắt buộc,
- đổi cách xử lý lỗi,
- đổi rule authorization.

Khi breaking, bạn nên:
- chạy song song v1/v2 trong thời gian đủ,
- đo adoption,
- có kế hoạch tắt v1.

> **NOTE**  
> Với event schema, bạn đã có semver rules trong `design/docs/0.ref/DDDPractical/templates.md`. Với REST/gRPC, bạn cũng cần kỷ luật tương tự, dù hình thức khác.

### 5.3 Chọn cơ chế versioning (thực dụng)

Bạn sẽ thấy nhiều lựa chọn: `/v1/...`, header version, content negotiation. Cái quan trọng không phải “đúng chuẩn”, mà là:
- team có duy trì được không,
- tooling/clients có làm được không,
- migration có rõ ràng không.

Trong bối cảnh handbook này, nguyên tắc thực dụng:
- Nếu bạn có nhiều client và governance phức tạp: cân nhắc `/v1` rõ ràng.
- Nếu internal-only và kiểm soát được client: header version có thể đủ.
- Dù chọn gì, hãy viết **deprecation policy**.

---

## 6) Error model: lỗi cũng là một phần của hợp đồng domain

Một API “DDD đúng” không chỉ trả dữ liệu đúng; nó phải trả **lý do từ chối** đúng.

### 6.1 Phân loại lỗi theo ý nghĩa nghiệp vụ

Trong ADLP, rất nhiều lỗi không phải “technical error”, mà là “domain rejection”:
- `LockExpired`
- `AlreadySubmitted`
- `BatchNotAssignable` (do skill mismatch hoặc trạng thái)
- `QualityGateFailed`

Nếu bạn trả tất cả là `400 Bad Request`, client sẽ không biết phải làm gì: retry? refresh? show message? escalate?

### 6.2 Một error envelope tối thiểu (gợi ý)

Bạn không cần tiêu chuẩn hóa quá sớm, nhưng cần nhất quán. Một ví dụ (tham khảo phong cách Problem Details):

```json
{
  "error_code": "LOCK_EXPIRED",
  "message": "Batch lock has expired. Please claim again.",
  "details": {
    "batch_id": "b_123",
    "lock_expired_at": "2025-12-20T10:00:00Z"
  },
  "correlation_id": "c_456"
}
```

Điểm quan trọng:
- `error_code` là thứ client dựa vào (không dựa vào message).
- `correlation_id` giúp debug end-to-end (gắn với tracing).

> **BEST PRACTICE**  
> Mỗi command endpoint nên định nghĩa rõ: lỗi nào retry được, lỗi nào không retry được. Nếu không, bạn sẽ tạo retry storm.

---

## 7) Best practices (kèm giải thích)

### 7.1 Đặt tên endpoint theo hành động nghiệp vụ khi cần

REST thuần khuyến khích resource noun. Nhưng domain có commands. Với DDD, hai thế giới gặp nhau ở chỗ: “resource” vẫn được dùng, nhưng hành động quan trọng nên là sub-resource hoặc command endpoint có tên nói thật.

Ví dụ tốt (Assignment):
- `POST /batches/{batch_id}/claim`
- `POST /batches/{batch_id}/release`
- `POST /batches/{batch_id}/reassign`

Vì sao tốt? Vì nó:
- thể hiện có rule và invariants,
- cho phép error model giàu ý nghĩa,
- tránh “PATCH status” vô nghĩa.

### 7.2 Tách rõ command vs query để tối ưu mỗi bên

Query có thể cache, paginate, denormalize. Command tập trung vào invariants và audit.

Trong ADLP:
- Query: “list my assigned batches”, “get labeling view”.
- Command: “submit transcript”, “accept/reject batch”.

### 7.3 Thiết kế idempotency cho command quan trọng

Những command có side effects đắt tiền (export/payout) nên hỗ trợ idempotency key. Nếu client retry do timeout, server vẫn chỉ thực hiện một lần.

> **EXAMPLE**  
> `POST /exports` với header `Idempotency-Key: <uuid>` để tránh tạo 2 export jobs.

### 7.4 Nhúng observability vào contract

Không đợi “ops làm sau”. Ít nhất:
- trả `correlation_id` cho client,
- log domain identifiers,
- dùng status code + error_code nhất quán.

### 7.5 Thiết kế API để “không cần gọi ngược DB producer”

Nếu consumer phải gọi back để lấy thêm dữ liệu, contract đang thiếu. Điều này đúng cho cả REST và event payload (đã nói ở Chương 24).

---

## 8) Trade-offs: API domain-rich có cái giá của nó

### 8.1 Domain-rich API có thể “dài hơn” CRUD

Bạn sẽ có nhiều endpoint hơn. Nhưng đổi lại:
- hành vi rõ,
- lỗi rõ,
- thay đổi ít phá,
- invariants được bảo vệ.

### 8.2 “View model API” có nguy cơ biến thành “god endpoint”

Query endpoint phục vụ UI rất dễ phình to. Cách giữ kỷ luật:
- define scope theo màn hình/use case,
- dùng pagination và “include” có kiểm soát,
- tách những phần có vòng đời khác nhau.

### 8.3 Versioning làm tăng chi phí governance

Bạn sẽ cần:
- contract tests,
- deprecation policy,
- tài liệu hóa.

Nhưng nếu không làm, chi phí sẽ quay về dưới dạng “sợ đổi vì sợ phá”.

---

## 9) Anti-patterns (triệu chứng → hậu quả → cách tránh)

### 9.1 CRUD API cho core domain

**Triệu chứng:** `PATCH /batch { status: ... }`, `PUT /transcript` như cập nhật bảng.  
**Hậu quả:** invariants bị phân tán, client phải biết “state machine”, lỗi mơ hồ.  
**Cách tránh:** chuyển thành commands, có error_code, có audit.

### 9.2 UI làm orchestrator

**Triệu chứng:** frontend gọi 5 service theo thứ tự; service fail ở bước 4 thì UI tự “đền bù”.  
**Hậu quả:** distributed monolith, khó test, khó retry, khó trace.  
**Cách tránh:** dùng process manager/orchestration ở backend (xem Chương 24), hoặc tạo facade cho use case.

### 9.3 Lộ schema/ID nội bộ

**Triệu chứng:** client truyền `db_user_id`, `segment_row_id`, join keys.  
**Hậu quả:** bạn không refactor được DB/model, mọi thay đổi là breaking.  
**Cách tránh:** dùng business identifiers, ẩn persistence details, provide query endpoint phù hợp.

### 9.4 Error “một màu”

**Triệu chứng:** mọi thứ là `400` hoặc `500`, message tùy hứng.  
**Hậu quả:** client retry sai, user không hiểu, debug mù.  
**Cách tránh:** error_code taxonomy + correlation_id + phân loại retryable.

---

## 10) Áp dụng vào ADLP: phác thảo API “đúng ngôn ngữ”

Dưới đây không phải spec đầy đủ, mà là cách bạn “đặt ngôn ngữ” đúng. Mục tiêu: đọc tên endpoint là hiểu business.

### 10.1 Task Assignment (core domain)

- Query:
  - “Lấy batch phù hợp để làm” (tìm/đề xuất): `GET /assignment/recommendations?labeler_id=...`
  - “Danh sách batch tôi đang giữ lock”: `GET /labelers/{labeler_id}/active-batches`
- Commands:
  - “Nhận batch”: `POST /batches/{batch_id}/claim`
  - “Trả batch”: `POST /batches/{batch_id}/release`
  - “Gia hạn lock” (nếu business cho phép): `POST /batches/{batch_id}/renew-lock`

Các lỗi domain nên có:
- `BATCH_NOT_ASSIGNABLE`
- `SKILL_MISMATCH`
- `LOCK_CONFLICT`
- `LOCK_EXPIRED`

### 10.2 Labeling (core-ish, UI heavy)

- Query (view model):
  - `GET /labeling/batches/{batch_id}/view` trả về:
    - segments + audio access (signed URLs hoặc tokens)
    - transcript draft
    - submission status
    - autosave checkpoint
- Commands:
  - `POST /labeling/batches/{batch_id}/autosave`
  - `POST /labeling/batches/{batch_id}/submit`

Điểm kỷ luật:
- autosave không được phá invariants của “submit”.
- submit phải idempotent (tránh double submit khi client timeout).

### 10.3 Quality + Export/Wallet (đắt tiền, cần audit)

- Quality commands:
  - `POST /quality/batches/{batch_id}/evaluate`
  - `POST /quality/batches/{batch_id}/accept`
  - `POST /quality/batches/{batch_id}/request-rework`
- Export commands:
  - `POST /exports` (idempotent) → tạo export job
  - `GET /exports/{export_id}` → trạng thái + signed download URL (khi ready)

Audit/trace:
- mọi quyết định accept/reject/payout phải có correlation_id, actor, reason (domain meaningful).

### 10.4 OpenAPI snippet (POST /batches/{batch_id}/claim)

```yaml
openapi: 3.0.3
paths:
  /batches/{batch_id}/claim:
    post:
      summary: Claim a batch for labeling
      operationId: claimBatch
      parameters:
        - name: batch_id
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [labeler_id, idempotency_key]
              properties:
                labeler_id:
                  type: string
                idempotency_key:
                  type: string
      responses:
        "200":
          description: Batch claimed
          content:
            application/json:
              schema:
                type: object
                properties:
                  batch_id: { type: string }
                  status: { type: string, example: "ASSIGNED" }
                  lock_expires_at: { type: string, format: date-time }
        "409":
          description: Domain rejection (lock conflict/expired)
          content:
            application/json:
              schema:
                type: object
                properties:
                  error_code: { type: string, example: "LOCK_CONFLICT" }
                  message: { type: string }
                  correlation_id: { type: string }
```

---

## 11) Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Mỗi endpoint được map tới một use case (màn hình/câu chuyện) rõ ràng  
> - [ ] Command endpoints có error_code theo domain + phân loại retryable  
> - [ ] Query endpoints không lộ orchestration; client không phải join 5 API  
> - [ ] Không lộ persistence keys/schema nội bộ  
> - [ ] Có versioning + deprecation policy (dù đơn giản)  
> - [ ] Có correlation_id trong response và log/tracing conventions  
> - [ ] Command quan trọng hỗ trợ idempotency key  

---

## 12) Artefacts/Deliverables

Sau chương này, tối thiểu bạn nên có:
1) **API catalog theo bounded context** (endpoints + use cases + owners).  
2) **Error taxonomy** (error_code list + retryable rules).  
3) **Versioning & deprecation policy** (REST/gRPC).  
4) **Contract test plan** (đặc biệt cho integrations quan trọng; tham chiếu `design/docs/0.ref/DDDPractical/checklist-integration.md`).  
5) **API style guide** (naming, pagination, idempotency, correlation).

---

## 13) Exercise (có hướng dẫn) — Thiết kế API contract cho workflow “Premium 48h”

### Bài toán
Bạn cần thiết kế API cho một phần workflow Premium 48h sao cho:
- không leak domain nội bộ,
- thể hiện rõ commands,
- có error model giúp client xử lý đúng,
- chuẩn bị cho eventual consistency (nếu có async bước sau).

Chọn scope hẹp để làm thật kỹ: **“Labeler claim batch → labeling → submit”**.

### Cách làm (gợi ý từng bước)

**Bước 1 — Viết câu chuyện bằng ngôn ngữ business**  
Viết 6–10 dòng mô tả: ai làm gì, điều gì có thể fail, điều gì là “không được phép”.

**Bước 2 — Tách query và command**  
Bạn cần tối thiểu:
- 1 query để UI load “labeling view”
- 2–3 command (claim, autosave, submit)

**Bước 3 — Định nghĩa error_code cho từng command**  
Gợi ý:
- `LOCK_CONFLICT`, `LOCK_EXPIRED`
- `ALREADY_SUBMITTED`
- `VALIDATION_FAILED` (kèm details)

**Bước 4 — Thêm idempotency + correlation**  
Chọn command nào cần idempotency (thường là submit).  
Quyết định correlation_id trả về ở đâu (header hay body) và cách propagate.

**Bước 5 — Viết “spec tối thiểu” (1 trang)**  
Mỗi endpoint viết:
- method + path
- request schema
- response schema
- error_code list
- retryable rules

### Đáp án tham khảo (khung, không phải bắt chước y chang)
- `POST /batches/{batch_id}/claim`
- `GET /labeling/batches/{batch_id}/view`
- `POST /labeling/batches/{batch_id}/autosave`
- `POST /labeling/batches/{batch_id}/submit` + `Idempotency-Key`

Tự kiểm:
- Client có phải PATCH status không? Nếu có, bạn đang rơi vào CRUD.
- Client có phải gọi 3 service để load view không? Nếu có, orchestration đang bị leak.
- Nếu submit timeout và client retry, có bị double submit không? Nếu có, thiếu idempotency.
