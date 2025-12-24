# Chương 29 — Tổ chức code theo DDD: Layered/Hexagonal, luật phụ thuộc, và cách tránh “DDD nửa mùa”

Bạn có thể làm đúng mọi thứ đến chương 28 (use case, domain model, events), rồi vẫn thất bại vì một lý do rất đời: **codebase không “giữ hình” được**. Sau 3–6 tháng:
- domain logic nằm rải rác trong controller, job, consumer, cron, UI;
- dependency đi ngược: domain import ORM, domain gọi HTTP;
- “service” thành God object, domain model thành DTO;
- mọi thứ phụ thuộc framework → refactor là một dự án.

Nói thẳng: DDD trong production sống nhờ hai thứ:
1) **Ranh giới ở level kiến trúc** (tầng, module, context).
2) **Luật phụ thuộc (dependency rules)** được giữ kỷ luật qua thời gian.

Chương này giúp bạn biến DDD thành “bộ khung” của codebase, để team có thể mở rộng mà không phá domain.

Ví dụ xuyên suốt: ADLP theo `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md`.

---

## Bạn sẽ nhận được gì sau chương này?

1) Một cách tổ chức code theo DDD dễ áp dụng (layered hoặc hexagonal) và khi nào chọn cái nào.  
2) Luật phụ thuộc “bất khả xâm phạm” để domain không bị kéo xuống hạ tầng.  
3) Heuristics tổ chức module theo bounded context (monorepo, mono-service, microservices).  
4) Best practices + trade-offs + anti-patterns (đặc biệt: anemic model, transaction rải rác, domain phụ thuộc framework).  
5) Áp dụng vào ADLP: đề xuất cấu trúc module cho 1 bounded context (Task Assignment hoặc Quality).  
6) Exercise có hướng dẫn: tự “chẩn đoán” một codebase và viết plan refactor 2 giờ.

---

## 1) Vấn đề thật sự của “tổ chức code” không nằm ở folder — nó nằm ở luật phụ thuộc

Bạn có thể đặt folder `domain/` nhưng vẫn sai, nếu:
- domain import `sqlalchemy`, `typeorm`, `requests`, `kafka`;
- domain throw HTTP error directly;
- domain method gọi repository/DB (tức domain biết persistence).

Folder chỉ là nhãn. Thứ bảo vệ DDD là **dependency direction**: domain phải là lõi, các thứ khác “bao quanh”.

> **NOTE**  
> Một test đơn giản: “Bạn có thể chạy unit test domain mà không cần DB, không cần network không?” Nếu không, domain đã bị kéo xuống infrastructure.

---

## 2) Layered Architecture: đơn giản, dễ bắt đầu, đủ tốt cho đa số đội

Layered architecture trong DDD thường có 4 tầng:
- **Domain**: entities, value objects, aggregates, domain services, domain events, domain errors.
- **Application**: use cases (command/query handlers), orchestrations trong một bounded context, ports/interfaces.
- **Infrastructure**: repositories implementation, ORM mapping, message broker adapter, external clients, caches.
- **Interfaces**: HTTP/gRPC controllers, CLI, consumer entrypoints, DTO mapping, auth middleware.

Điểm cốt lõi: **tầng trong không phụ thuộc tầng ngoài**.

### 2.1 Một ví dụ “đúng” cho ADLP (một bounded context)

Giả sử bạn triển khai Task Assignment Context.

- `assignment/domain/`
  - `Batch` (aggregate root), `Assignment` (entity), `Lock` (VO), `Priority` (VO)
  - domain events: `BatchAssigned`, `BatchLockExpired`
  - domain errors: `LockConflict`, `NotAssignable`
- `assignment/application/`
  - `ClaimBatchHandler`, `ReleaseBatchHandler`, `RecommendBatchQueryHandler`
  - ports: `BatchRepository`, `Clock`, `CorrelationIdProvider`
- `assignment/infrastructure/`
  - `PostgresBatchRepository`
  - `KafkaPublisher` (outbox publisher)
  - `RedisReadModel` (nếu có)
- `assignment/interfaces/`
  - HTTP controllers (map request → command → handler)
  - consumers (map event → handler)

