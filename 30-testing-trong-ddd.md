# Chương 30 — Testing trong DDD: test đúng thứ “đắt tiền”, đúng tầng, đúng ranh giới

Một hệ thống DDD “đẹp trên giấy” có thể chết rất nhanh trong production vì một lý do buồn cười: team không có chiến lược testing tương xứng với độ phức tạp của domain. Khi đó:
- bug không nằm ở thuật toán — nó nằm ở “luật nghiệp vụ” bị hiểu sai,
- bug không nằm ở một hàm — nó nằm ở tương tác giữa contexts (integration),
- bug không “crash” — nó “sai âm thầm” (double payout, double export, lock không hết hạn, event bị mất).

DDD làm bạn tách domain, tách contexts, tách integration. Điều đó giúp thiết kế bền, nhưng cũng đòi hỏi một kiểu test khác: **test theo ranh giới** và **test theo kịch bản nghiệp vụ**.

Ví dụ xuyên suốt: ADLP (AI Data Labeling Platform). Ta sẽ dùng những “điểm đắt tiền” làm nền:
- Assignment lock TTL (core correctness + concurrency),
- Submit/Accept (state machine + audit),
- Event-driven workflow (Quality → Export/Wallet) với idempotency/outbox,
- “Premium 48h” như một kịch bản end-to-end có SLO (liên kết Chương 26).

---

## Bạn sẽ nhận được gì sau chương này?

1) Một chiến lược test thực dụng cho DDD: test pyramid “đúng nghĩa” (domain-heavy, integration-smart).  
2) Cách viết unit test cho invariants/aggregates (không cần DB, không cần framework).  
3) Khi nào cần integration test (DB, message broker, outbox) và cách giữ nó không chậm.  
4) Contract testing cho REST và events để giảm coupling giữa contexts/services.  
5) Event-driven testing: idempotency, retries, DLQ, replay (những thứ hay “vỡ âm thầm”).  
6) Exercise có hướng dẫn: tự thiết kế test plan cho 1 workflow ADLP và tự kiểm bằng checklist.

---

## 1) “Test đúng thứ đắt tiền”: triết lý test trong DDD

Trong hệ thống CRUD đơn giản, bạn có thể sống với nhiều integration tests và ít unit tests. Nhưng trong DDD, thứ “đắt tiền” là:
- **ngôn ngữ và luật** (invariants),
- **ranh giới** (bounded contexts + contracts),
- **tương tác** (eventual consistency, idempotency, retries),
- **các tình huống hiếm** (race conditions, duplicate events, out-of-order).

Vì vậy, test trong DDD nên tối ưu theo mục tiêu:
1) Bắt lỗi hiểu sai domain càng sớm càng tốt (unit tests cho domain).  
2) Bắt lỗi “tương tác thật” ở những chỗ có side effects (integration tests chọn lọc).  
3) Bảo vệ contract giữa contexts (contract tests).  
4) Bảo vệ workflow event-driven khỏi “sai âm thầm” (event-driven tests + runbook checks).

> **NOTE**  
> “Coverage cao” không cứu bạn nếu bạn test sai thứ. DDD cần “độ phủ đúng điểm đau”.

---

## 2) Test Pyramid theo DDD: nhiều domain tests, ít end-to-end tests nhưng có “contract guardrails”

Hãy nghĩ test portfolio như một danh mục đầu tư:
- **Nhiều** unit tests cho domain (rẻ, nhanh, ổn định, bắt lỗi logic).
- **Vừa đủ** integration tests (đắt, chậm, nhưng bắt lỗi mapping/transaction/outbox).
- **Rất ít** end-to-end tests (đắt nhất, flaky, dùng để “smoke” và bảo vệ kịch bản thật sự critical).
- **Contract tests** như hàng rào: không phải “thêm tầng test”, mà là cách giảm nhu cầu e2e.

