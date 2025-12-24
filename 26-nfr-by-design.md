# Chương 26 — NFR by Design: biến “phi chức năng” thành tiêu chí thiết kế, không phải checklist cuối dự án

Bạn có thể thiết kế domain model rất đẹp, boundaries rất đúng… rồi vẫn thất bại vì một lý do “không ai nghĩ nó là domain”: hệ thống chậm, không đo được, không phục hồi được, không audit được, hoặc “đúng nghiệp vụ” nhưng vi phạm compliance.

Trong ADLP, những rủi ro này không phải lý thuyết:
- Premium 48h là lời hứa kinh doanh. Nếu queue bị backlog 8 tiếng vì prelabeling chậm và không có cảnh báo sớm, bạn sẽ **vi phạm cam kết** dù domain flow đúng.
- Payout phải đúng 100%. Nếu idempotency sai hoặc audit trail thiếu, bạn không chỉ “bug” — bạn **mất tiền và mất niềm tin**.
- Dữ liệu audio/transcript liên quan privacy. Nếu retention không rõ hoặc access control lỏng, bạn **dính rủi ro pháp lý**.

Chương này dạy một thói quen: **thiết kế NFR như một phần của domain & bounded contexts**, không phải “phiên họp performance cuối sprint”.

Nguồn tham chiếu chính cho ADLP: `design/docs/2.StrategicDesign/DDD_STRATEGIC_DESIGN_V0.2.md` (đặc biệt ADR-007 về SLO theo context).

---

## Bạn sẽ nhận được gì sau chương này?

1) Một cách nghĩ “NFR by design”: biến NFR thành driver cho kiến trúc, integration và data strategy.  
2) Khung chốt SLO theo Bounded Context (theo ADR-007) và cách dùng SLO để ra quyết định.  
3) Kỹ thuật/chiến thuật thiết kế thực dụng cho 6 nhóm NFR: scalability, observability, security, compliance, performance, resilience.  
4) Best practices + trade-offs + anti-patterns (những lỗi hay gây “toang trong production”).  
5) Áp dụng vào ADLP: SLO tiering, event processing SLO, retention/DR, và “premium 48h” như một kịch bản kiểm thử NFR.  
6) Exercise có hướng dẫn: tự chốt NFR pack cho một workflow end-to-end và biến nó thành checklist triển khai/đo đạc.

---

## 1) “NFR” thật ra là cái gì? (và vì sao nó luôn quay lại cắn bạn)

Functional Requirements trả lời: “hệ thống làm được việc X không?”  
Non-Functional Requirements (NFR) trả lời: “làm việc X **với chất lượng** như thế nào, trong điều kiện nào, với rủi ro nào?”

Nếu FR là “đường đi”, NFR là “luật giao thông + chất lượng mặt đường + thời tiết”. Bạn không thể vẽ bản đồ mà bỏ qua chúng.

> **NOTE**  
> NFR không “phi” chức năng. Nó là “điều kiện để chức năng có giá trị trong thực tế”.

### 1.1 NFR không phải là “mục tiêu chung chung”

Các câu kiểu:
- “Hệ thống phải nhanh”
- “Hệ thống phải ổn định”
- “Hệ thống phải bảo mật”

…không giúp bạn thiết kế. NFR phải được diễn đạt như **kịch bản (scenario)**: có stimulus, bối cảnh, thước đo.

> **EXAMPLE**  
> “Khi có 2.000 labelers hoạt động đồng thời (stimulus), Labeling API p95 < 500ms và error rate < 0.2% trong giờ cao điểm (measure), nếu breach thì degrade có kiểm soát: ưu tiên autosave và read-only mode cho attachments (response).”

### 1.2 NFR theo context, không phải “một SLO cho cả hệ thống”

DDD v0.2 đã chốt một quyết định rất đúng: **SLO theo Bounded Context** (ADR-007). Tại sao?
- “Login chậm” và “Export chậm” có business impact khác nhau.
- “Payout sai” nguy hiểm hơn “profile search chậm”.
- Một SLO chung sẽ kéo bạn vào thiết kế sai: hoặc over-engineer, hoặc under-protect.

---

## 2) NFR by Design: quy trình thực dụng (làm trong 60–90 phút)

Hãy coi đây là “mini-workshop” bắt buộc sau Strategic/Tactical và trước khi chốt kiến trúc/integration chi tiết.

### Bước 1 — Chọn 1–2 kịch bản “đắt tiền nhất”

Trong ADLP, kịch bản kinh điển là **Premium 48h**:
1) Ingestion nhận audio
2) Prelabeling chạy model
3) Assignment phân batch cho labelers
4) Labeling edit + submit
5) Quality evaluate/review
6) Export dataset + Payout