Tầng application nói với infrastructure thông qua interface/port.

### 2.2 Khi layered đủ và khi layered bắt đầu “đau”

Layered đủ khi:
- team nhỏ–vừa,
- bạn muốn ship nhanh,
- bạn có kỷ luật để không “bóp méo” tầng.

Layered bắt đầu đau khi:
- dependency injection phức tạp,
- bạn có quá nhiều adapter và muốn “đảo chiều” từ ngoài vào,
- hoặc team cứ vô tình “đi tắt” vì tiện (domain import ORM).

Khi đó, hexagonal/clean giúp bạn “bắt buộc” dependency direction rõ hơn.

---

## 3) Hexagonal/Clean Architecture: làm rõ ports/adapters, chống “đi tắt”

Hexagonal (Ports & Adapters) nói một câu rất mạnh:
> “Core (domain + application) chỉ biết các port. Thế giới bên ngoài là adapter cắm vào port.”

Thực tế, nhiều đội “làm layered nhưng gọi tên hexagonal”. Không sao. Quan trọng là:
- bạn có **ports** ở application,
- bạn có **adapters** ở infrastructure/interfaces,
- và core không import framework.

### 3.1 Ports là gì? (trong đời thực)

Port là interface mô tả “core cần gì từ bên ngoài”, ví dụ:
- `BatchRepository` (lưu/đọc aggregate)
- `EventBus` (publish integration events)
- `AudioStorage` (signed URL)
- `Clock` (thời gian — để test deterministic)

Nếu bạn không có ports, application layer sẽ import thẳng DB/HTTP và bạn mất khả năng test/refactor.

> **BEST PRACTICE**  
> Những thứ “đầu độc test” (time, random, network, DB) nên đi qua port để fake/mocking dễ và core không bị kéo xuống infra.

---

## 4) Luật phụ thuộc (Dependency Rules) — 6 luật bạn nên khắc lên tường

1) `domain` không import `application`, `infrastructure`, `interfaces`.  
2) `application` có thể import `domain`, nhưng không import `infrastructure` (chỉ biết port/interface).  
3) `infrastructure` implement ports của `application` và có thể import framework.  
4) `interfaces` chỉ mapping và gọi `application`, không chứa invariants.  
5) Tất cả “side effects” (DB/network/broker) nằm ở `infrastructure`.  
6) Domain errors là ngôn ngữ chính; mapping sang HTTP codes chỉ nằm ở `interfaces`.

Nếu bạn giữ 6 luật này, codebase sẽ “giữ hình” lâu hơn mọi refactor.

> **WARNING**  
> Luật phụ thuộc không được giữ bằng niềm tin. Nó cần cơ chế: review checklist, lint rules, hoặc module boundaries test.

---

## 5) Tổ chức theo Bounded Context: module boundary quan trọng hơn package boundary

Từ Strategic Design, bạn đã có bounded contexts. Tổ chức code nên phản ánh điều đó:
- mỗi context là một module có lifecycle riêng,
- có “published contracts” rõ (API, events),
- hạn chế shared code.

### 5.1 Shared code: cái gì nên share, cái gì không?

Nên share:
- primitives kỹ thuật (logging wrapper, config loader),
- cross-cutting infra helpers (tracing propagation, error envelope utilities),
- contract schemas (published language) nếu bạn có governance rõ.

Không nên share:
- domain objects giữa contexts (đặc biệt aggregate/entities),
- business rules,
- repositories/query logic.

> **NOTE**  
> “Shared Kernel” là một pattern có cái giá rất đắt. Nếu bạn share domain code giữa contexts, bạn phải chấp nhận coupling và đồng bộ release.

### 5.2 Monorepo vs multi-repo không quan trọng bằng “ownership clarity”

Bạn có thể:
- monorepo nhưng module boundaries cứng (tốt),
- hoặc multi-repo nhưng team vẫn copy/paste và phụ thuộc ngầm (tệ).

DDD cần:
- owner rõ,
- contract rõ,
- và kiểm soát phụ thuộc rõ.

---

## 6) Best practices (kèm giải thích)

### 6.1 Đặt “domain purity” làm ưu tiên số 1 cho core contexts

