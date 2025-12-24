# Chương 28 — Từ Use Case đến Code: Command Handling, Validation, Transaction, và Publishing Events (làm đúng ngay từ ngày đầu)

Ở giai đoạn “đã chốt domain”, nhiều dự án DDD lại thất bại vì một vấn đề tưởng nhỏ: **không có một chuẩn thực dụng để biến use case thành code**. Kết quả là:
- domain rules rò rỉ vào controller,
- transaction boundary rải rác, lúc commit lúc không,
- event publish “bắn đại”, không đảm bảo đúng–một–lần,
- validation và authorization bị trộn lẫn, debug không ra vì sao bị từ chối.

DDD không yêu cầu bạn dùng framework nào. Nó yêu cầu bạn giữ một lời hứa: **quy tắc nghiệp vụ phải sống ở nơi có thể bảo vệ và kiểm thử**.

Chương này cho bạn một “xương sống triển khai” (implementation spine) đủ đơn giản để dùng cho MVP, nhưng đủ kỷ luật để bạn không phải rewrite khi hệ thống lớn lên.

Ví dụ xuyên suốt: ADLP (AI Data Labeling Platform) theo `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`. Ta sẽ bám một use case rất điển hình: **Labeler Submit Transcript** — vì nó chạm cả correctness, concurrency, audit, và integration sang Quality/Export/Payout.

---

## Bạn sẽ nhận được gì sau chương này?

1) Một “template tinh thần” để triển khai bất kỳ use case nào theo DDD (command → domain → persistence → events).  
2) Cách phân lớp trách nhiệm: controller vs application service/handler vs domain model vs infrastructure.  
3) Kỷ luật transaction boundary + outbox/publish-after-commit để tránh “đúng trong DB, sai ngoài đời”.  
4) Best practices + trade-offs + anti-patterns khi hiện thực DDD.  
5) Áp dụng vào ADLP: triển khai skeleton cho `SubmitTranscript` và mapping sang các use case khác.  
6) Exercise có hướng dẫn: bạn tự triển khai một use case “đắt tiền” và tự kiểm bằng checklist.

---

## 1) Use case trong DDD: nó là gì, và vì sao nó phải “đứng giữa”

Trong DDD thực dụng, bạn sẽ thấy bốn “vùng trách nhiệm”:
- **Interfaces**: nhận request, trả response (HTTP/gRPC/CLI/consumer).
- **Application**: điều phối use case (orchestration trong một bounded context).
- **Domain**: nơi sống của invariants (rules phải luôn đúng).
- **Infrastructure**: DB, message bus, external systems, caches.

Use case nằm ở application layer. Nó có một nhiệm vụ không glamorous nhưng cực quan trọng:
> “Đưa hệ thống từ trạng thái A sang trạng thái B theo đúng luật của domain, với transaction boundary rõ ràng, và phát tín hiệu cho bên ngoài.”

Nếu bạn không có cấu trúc rõ, bạn sẽ trượt về hai cực:
- Anemic domain model: domain chỉ là DTO, mọi rule nằm trong service.
- “Domain-god”: domain cố gắng gọi DB, gọi HTTP, tự publish event… rồi bạn không test được.

---

## 2) Xương sống triển khai (Implementation Spine) cho 1 use case

Hãy ghi nhớ một chuỗi hành động, và cố giữ nó giống nhau cho mọi use case:

1) **Parse & authenticate** (interfaces)  
2) **Authorize + validate input** (application, và một phần domain validation)  
3) **Load aggregate(s)** (repository)  
4) **Execute domain method** (enforce invariants)  
5) **Persist** (unit of work / transaction)  
6) **Publish domain/integration events** (sau commit hoặc via outbox)  
7) **Return** (interfaces)  

Nghe đơn giản, nhưng 80% lỗi production nằm ở việc bạn phá chuỗi này: validate sau khi đã write, publish trước khi commit, hoặc “quên” idempotency.

> **NOTE**  
> Bạn không cần framework phức tạp. Bạn cần kỷ luật: “một use case đi qua một spine”.

---

## 3) Ví dụ ADLP: Use case “Submit Transcript” (từ câu chuyện đến code)

### 3.1 Câu chuyện nghiệp vụ (không nói về code)