Nếu workflow này vỡ (chậm/không đo được/không recover), business vỡ.

### Bước 2 — Chốt SLO tier theo context (dùng ADR-007 như baseline)

Bạn không cần phát minh lại. ADR-007 đã có tiering (Critical/High/Standard/Low) và ví dụ metric.

Điều bạn cần làm là:
- xác nhận tier bằng business (đâu là critical path),
- chọn metric “đại diện” (latency, availability, throughput, error rate, correctness),
- ghi rõ **consequence of breach** (đây là phần khiến team nghiêm túc).

> **WARNING**  
> Nếu bạn không ghi “consequence of breach”, SLO sẽ trở thành giấy trang trí. Khi production cháy, không ai biết ưu tiên xử lý cái gì.

### Bước 3 — Tạo “NFR tactics” cho từng context

Với mỗi context, chọn 3–5 tactics thật cụ thể. Ví dụ:
- Prelabeling (async): scale theo queue depth, DLQ policy, backpressure, circuit breaker tới model endpoint.
- Wallet (financial): idempotency + audit trail 100% + outbox + reconciliation job.
- Identity: rate limit + lockout policy + token refresh p95.

### Bước 4 — Biến NFR thành artefacts và acceptance criteria

NFR phải “đi được vào backlog”:
- Dashboard/alerts cụ thể (observability)
- Load test plan hoặc replay plan (performance)
- Threat model + security tests (security)
- Retention matrix + deletion workflow (compliance)
- DR test cadence (resilience)

---

## 3) Scalability: scale cái gì, theo tín hiệu nào, và scale đến đâu?

Scalability không chỉ là “auto-scaling”. Nó là năng lực hệ thống **tăng tải mà không đổi cách vận hành** (hoặc đổi rất ít).

### 3.1 Scale theo “đơn vị đúng” của domain

Trong ADLP, “đơn vị công việc” thường là:
- dataset
- batch
- segment
- labeling session
- quality evaluation

Bạn muốn scale theo **segment/batch throughput**, không theo “CPU chung chung”.

> **EXAMPLE**  
> Prelabeling SLO: throughput > 10 segments/s, queue depth < 1000.  
> Đây là tín hiệu scale rất domain-specific: queue depth chính là “nợ công việc”.

### 3.2 Trade-off: scale read vs scale write

Ở nhiều hệ thống, read tăng nhanh hơn write. DDD giúp bạn tách:
- command path (write, invariants, correctness)
- query path (read models, cache, denormalization)

Trong ADLP:
- Labeling UI đọc rất nhiều (segments, audio links, autosave state) → read model + cache là hợp lý.
- Wallet write phải cực đúng → scale write bằng sharding/partitioning phải đi kèm audit/reconciliation.

> **BEST PRACTICE**  
> Hãy viết SLO theo “throughput domain” (segments/s, batches/min, exports/day) thay vì chỉ ghi RPS.

### 3.3 Anti-pattern: “scale bằng cách tăng timeout”

Khi hệ thống chậm, nhiều team làm:
- tăng timeout client,
- tăng retry,
- tăng size instance,

…và vô tình tạo “thác lũ” (retry storm) làm mọi thứ tệ hơn.

> **WARNING**  
> Timeout là “hợp đồng”: tăng timeout làm tăng concurrency bị giữ lại → hệ thống chết nhanh hơn khi có spike.

---

## 4) Observability: nếu không đo được, bạn không có hệ thống (chỉ có niềm tin)

Observability không phải “có log”. Nó là khả năng trả lời 3 câu khi production có vấn đề:
1) Có vấn đề gì? (symptom)
2) Vấn đề nằm ở đâu? (localization)
3) Vì sao xảy ra? (root cause)

### 4.1 Ba trụ: logs, metrics, traces (và correlation)

Trong hệ thống DDD với nhiều contexts, correlation là bắt buộc:
- `correlation_id`: đi xuyên toàn workflow
- `causation_id`: event nào gây ra hành động này

Nếu thiếu correlation:
- bạn không thể debug “premium 48h” end-to-end,
- bạn không thể chứng minh SLA/SLO,
- bạn không thể làm postmortem đúng.

> **BEST PRACTICE**  
> Thiết kế envelope event và request headers ngay từ đầu để có `correlation_id`/`causation_id`. Đừng “để sau”.

### 4.2 Observability theo SLO

SLO không chỉ là con số; nó quyết định dashboard và alert.