Trong ADLP, Task Assignment và Quality là core. Ở những context này:
- domain methods phải express invariants,
- unit tests phải dày,
- domain không phụ thuộc ORM.

Generic contexts (Identity, Profile) có thể tactical-lite hơn, nhưng vẫn nên giữ dependency direction.

### 6.2 Use Case handlers là “điểm vào” duy nhất để thay đổi state

Nếu bạn thấy code thay đổi domain state ở:
- controller,
- consumer handler trực tiếp,
- cron job,

…hãy refactor để chúng gọi use case handlers, không bypass application layer.

Vì sao? Vì:
- transaction boundary,
- idempotency,
- and publish policy

…đều nằm ở use case.

### 6.3 Tách query models khỏi domain models khi cần (CQRS-lite)

Đừng bắt domain model phục vụ mọi query. Domain model phục vụ invariants.  
Nếu UI cần view model tổng hợp, hãy tạo read model ở application/infrastructure (tùy).

### 6.4 Đưa “time” và “policy” thành dependency

Nhiều rule phụ thuộc policy version và thời gian (TTL, deadlines). Nếu bạn hardcode:
- test sẽ flaky,
- change policy sẽ đau.

Hãy:
- đưa `Clock` thành port,
- đưa policy config có version,
- log policy_version khi quyết định.

### 6.5 Viết “module boundary tests” hoặc lint rules sớm

Bạn không cần hoàn hảo, nhưng cần một cách tự động nhắc team khi ai đó làm `domain import orm`.

---

## 7) Trade-offs: cấu trúc càng “sạch” càng tốn chi phí ban đầu

### 7.1 Bạn sẽ viết thêm glue code

Ports/adapters, mapping DTO, error translation… là chi phí thật.

Nhưng nếu domain của bạn “đắt tiền” (ADLP core contexts), glue code thường rẻ hơn:
- bug concurrency,
- payout sai,
- workflow vỡ âm thầm,
- refactor đau.

### 7.2 Over-architecture là có thật

Nếu bạn có một CRUD context rất đơn giản, cố áp full hexagonal sẽ làm team chậm lại.  
DDD thực dụng cho phép “tactical-lite” ở generic domains, miễn là không phá boundary.

> **BEST PRACTICE**  
> Đầu tư kiến trúc theo “core domain gradient”: core sạch và test dày; supporting vừa đủ; generic tối giản nhưng không đi ngược luật phụ thuộc.

---

## 8) Anti-patterns (triệu chứng → hậu quả → cách tránh)

### 8.1 Anemic Domain Model (“DDD nửa mùa”)

**Triệu chứng:** entities chỉ có getter/setter; mọi rule nằm trong service.  
**Hậu quả:** rule rò rỉ, duplication, test khó, invariants không ai bảo vệ.  
**Cách tránh:** đưa rule vào aggregate/VO; handlers chỉ điều phối (như Chương 28).

### 8.2 Domain phụ thuộc framework/ORM

**Triệu chứng:** domain annotations ORM khắp nơi; domain gọi repository; domain chứa SQL.  
**Hậu quả:** domain không test được; refactor khó; lock-in công nghệ.  
**Cách tránh:** mapping ở infrastructure; domain thuần; repository interfaces ở application/domain boundary.

### 8.3 Transaction boundary rải rác

**Triệu chứng:** commit ở nhiều nơi; gọi HTTP/broker trong transaction; side effect không kiểm soát.  
**Hậu quả:** inconsistent state, deadlocks, bug khó tái hiện.  
**Cách tránh:** UoW trong handler; publish-after-commit hoặc outbox.

### 8.4 “Shared everything”

**Triệu chứng:** `common/` chứa domain objects dùng chung; contexts import lẫn nhau.  
**Hậu quả:** coupling, release coordination, đổi một nơi vỡ nhiều nơi.  
**Cách tránh:** share contract/published language, không share domain; dùng ACL khi cần (ADR-006).

### 8.5 “Directory-driven design”

**Triệu chứng:** đổi folder liên tục, nhưng luật phụ thuộc không có.  
**Hậu quả:** cảm giác “đã làm DDD” nhưng domain vẫn bị phá.  
**Cách tránh:** chốt dependency rules + kiểm soát tự động.