Labeler đang làm một batch. Họ chỉnh transcript theo segments và bấm Submit. Khi submit:
- hệ thống phải đảm bảo batch đang thuộc lock của họ (không submit hộ người khác),
- batch chưa bị hết hạn hoặc bị revoke,
- transcript đáp ứng rule tối thiểu (không rỗng, format hợp lệ, …),
- hệ thống ghi nhận một “submission snapshot” để Quality đánh giá,
- và phát tín hiệu để Quality bắt đầu chấm (async).

Trong ngôn ngữ domain, bạn muốn nói: `SubmitTranscript(batch_id, labeler_id, payload)` dẫn đến `TranscriptSubmitted`.

### 3.2 Chia trách nhiệm đúng chỗ (bản đồ tư duy)

- Controller (Interfaces): nhận HTTP request, map sang command, trả HTTP response.
- Command Handler (Application): authorize, load aggregate, gọi domain, commit, publish.
- Batch aggregate (Domain): kiểm tra invariant “được submit không”, chuyển trạng thái, tạo domain event.
- Repository/UoW (Infrastructure): load/save + transaction boundary.
- Outbox/Publisher (Infrastructure): đảm bảo publish đúng cách.

Đây là điểm mấu chốt: **domain không biết HTTP**, controller không biết invariants chi tiết.

---

## 4) Command Handling: “ý định” phải rõ, và phải idempotent khi cần

### 4.1 Command là “request thay đổi”, không phải “DTO update”

Một command tốt mang theo:
- danh tính actor (labeler_id),
- danh tính target (batch_id),
- dữ liệu cần thiết cho quyết định domain (transcript payload + metadata),
- idempotency key (nếu client có thể retry).

Trong ADLP, `submit` là loại command mà client rất hay retry (mạng chập chờn, timeout). Nếu bạn không thiết kế idempotency, bạn sẽ gặp:
- double submit,
- double quality job,
- hoặc state “submitted” nhưng event publish bị mất.

> **BEST PRACTICE**  
> Với các command “đắt tiền” (submit, accept, export, payout), hãy hỗ trợ idempotency key và lưu “command receipt”.

### 4.2 Một skeleton (pseudo-code, không phụ thuộc ngôn ngữ)

```text
SubmitTranscriptCommand {
  command_id / idempotency_key
  batch_id
  labeler_id
  transcript_payload
}
```

Application handler phải đảm bảo: cùng `idempotency_key` → cùng kết quả (hoặc cùng lỗi domain), không tạo side effects mới.

---

## 5) Validation: cái gì validate ở đâu?

Validation thường bị làm sai ở hai hướng:
- validate tất cả ở controller (rồi domain chỉ “nuốt” dữ liệu),
- hoặc để domain validate cả “HTTP shape” (lẫn lộn tầng).

Cách thực dụng:

### 5.1 Input validation (application/interfaces)

Những thứ thuộc “định dạng request”:
- field thiếu, sai type,
- size limit, format JSON,
- constraints kỹ thuật (max payload size, allowed charset).

Đây là “không đúng shape” → trả lỗi sớm.

### 5.2 Domain validation (domain model)

Những thứ thuộc “đúng nghĩa nghiệp vụ”:
- batch có đang ở trạng thái cho phép submit không?
- labeler có sở hữu lock không?
- transcript có đáp ứng rule tối thiểu không? (ví dụ policy: không để trống, không chứa ký tự cấm)

Đây là “đúng shape nhưng không được phép” → trả domain rejection (error_code rõ).

> **NOTE**  
> Nếu bạn để domain validation ở controller, bạn sẽ copy/paste rule và sẽ sai khác nhau theo thời gian.

---

## 6) Transaction boundary: “một use case = một transaction” (và khi nào phá lệ)

Trong đa số bounded context, một use case nên nằm trong một transaction:
- load aggregate,
- apply rule,
- save,
- commit.

Nếu bạn commit giữa chừng, invariants có thể bị phá vì state trung gian bị lộ.

### 6.1 Concurrency: optimistic locking và invariants

Use case `SubmitTranscript` rất dễ có race:
- labeler submit 2 lần do double click,
- TTL lock expire đúng lúc submit,
- admin revoke batch đúng lúc submit.