Một tỷ lệ thực dụng (tham khảo, không giáo điều):
- 60–80%: domain unit tests
- 15–30%: integration tests (DB/outbox/broker)
- 5–10%: e2e/smoke tests
- contract tests: tuỳ mức độ integration (nhưng “quan trọng hơn số lượng”)

---

## 3) Unit test cho Domain: invariants, state machine, và “đúng nghĩa nghiệp vụ”

### 3.1 Unit test domain nên test cái gì?

Trong DDD, unit test domain tập trung vào:
- invariants (“luật phải luôn đúng”),
- state transitions (command → event → state),
- value objects (validation/normalization),
- domain errors (rejection đúng lý do).

Ví dụ ADLP (Assignment):
- invariant: “một batch chỉ có một lock active tại một thời điểm”
- transition: `ClaimBatch` → `BatchClaimed` và state lock set
- rejection: `LockConflict`, `NotAssignable`

Ví dụ ADLP (Quality):
- invariant: “batch chỉ accept khi evaluation hợp lệ theo policy_version”
- rejection: `EvaluationMissing`, `AlreadyDecided`, `PolicyMismatch`

### 3.2 Unit test domain không nên test cái gì?

Không test:
- ORM mapping,
- SQL query,
- HTTP status codes,
- message broker delivery,

…vì đó không phải domain. Đưa những thứ đó sang integration/contract tests.

> **BEST PRACTICE**  
> Domain unit tests phải chạy được không cần DB/network, và deterministic (có `Clock`/random qua port nếu cần).

### 3.3 Một mẫu unit test “đúng chất DDD” (pseudo)

Bạn không cần ngôn ngữ nào cụ thể; mục tiêu là mạch:
1) given state
2) when command
3) then state + event + no side effects

```python
def test_claim_batch_success():
    # 1. Given: Batch available
    batch = BatchFactory.create_available(id="b1", tier="premium")
    labeler = Labeler(id="l1", tier="premium")
    t0 = datetime(2024, 1, 1, 10, 0, 0)

    # 2. When: Claim command
    batch.claim(labeler, now=t0, ttl_minutes=60)

    # 3. Then: State set & Event emitted
    assert batch.status == BatchStatus.ASSIGNED
    assert batch.lock.owner_id == "l1"
    assert batch.lock.expires_at == datetime(2024, 1, 1, 11, 0, 0)
    
    assert len(batch.events) == 1
    event = batch.events[0]
    assert isinstance(event, BatchClaimed)
    assert event.batch_id == "b1"
```

---

## 4) Integration tests: test “điểm tiếp xúc với thế giới thật”

Integration tests trong DDD nên chọn lọc. Chúng tồn tại để bắt lỗi ở ranh giới:
- repository implementation (mapping aggregate ↔ DB),
- transaction boundary + optimistic locking,
- outbox (write+publish semantics),
- event consumers (idempotency + side effects),
- external adapters (ACL).

### 4.1 Integration test cho repository + transaction

Trong ADLP, những lỗi hay xảy ra:
- optimistic lock không hoạt động → double claim,
- transaction boundary sai → state commit nhưng event không được ghi nhận,
- mapping VO sai → policy_version mất.

Integration test nên kiểm:
- save/load aggregate roundtrip giữ invariants data,
- concurrent update tạo conflict đúng (ví dụ version mismatch),
- transaction rollback không để lại outbox record “mồ côi”.

### 4.2 Integration test cho outbox

Outbox là nơi “sai âm thầm” thích xảy ra. Bạn nên có ít nhất:
- test: “commit state + outbox record trong cùng transaction”
- test: “publisher publish thành công → outbox mark published”
- test: “publisher fail → retry policy, không mất record”

```python
def test_outbox_written_in_transaction(uow, db_session):
    # Integration test with real DB
    handler = SubmitHandler(uow)
    handler.handle(cmd)

    # Check DB directly
    stored_batch = db_session.execute("SELECT status FROM batches...").fetchone()
    stored_outbox = db_session.execute("SELECT * FROM outbox...").fetchone()
    
    assert stored_batch.status == "SUBMITTED"
    assert stored_outbox is not None
    assert stored_outbox.topic == "quality.batch.submitted"
```