Trong ADR-007, mỗi context có metric và cách đo. Hãy biến nó thành:
- golden signals (latency, traffic, errors, saturation),
- SLI definition (đo ở đâu),
- alerting policy (khi nào gọi người).

> **EXAMPLE**  
> Task Assignment (Critical): availability 99.95%, p95 < 200ms.  
> Alert không nên chỉ là “CPU > 80%”. Nó phải là “p95 breach + error budget burn rate”.

### 4.3 Anti-pattern: log như “truyện dài”

Log quá nhiều nhưng không cấu trúc:
- không có trace_id,
- không có domain identifiers (batch_id, dataset_id),
- không có event version,

…thì khi cần debug bạn vẫn mù.

---

## 5) Security: “generic domain” nhưng ảnh hưởng mọi quyết định

Trong DDD v0.2, Identity & Access là generic domain nhưng critical (SLO 99.95%). Điều đó nói một điều: generic domain không có nghĩa là “làm qua loa”.

### 5.1 Threat model theo workflow

Hãy threat-model theo kịch bản:
- Labeler truy cập audio → signed URL, expiry, scope
- Admin export dataset → permission boundary, audit
- Payment provider → ACL, secrets management

### 5.2 Trade-off: tốc độ phát triển vs bề mặt tấn công

Ví dụ ADLP:
- Nếu để Labeling UI gọi trực tiếp S3 public URL → nhanh, nhưng lộ data.
- Nếu dùng signed URL + short TTL + policy → phức tạp hơn, nhưng giảm rủi ro.

> **BEST PRACTICE**  
> Quy tắc “least privilege” phải gắn với bounded contexts: context nào được làm gì, gọi được ai, đọc được data nào.

### 5.3 Anti-pattern: “auth ở gateway là đủ”

Gateway auth không thay thế được:
- authorization theo domain rules,
- audit trail theo context,
- protection ở data plane (S3/object store).

---

## 6) Compliance & Privacy: retention, audit, và quyền xoá dữ liệu

Compliance không chỉ cho enterprise. Với dữ liệu audio/transcript, “không có retention policy” là một khoản nợ pháp lý.

DDD v0.2 đã có bảng retention & lifecycle; đây là một best practice hiếm thấy ở tài liệu thiết kế: nó biến compliance thành artefact.

### 6.1 Retention là một phần của domain

Nếu business nói “raw audio chỉ giữ 2 năm”, thì domain cần:
- policy rõ,
- job dọn dữ liệu,
- audit chứng minh đã xóa,
- xử lý liên đới (derived artifacts).

> **EXAMPLE**  
> Audit logs: 90 ngày hot (CloudWatch), 7 năm cold (S3 Glacier).  
> Đây là quyết định kiến trúc + chi phí, không phải câu “chúng ta sẽ log”.

### 6.2 Anti-pattern: “xóa là DROP TABLE”

Data deletion trong hệ thống phân tán cần:
- traceability,
- đảm bảo không còn bản sao ở read models/cache,
- xử lý backup/restore và legal hold.

---

## 7) Performance: tối ưu đúng chỗ, đúng thời điểm

Performance trong DDD không nên bắt đầu bằng “tối ưu SQL”. Nó bắt đầu bằng:
- chọn boundaries đúng để giảm coupling,
- chọn read models đúng để tránh join xuyên context,
- chọn cache đúng để giảm load read.

### 7.1 Performance theo critical path

Trong ADLP, critical path khác nhau theo persona:
- Labeler: Assignment + Labeling latency
- Ops/QA: Quality processing time
- Customer: Export latency

Tối ưu cái không ai dùng chỉ tạo nợ.

### 7.2 Trade-off: consistency vs latency

Bạn sẽ thường phải chọn:
- sync (stronger consistency, lower latency for user) vs async (looser consistency, higher throughput),
- hoặc read model eventual consistency để UI nhanh.

> **NOTE**  
> Hãy nói rõ “độ trễ chấp nhận được” trong ngôn ngữ business: “assignment có thể lệch 2 giây” khác hoàn toàn “payout có thể lệch 2 giây”.

---

## 8) Resilience: thiết kế để hỏng (và vẫn sống)

Resilience là năng lực:
- chịu lỗi (fault tolerance),
- phục hồi (recovery),
- và giới hạn blast radius.

DDD giúp bạn có “nơi để đặt bulkhead”: bounded context/deployment unit chính là bulkhead tự nhiên.

### 8.1 Retry/DLQ là một phần của correctness

ADR-007 có event processing SLO và retry/DLQ thresholds. Đây không phải “ops detail”; nó là thiết kế workflow.