Bạn cần một cơ chế để “đụng nhau thì phát hiện”:
- optimistic locking (version field trên aggregate),
- hoặc pessimistic lock (ít dùng hơn, đắt hơn).

DDD không ép bạn chọn cái nào, nhưng bắt bạn trả lời: “nếu concurrent thì hệ thống cư xử ra sao?”

> **BEST PRACTICE**  
> Bắt đầu bằng optimistic locking cho aggregate roots; chỉ dùng pessimistic khi bạn chứng minh được contention và có lợi ích rõ.

### 6.2 Khi nào “một use case” không gói trong một transaction?

Khi use case cần orchestration dài (nhiều context) hoặc có bước async (Quality, Export, Payout). Lúc đó:
- bounded context vẫn commit transaction của nó,
- phần còn lại đi qua integration events / process manager.

Đừng cố kéo nhiều context vào một distributed transaction. Chương 24/26 đã giải thích trade-off: correctness + resilience phải được thiết kế bằng idempotency, retries, SLO.

---

## 7) Publishing events: đừng “bắn event” trước khi commit

Đây là lỗi kinh điển:
1) Save DB
2) Publish event
3) Publish fail (network)
4) DB đã commit → consumer không bao giờ biết.

Hoặc tệ hơn:
1) Publish event
2) DB commit fail
3) Consumer xử lý một điều chưa xảy ra.

Có hai cách thực dụng:

### 7.1 Publish-after-commit (đơn giản, phù hợp MVP)

- Commit transaction
- Sau commit, publish event
- Nếu publish fail, bạn cần retry/compensation (vẫn có rủi ro “mất event” nếu process chết)

### 7.2 Outbox pattern (khuyến nghị cho event-driven nghiêm túc)

- Trong cùng transaction, write state + write outbox record
- Một publisher job đọc outbox và publish
- Đảm bảo “ít nhất một lần” (at-least-once), kết hợp idempotency ở consumer

Trong ADLP, những event liên quan payout/export/quality là “đắt tiền” → outbox đáng giá.

> **WARNING**  
> “Publish trực tiếp trong transaction” thường tạo coupling tới message broker, làm transaction chậm và khó debug. Outbox giúp bạn tách coupling ra.

---

## 8) Best practices (kèm giải thích)

### 8.1 Controller mỏng, handler dày vừa đủ, domain dày đúng chỗ

Controller chỉ nên:
- map request → command,
- xử lý auth token (hoặc middleware),
- trả response + correlation_id.

Handler chịu trách nhiệm:
- authorize,
- load/save,
- transaction,
- publish.

Domain chịu trách nhiệm:
- invariants và state transitions.

Nếu controller “biết” domain rules, bạn sẽ khó test và khó reuse (consumer/event handler cũng cần rule đó).

### 8.2 Đưa invariants vào domain method (không rải if-else)

Một aggregate method tốt làm ba việc:
1) kiểm tra điều kiện,
2) đổi state,
3) tạo event.

Điều này làm bạn test được hành vi domain như một “máy trạng thái có luật”.

### 8.3 Định nghĩa error_code theo domain (để client và retry policy đúng)

Error không chỉ để hiển thị. Nó quyết định:
- có retry không,
- có re-claim batch không,
- có escalation không.

Đừng để error model thành `500` “cho nhanh”.

### 8.4 Correlation ID là bắt buộc cho workflow dài

Ngay từ handler:
- lấy correlation_id từ request (hoặc tạo mới),
- propagate vào domain event / integration event.

Sau này bạn mới debug end-to-end được (Chương 26).

---

## 9) Trade-offs: DDD “đúng” nhưng không được làm chậm đội

### 9.1 Kỷ luật có chi phí ban đầu

Bạn sẽ viết nhiều “boilerplate” hơn:
- command object,
- handler,
- repository interface,
- unit of work,
- error codes.

Nhưng đổi lại:
- logic không rò rỉ,
- test dễ,
- refactor ít đau,
- production ít “bất ngờ”.

### 9.2 Outbox không miễn phí

Outbox thêm:
- bảng outbox,
- publisher worker,
- DLQ/retry.

Nhưng nếu bạn làm event-driven nghiêm túc, outbox thường rẻ hơn so với debug “mất event” trong production.