---

## 9) Áp dụng vào ADLP: cấu trúc mẫu cho một context (Quality Assurance)

Một cấu trúc module thực dụng (ví dụ ngôn ngữ bất kỳ):

```text
quality/
  domain/
    aggregates/
      QualityEvaluation
      ReviewDecision
    value_objects/
      QualityScore
      PolicyVersion
    events/
      BatchEvaluated
      BatchAccepted
    errors/
      AlreadyDecided
      PolicyMismatch
  application/
    commands/
      EvaluateBatch
      AcceptBatch
    handlers/
      EvaluateBatchHandler
      AcceptBatchHandler
    ports/
      QualityRepository
      EventPublisher
      Clock
  infrastructure/
    persistence/
      PostgresQualityRepository
      orm_mapping/...
    messaging/
      OutboxPublisher
      KafkaConsumerAdapters
  interfaces/
    http/
      controllers/...
      dto/...
    consumers/
      on_BatchSubmitted/...
```

Nhìn vào cây này, một dev mới có thể đoán:
- domain rules ở đâu,
- use cases ở đâu,
- side effects ở đâu,
- và integration entrypoints ở đâu.

> **NOTE**  
> “Tách theo feature” và “tách theo layer” không mâu thuẫn. Bạn hoàn toàn có thể tách theo bounded context (feature) và bên trong vẫn có layers.

---

## 10) Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] `domain` không import framework/ORM/HTTP/broker  
> - [ ] `application` chỉ biết ports, không biết DB cụ thể  
> - [ ] Controllers/consumers không chứa invariants (chỉ map + gọi handler)  
> - [ ] Một use case đi qua một “spine” nhất quán (Chương 28)  
> - [ ] Có quy tắc/kiểm tra tự động cho module boundaries  
> - [ ] Shared code không chứa domain objects giữa contexts  
> - [ ] Core contexts có unit tests dày cho domain invariants  

---

## 11) Artefacts/Deliverables

Sau chương này, bạn nên tạo được:
1) **Architecture skeleton** cho từng bounded context (folders/modules + dependency rules).  
2) **Dependency rules document** (6 luật ở mục 4 + cách enforce).  
3) **Refactoring guide**: “nếu logic rò rỉ ra controller thì làm gì”.  
4) **Shared code policy**: cái gì được share, cái gì cấm share.  

---

## 12) Exercise (có hướng dẫn) — “Chẩn đoán DDD nửa mùa” trong 30 phút

### Bài toán
Giả sử bạn nhận một codebase (có thể là ADLP phiên bản MVP) và cảm giác “domain không nằm ở đâu cả”. Bạn cần chẩn đoán nhanh và đưa ra plan refactor nhỏ (2 giờ) để đưa hệ thống về đúng hướng.

### Cách làm (gợi ý từng bước)

**Bước 1 — Tìm 1 use case đắt tiền**  
Chọn một use case dễ gây tranh chấp: `AcceptBatch` hoặc `SubmitTranscript`.

**Bước 2 — Trace đường đi của rule**  
Tìm xem invariant nằm ở đâu:
- controller?
- service?
- SQL?
- UI?

Ghi lại “điểm rò rỉ” (leak points).

**Bước 3 — Kiểm tra dependency direction**  
Tìm các import ngược:
- domain import ORM
- domain gọi HTTP
- handler import repository implementation

**Bước 4 — Viết plan refactor 2 giờ (rất nhỏ nhưng đúng hướng)**  
Gợi ý plan:
1) Tạo `domain/` với aggregate + domain errors cho use case đó  
2) Tạo `application/` handler giữ transaction boundary  
3) Controller gọi handler (mỏng lại)  
4) Infrastructure implement repository port  
5) Thêm 2 unit tests cho invariants chính  

**Bước 5 — Tự kiểm**  
Sau refactor, trả lời:
- domain có test chạy không cần DB không?
- controller còn biết invariants không?
- publish policy có rõ không?

### Output mong đợi
1) Danh sách 5 leak points (folder/file mô tả).  
2) Một plan refactor 2 giờ (5–7 bước).  
3) Hai invariants được đưa vào domain method + unit tests tương ứng.