> **WARNING**  
> Nếu bạn dùng outbox nhưng không test outbox, bạn đang tạo một hộp Pandora: production sẽ dạy bạn bằng outage.

### 4.3 Integration test cho consumer idempotency

Với at-least-once delivery, duplicates là chuyện bình thường.

Trong ADLP:
- `BatchAccepted` consumer ở Wallet phải không double credit.
- `ExportRequested` consumer ở Export phải không tạo 2 export jobs.

Integration test nên mô phỏng:
1) consume same event twice
2) verify side effect chỉ xảy ra một lần (idempotency key đúng)
3) verify state/ledger/audit đúng

---

## 5) Contract tests: bảo vệ integration mà không cần chạy end-to-end

Contract tests trong DDD phục vụ một mục tiêu: **giữ coupling ở mức contract**, không ở mức “implementation vô tình”.

### 5.1 API contract tests (REST/gRPC)

Bạn nên có tests xác nhận:
- request/response schema (fields, types, optionality),
- error model (error_code semantics),
- backward compatibility (thêm field không phá),
- pagination/sort behavior (nếu là contract).

Đây là “lời hứa” giữa producer và consumer. Nó tránh việc consumer “đoán” behavior.

### 5.2 Event contract tests (Published Language)

Với events, contract còn quan trọng hơn REST, vì:
- consumer thường async → lỗi phát hiện muộn,
- event payload sai → workflow vỡ âm thầm.

Event contract test nên bảo vệ:
- envelope chuẩn (correlation/causation/version),
- schema versioning rules,
- consumer tolerant unknown fields,
- “payload tối thiểu nhưng đủ” (consumer không cần gọi ngược DB).

```python
def test_batch_accepted_event_schema_compatibility():
    # Use schema registry or JSON schema validator
    event_payload = {
        "event_id": "evt_123",
        "batch_id": "b_123",
        "dataset_version": "v2", 
        # "amount": 10.5  <-- removing this would break contract if consumer expects it
    }
    
    # Assert against Schema Registry
    errors = schema_registry.validate("BatchAccepted", "v1.0", event_payload)
    assert not errors, "Event payload must match Agreed Schema v1.0"
```

> **EXAMPLE**  
> Khi Quality publish `BatchAccepted`, Wallet cần đủ thông tin để credit: batch_id, labeler_id, amount hoặc reference để tính (tuỳ policy). Nếu thiếu, Wallet sẽ gọi ngược Quality DB — phá boundary.

### 5.3 Consumer-driven contract (CDC): khi nào đáng dùng?

CDC hữu ích khi:
- có nhiều consumer,
- domain thay đổi nhanh,
- bạn muốn consumer “đặt yêu cầu” cho contract.

Trade-off:
- governance phức tạp hơn,
- cần tooling/cadence.

DDD thực dụng: dùng CDC cho integration “đắt tiền” (Wallet, Assignment, Quality, Export), không cần lạm dụng cho mọi thứ.

---

## 6) Event-driven testing: test workflow “sai âm thầm” (retries, DLQ, out-of-order)

Đây là phần nhiều đội bỏ qua — và đó là lý do họ sợ event-driven.

### 6.1 Test retries và DLQ như một phần của correctness

Theo tinh thần ADR-007 (event processing SLO):
- bạn không chỉ test “consumer xử lý đúng khi mọi thứ tốt”
- bạn test “consumer cư xử đúng khi event fail 3 lần”:
  - nó có vào DLQ không?
  - owner có nhận alert không?
  - có runbook không?

### 6.2 Out-of-order và duplicates: phải có chiến lược

Ví dụ ADLP:
- `BatchSubmitted` có thể đến trước `BatchClaimed` (tuỳ pipeline, hoặc do replay).

Bạn cần quyết định:
- consumer có chấp nhận out-of-order và buffer không?
- hay consumer sẽ reject và đẩy vào DLQ?
- hay consumer sẽ query read model để xác minh?