### 9.3 Optimistic locking cần thiết kế UX

Khi conflict xảy ra, client phải:
- refresh view,
- retry,
- hoặc hiển thị “batch đã đổi trạng thái”.

Không có “free lunch” cho concurrency.

---

## 10) Anti-patterns (triệu chứng → hậu quả → cách tránh)

### 10.1 “God Service” điều khiển mọi thứ

**Triệu chứng:** application service chứa 500 dòng if-else; domain entities chỉ có getter/setter.  
**Hậu quả:** invariants rò rỉ, test khó, refactor đau.  
**Cách tránh:** chuyển rule vào aggregate methods/VO; handler chỉ điều phối.

### 10.2 Transaction rải rác

**Triệu chứng:** `save()` nhiều lần, commit ở nhiều nơi, gọi HTTP trong transaction.  
**Hậu quả:** deadlock, partial update, inconsistent state.  
**Cách tránh:** một UoW/transaction cho use case; side effects sau commit; dùng outbox.

### 10.3 Publish event “bắn đại”

**Triệu chứng:** publish trước commit, không có outbox, không idempotency.  
**Hậu quả:** mất event, duplicate side effects, workflow vỡ âm thầm.  
**Cách tránh:** publish-after-commit hoặc outbox + consumer idempotency.

### 10.4 Validation đặt sai tầng

**Triệu chứng:** controller có domain rules; domain validate JSON shape.  
**Hậu quả:** rule bị copy; lỗi khó hiểu; khó reuse khi consumer khác gọi.  
**Cách tránh:** input validation ở interfaces/app; domain validation ở domain.

---

## 11) Áp dụng vào ADLP: mapping một use case thành “bộ artefacts”

Với `SubmitTranscript`, bạn nên ra được tối thiểu:
1) Command: `SubmitTranscriptCommand` (có idempotency key).  
2) Domain method: `Batch.submitTranscript(...)` (enforce invariants).  
3) Domain event: `TranscriptSubmitted` (owner context: Labeling).  
4) Integration event: `BatchSubmitted` hoặc `TranscriptSubmitted` published language cho Quality (tùy governance).  
5) Error codes: `LOCK_EXPIRED`, `ALREADY_SUBMITTED`, `BATCH_REVOKED`, `VALIDATION_FAILED`.  
6) Transaction boundary: UoW commit + outbox record.  
7) Observability: correlation_id propagate; metrics submit success/fail; lag nếu async.

Bạn có thể áp dụng y nguyên spine này cho:
- `ClaimBatch` (Assignment)
- `AcceptBatch` (Quality)
- `CreateExport` (Export)
- `CreditEarning` (Wallet)

Chỉ khác invariants và mức độ “đắt tiền” (để quyết định outbox, retries, DLQ).

355: ---

### 11.1 Full Code Example (Python-like)

```python
# 1. Command
@dataclass
class SubmitTranscriptCommand:
    batch_id: UUID
    labeler_id: UUID
    transcript_payload: dict
    idempotency_key: str

# 2. Handler (Application Layer)
class SubmitTranscriptHandler:
    def __init__(self, uow: UnitOfWork, outbox: Outbox):
        self.uow = uow
        self.outbox = outbox

    def handle(self, cmd: SubmitTranscriptCommand):
        with self.uow:
            # A. Load Aggregate
            batch = self.uow.batches.get(cmd.batch_id)
            if not batch:
                raise BatchNotFound(cmd.batch_id)

            # B. Domain Logic (Invariants Check & State Change)
            # This method raises DomainError if logic fails (e.g. lock expired)
            # It also adds 'TranscriptSubmitted' event to batch.changes
            batch.submit_transcript(cmd.labeler_id, cmd.transcript_payload)

            # C. Save State
            self.uow.batches.save(batch)

            # D. Publish/Outbox Integration Event
            # Transform Domain Event -> Integration Event (Public Contract)
            integration_event = BatchSubmitted(
                event_id=uuid.uuid4(),
                batch_id=str(batch.id),
                labeler_id=str(cmd.labeler_id),
                occurred_at=datetime.utcnow().isoformat(),
                payload_summary=summarize(cmd.transcript_payload)
            )
            
            self.outbox.add(topic="quality.batch.submitted", 
                            key=cmd.batch_id, 
                            payload=integration_event)
            
            # E. Commit Transaction
            self.uow.commit()

# 3. Domain Model (Batch Aggregate)
class Batch(AggregateRoot):
    def submit_transcript(self, labeler_id, payload):
         if self.status != BatchStatus.ASSIGNED:
             raise DomainError("Batch not in assigned state")
         
         if self.assignment.worker_id != labeler_id:
             raise DomainError("Not owned by labeler")

         if self.assignment.is_expired(datetime.utcnow()):
             raise DomainError("Lock expired")

         # Validation on payload content checks
         if not payload.get('segments'):
             raise DomainError("Empty transcript")

         self.status = BatchStatus.SUBMITTED
         self.transcript = payload
         self.add_domain_event(TranscriptSubmitted(self.id, labeler_id))
```