Ví dụ:
- `EarningCredited`: 10 retries, DLQ và manual intervention → vì sai là mất tiền.
- `QualityEvaluated`: latency < 5min, 5 retries → vì backlog ảnh hưởng payout.

### 8.2 DR: RPO/RTO theo context

Với DR, câu hỏi đúng là: “context này mất dữ liệu tối đa bao lâu thì chấp nhận được (RPO)? ngưng dịch vụ tối đa bao lâu (RTO)?”

Wallet có RPO < 1 minute vì financial integrity. Prelabeling có thể replay từ Kafka.

> **BEST PRACTICE**  
> DR plan phải có “cách test”: diễn tập restore theo cadence (ví dụ mỗi quý) và có tiêu chí pass/fail.

### 8.3 Anti-pattern: “cứ thêm retry”

Retry không có:
- idempotency,
- backoff + jitter,
- circuit breaker,
- rate limit,

…sẽ biến một lỗi nhỏ thành outage lớn.

---

## 9) Artefacts/Deliverables (đầu ra tối thiểu)

Sau chương này, với mỗi dự án/bounded context, bạn nên có:
1) **SLO table theo context** (baseline ADR-007 + chỉnh theo business).  
2) **Event processing SLO** (max latency + retry + DLQ + owner).  
3) **Retention & lifecycle matrix** (data types, hot/cold, deletion policy).  
4) **Observability pack**: dashboard + alerts + tracing/correlation conventions.  
5) **Security pack**: threat model theo workflow + access matrix theo context.  
6) **DR pack**: RPO/RTO + backup strategy + test cadence.

> **CHECKLIST**  
> - [ ] Mỗi context có tier + 3–5 SLO metrics đo được  
> - [ ] Mỗi workflow quan trọng có `correlation_id` end-to-end  
> - [ ] Retry policy đi kèm idempotency + DLQ + owner rõ ràng  
> - [ ] Có retention policy cho dữ liệu nhạy cảm + audit cách xóa  
> - [ ] Có RPO/RTO + diễn tập restore định kỳ

---

## 10) Exercise (có hướng dẫn) — Thiết kế “NFR pack” cho Premium 48h

### Bài toán
Bạn là architect cho ADLP. Business cam kết: “Premium dataset được hoàn tất trong 48 giờ”. Bạn cần biến cam kết này thành NFR thực dụng để team có thể implement và vận hành.

### Cách làm (gợi ý từng bước)

**Bước 1 — Vẽ timeline và xác định “điểm chết”**  
Liệt kê các chặng: Ingestion → Prelabeling → Assignment → Labeling → Quality → Export/Payout.  
Chọn 2 chặng nếu chậm sẽ khiến toàn cam kết fail (thường là Prelabeling backlog + Quality backlog).

**Bước 2 — Chọn 4–6 metrics thật sự quyết định 48h**  
Ví dụ (bạn có thể chọn khác):
- queue depth/lag của Prelabeling
- throughput segments/s
- số batch đang “stuck” ở Assignment lock
- time-to-review trung bình của Quality
- end-to-end lead time của Premium
- error budget burn rate của Assignment/Labeling

**Bước 3 — Chốt SLO theo context**  
Gợi ý dùng ADR-007 làm baseline, rồi thêm SLO đặc thù Premium:
- Prelabeling: queue depth < X, latency p95 < 10s, throughput > Y
- Task Assignment: p95 < 200ms, lock expiry 100%
- Quality: processing time < 5min/batch, review SLA < 4h

**Bước 4 — Chọn tactics để đạt SLO**  
Gợi ý:
- Auto-scale workers theo queue depth (có cooldown)
- Backpressure: nếu Quality backlog cao, giảm intake Premium mới hoặc tăng reviewer capacity
- Circuit breaker với model endpoint + fallback (mark “needs manual prelabel”)
- DLQ + runbook cho sự kiện quan trọng (BatchSubmitted, QualityEvaluated, EarningCredited)

**Bước 5 — Định nghĩa “Definition of Done” cho NFR**  
Ví dụ:
- Có dashboard Premium lead time, alert khi > 36h (để còn 12h xử lý)
- Có trace end-to-end cho 1 premium batch (correlation id)
- Có load test tối thiểu cho Assignment API ở peak
- Có DR test cho Wallet theo RPO/RTO

### Output mong đợi (nộp bài)
1) Một bảng SLO rút gọn cho 4 contexts (Prelabeling, Assignment, Quality, Export/Wallet).  
2) Một danh sách 5 alerts và 3 dashboards.  
3) Một policy retry/DLQ cho 3 event types quan trọng.  
4) Một runbook 1 trang: “Premium backlog vượt ngưỡng” (ai làm gì trước).