Không có câu trả lời chung; nhưng phải có test cho chiến lược đã chọn.

### 6.3 Replay tests (nếu bạn dựa vào replay)

Nếu bạn nói “chúng ta có thể replay events để rebuild read model”, bạn cần một test tối thiểu:
- replay N events theo thứ tự → read model đúng
- replay lại lần 2 → read model không double count (idempotent)

---

## 7) Best practices (kèm giải thích)

### 7.1 Đặt tên test theo ngôn ngữ nghiệp vụ

Test name nên là câu chuyện:
- “cannot submit when lock expired”
- “credits earning exactly once for accepted batch”

Nó giúp người mới hiểu domain. Test trở thành documentation sống.

### 7.2 Test invariants bằng “table of examples”

Nhiều rule domain là “if policy then behavior”. Hãy dùng table-driven tests:
- policy_version v1/v2
- tier Standard/Premium
- deadline near/far

### 7.3 Đừng để integration tests thành “một cái ao”

Giữ integration tests:
- tối thiểu phụ thuộc,
- chạy được trong CI,
- có dữ liệu setup rõ,
- không cần chạy 10 services để test 1 repository.

### 7.4 Contract tests là điều kiện để versioning “đỡ đau”

Nếu bạn muốn API/events tiến hóa (Chương 27, 24), contract tests là dây an toàn.

### 7.5 Test observability ở chỗ critical

Bạn không cần assert mọi log line. Nhưng với workflow đắt tiền, hãy test:
- correlation_id được propagate vào event envelope,
- error_code mapping đúng,
- metrics counter increment cho success/fail (nếu bạn có abstraction).

---

## 8) Trade-offs: test càng “thật” càng đắt

### 8.1 End-to-end tests không phải kẻ thù, nhưng là tài sản đắt

E2E tests giúp bạn ngủ ngon cho 1–3 kịch bản critical. Nhưng:
- chậm,
- flaky,
- khó debug.

DDD tốt giúp bạn giảm phụ thuộc vào E2E bằng unit+contract.

### 8.2 CDC và schema registry làm governance nặng hơn

Nhưng nếu bạn có nhiều team/contexts, nó thường rẻ hơn so với “phá production”.

### 8.3 Event-driven tests cần mô phỏng nhiều failure modes

Đây là chi phí thật. Nhưng với payout/export, chi phí đó rẻ hơn lỗi tài chính/compliance.

---

## 9) Anti-patterns (triệu chứng → hậu quả → cách tránh)

### 9.1 Chỉ test controller/service, không test domain

**Triệu chứng:** unit tests chỉ cover endpoints; domain methods không có test.  
**Hậu quả:** invariants bị hiểu sai; refactor làm vỡ logic; bug “đắt tiền”.  
**Cách tránh:** domain-first tests: invariants/state machine/value objects.

### 9.2 “Mock mọi thứ” trong integration

**Triệu chứng:** integration test mock DB/broker → thực ra không test integration.  
**Hậu quả:** production mới lộ mapping/outbox lỗi.  
**Cách tránh:** chọn lọc integration tests chạy với DB thật hoặc testcontainers (tuỳ môi trường).

### 9.3 Không test idempotency

**Triệu chứng:** consumer retry nhưng side effects bị double.  
**Hậu quả:** double credit, double export, inconsistent ledger.  
**Cách tránh:** có test “consume twice” cho mỗi side effect đắt tiền.

### 9.4 Không test contract, chỉ rely on “team nói nhau”

**Triệu chứng:** đổi field làm consumer crash; event versioning lộn xộn.  
**Hậu quả:** distributed monolith và fear of change.  
**Cách tránh:** contract tests + versioning rules (semver) + deprecation plan.

---

## 10) Áp dụng vào ADLP: một test plan mẫu theo “điểm đắt tiền”

Bạn có thể dùng khung này như starter:

### 10.1 Task Assignment Context