---

## 12) Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Use case được viết thành command rõ ý định (không phải DTO update)  
> - [ ] Input validation (shape) tách khỏi domain validation (meaning)  
> - [ ] Aggregate root enforce invariants qua domain method (không rải if-else)  
> - [ ] Một use case có một transaction boundary rõ  
> - [ ] Concurrency strategy rõ (optimistic locking / conflict behavior)  
> - [ ] Event publish không xảy ra trước commit; ưu tiên outbox cho workflow đắt tiền  
> - [ ] Error_code theo domain + retryable rules  
> - [ ] Correlation_id propagate xuyên request/events  

---

## 13) Artefacts/Deliverables

Sau chương này, bạn nên có một “Implementation Pack” cho mỗi bounded context quan trọng:
1) **Use case catalog**: danh sách command + owner + invariants chính.  
2) **Error taxonomy**: error_code + semantics + retryability.  
3) **Transaction + event publishing policy** (outbox/after-commit) theo mức độ critical.  
4) **Idempotency policy** cho các command đắt tiền.  
5) **Observability conventions**: correlation_id, log fields, metrics tối thiểu.

---

## 14) Exercise (có hướng dẫn) — Triển khai skeleton cho “Accept Batch” (Quality Assurance)

### Bài toán
Trong ADLP, Quality Assurance có use case `AcceptBatch` (hoặc `ApproveQualityDecision`): khi batch đạt chuẩn, hệ thống accept để:
- cho phép Export đóng gói dataset,
- kích hoạt Wallet credit earning (hoặc enqueue payout workflow),
- ghi audit trail.

Bạn cần viết skeleton code (không cần chạy thật) theo đúng “implementation spine”.

### Cách làm (gợi ý từng bước)

**Bước 1 — Viết invariants (3 dòng là đủ)**  
Gợi ý:
- batch chỉ accept nếu đã có kết quả evaluate hợp lệ theo policy_version,
- không accept nếu đã accept/reject trước đó,
- actor phải có quyền (reviewer/admin).

**Bước 2 — Định nghĩa command + error_code**  
Command: `AcceptBatchCommand(batch_id, reviewer_id, idempotency_key, reason)`  
Error_code gợi ý: `ALREADY_DECIDED`, `EVALUATION_MISSING`, `POLICY_MISMATCH`, `UNAUTHORIZED`.

**Bước 3 — Định nghĩa aggregate method**  
`QualityDecision.accept(...)` hoặc `QualityEvaluation.accept(...)` (tùy model).  
Method phải: validate → change state → produce domain event.

**Bước 4 — Viết handler theo xương sống**  
Load aggregate → call domain → save → commit → write outbox/publish event.

**Bước 5 — Chọn published event và consumer**  
Event: `BatchAccepted` (owner: Quality)  
Consumers: Export, Wallet/Payment.  
Ghi rõ idempotency key gợi ý cho consumer: `batch_id + decision_version`.

**Bước 6 — Tự kiểm bằng checklist**  
Bạn có publish trước commit không?  
Bạn có retry policy + idempotency không?  
Bạn có correlation_id không?

### Output mong đợi (nộp bài)
1) Một trang mô tả use case: command, invariants, errors, events.  
2) Một pseudo-code skeleton cho handler + domain method (10–30 dòng).  
3) Một đoạn “operational note”: metric/alert nào chứng minh use case này đang chạy đúng (theo tinh thần Chương 26).