- Domain unit tests:
  - claim/release/renew lock invariants
  - TTL expiry transitions
  - priority policy computations (VO)
- Integration tests:
  - optimistic locking prevents double claim
  - repository roundtrip preserves lock_until
- Contract tests:
  - API `POST /batches/{id}/claim` error_code semantics
  - event `BatchAssigned` schema compatibility

### 10.2 Labeling Context

- Domain unit tests:
  - submit only when lock owned and not expired
  - idempotent submit (same idempotency_key)
- Integration tests:
  - outbox record written on submit
  - replay submit event does not double create downstream job (if applicable)

### 10.3 Quality + Wallet/Export integration

- Consumer idempotency tests:
  - `BatchAccepted` consumed twice → Wallet credits once
  - `BatchAccepted` consumed twice → Export creates one job
- DLQ tests (smoke):
  - invalid payload → DLQ with reason + alert triggered

### 10.4 Premium 48h (smoke e2e)

Chỉ cần 1–2 smoke tests:
- một premium batch chạy qua “happy path”
- một failure mode (prelabeling backlog hoặc quality delay) để kiểm alerting/SLO breach behavior

---

## 11) Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Core domain có unit tests cho invariants/state transitions (không cần DB)  
> - [ ] Có integration tests cho repository + transaction boundary + optimistic locking  
> - [ ] Có outbox tests (write+publish semantics) nếu dùng event-driven  
> - [ ] Mỗi consumer đắt tiền có test idempotency (“consume twice”)  
> - [ ] Có contract tests cho REST/events (schema + error model + backward compatibility)  
> - [ ] Có ít nhất 1–2 smoke e2e cho kịch bản critical (Premium 48h)  
> - [ ] Có kiểm tra correlation_id propagation cho workflow dài  

---

## 12) Artefacts/Deliverables

Sau chương này, bạn nên có:
1) **Test strategy doc** theo bounded context (domain/integration/contract/e2e).  
2) **Contract definitions** (OpenAPI/Proto + event schemas) và tests bảo vệ.  
3) **Idempotency test suite** cho side effects quan trọng.  
4) **Outbox/inbox test suite** (nếu dùng).  
5) **Smoke scenarios** cho 1–2 workflow critical + mapping tới SLO/alerts.

---

## 13) Exercise (có hướng dẫn) — Viết test plan cho “BatchAccepted → Wallet credit → Export job”

### Bài toán
Trong ADLP, khi Quality accept batch (`BatchAccepted`), hệ thống sẽ:
- Wallet credit earning cho labeler,
- Export tạo export job cho dataset version.

Bạn cần thiết kế test plan đủ để đảm bảo:
- không double credit/double export,
- contract event không phá consumer,
- failure sẽ đi vào DLQ và có owner xử lý.

### Cách làm (gợi ý từng bước)

**Bước 1 — Liệt kê rủi ro (3 dòng)**  
Gợi ý:
- duplicate events (at-least-once),
- out-of-order,
- publish lost (outbox/publisher fail),
- consumer fail (transient/permanent).

**Bước 2 — Chọn loại test cho từng rủi ro**  
Mapping thực dụng:
- duplicate → consumer idempotency integration test (“consume twice”)
- contract break → event contract tests (schema + version)
- publish lost → outbox integration test (write+retry)
- consumer fail → DLQ smoke test + runbook check

**Bước 3 — Viết 5 test cases (dạng câu chuyện)**
Ví dụ format:
- “credits once when same BatchAccepted delivered twice”
- “routes to DLQ after max retries for invalid payload”

**Bước 4 — Chọn metrics/alerts liên quan (liên kết Chương 26)**
Gợi ý:
- consumer lag,
- DLQ size,
- credit success/fail count,
- export job created count.

### Output mong đợi
1) Danh sách 5 test cases + loại test (unit/integration/contract/smoke).  
2) Một mô tả idempotency key strategy (batch_id + decision_version).  
3) Một checklist runbook 1 trang: “DLQ BatchAccepted tăng đột biến”.

