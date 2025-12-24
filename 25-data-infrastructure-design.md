# Chương 25 — Data & Infrastructure Design: Database per Service, Read Models, Event Store, Caching & Performance

DDD nói nhiều về model và boundaries, nhưng nếu data strategy sai, bạn sẽ phá boundaries bằng đường tắt: join xuyên context, share schema, hoặc “gọi thẳng DB cho nhanh”. Chương này tập trung vào phần hay gây tranh cãi nhất khi triển khai DDD: **dữ liệu và hạ tầng**.

Chúng ta sẽ trả lời các câu hỏi thực dụng:
- “Database per service” có ý nghĩa gì, và khi nào áp dụng?
- Nếu không join xuyên context, đọc dữ liệu kiểu gì? (read models/CQRS-lite)
- Event store có cần không? audit trail làm sao?
- Cache đặt ở đâu để không phá invariants?
- Performance tối ưu kiểu gì mà không phá domain boundaries?

Ví dụ xuyên suốt: ADLP (MVP audio STT), nơi dữ liệu trải trên S3 + Postgres + queues, với quality/review/audit requirements.

---

## Bạn sẽ nhận được gì sau chương này?

1) Hiểu “database per service” như một quyết định strategic để bảo vệ BC boundaries.  
2) Biết cách dùng read models để tránh join xuyên context.  
3) Biết khi nào cần event store và khi nào audit log + versioning là đủ.  
4) Cách dùng cache đúng: cache reads, không phá invariants.  
5) Áp dụng vào ADLP: data ownership cho Ingestion/Prelabeling/Assignment/Labeling/Quality/Export.  
6) Anti-patterns và exercise có hướng dẫn.

---

## 1) Database per Service: mục tiêu là boundary, không phải “mỗi service một DB cho vui”

“Database per service” thường bị hiểu sai thành “mỗi service phải có một Postgres instance riêng”. Thực tế, mục tiêu là:
- **schema ownership**: context nào sở hữu sự thật của dữ liệu đó,
- **evolution độc lập**: schema change không phá context khác,
- **tránh coupling qua DB** (join xuyên context).

Bạn có thể có cùng một Postgres cluster nhưng schema tách và access control tách. Điều quan trọng là boundary và quyền.

Trong ADLP:
- Quality context sở hữu quality scores, reviews, decisions.
- Labeling context sở hữu transcript edits, autosave state, submission metadata.
- Export context sở hữu dataset versions và export jobs.

Nếu Export đọc thẳng transcript tables của Labeling bằng join, bạn đã phá BC boundary.

### 1.1 Code Example: Separate Schemas

Dù dùng chung 1 Postgres server, hãy tách schema logical:

```sql
-- Context: Labeling
CREATE SCHEMA labeling;
CREATE TABLE labeling.batches (
    batch_id UUID PRIMARY KEY,
    status VARCHAR(50), -- Assigned/Submitted
    assignment_json JSONB
);

-- Context: Quality (Separate ownership)
CREATE SCHEMA quality;
CREATE TABLE quality.evaluations (
    evaluation_id UUID PRIMARY KEY,
    batch_ref_id UUID NOT NULL, -- Logical Ref only
    score DECIMAL(5,2),
    decision VARCHAR(50) -- Accepted/Rejected
);
-- NO FOREIGN KEY from quality.batch_ref_id to labeling.batches!
```

**Quy tắc vận hành thực dụng**
- Không tạo FK xuyên schema; dùng logical reference + event/API.  
- Quyền DB theo context: chỉ context owner mới có quyền write.  
- Nếu cần query tổng hợp, hãy đi qua read model (CQRS-lite).


---

## 2) Read Models (CQRS-lite): cách đọc dữ liệu mà không xuyên tường

Khi tách boundaries, bạn sẽ gặp câu hỏi: “Vậy list màn hình dashboard lấy data từ đâu?”

CQRS-lite giải quyết bằng cách:
- writes theo domain model trong mỗi context,
- reads dùng read model/projection được build từ events hoặc từ ETL nhẹ.

### Ví dụ ADLP
Ops dashboard cần:
- queue depth,
- batch status,
- review backlog,
- SLA metrics.

Thay vì join nhiều DB, bạn build một read model “OpsView” từ events như:
- BatchAssigned, BatchSubmitted, ReviewRequired, BatchAccepted.

Read model không cần hoàn hảo; nó cần “đủ nhanh và đủ đúng” cho mục đích đọc.

> **NOTE**  
> Read model có thể eventual consistency. Đừng dùng read model để enforce invariants.

### 2.1 Example Read Model Schema (OpsView)

```sql
-- Read-optimized table (possibly de-normalized)
CREATE TABLE ops_dashboard_view (
    batch_id UUID PRIMARY KEY,
    current_status VARCHAR(50),
    assigned_labeler_id UUID,
    submitted_at TIMESTAMP,
    review_status VARCHAR(50),
    sla_breach_deadline TIMESTAMP,
    INDEX idx_sla (sla_breach_deadline)
);
-- Populated by event handlers (Batches assigned -> Insert, Submitted -> Update)
```

### 2.2 Read model pipeline (event → projection)

```text
BatchAssigned  -> insert OpsView (status=ASSIGNED, assigned_at=...)
BatchSubmitted -> update OpsView (status=SUBMITTED, submitted_at=...)
ReviewRequired -> update OpsView (review_status=REVIEW_REQUIRED)
BatchAccepted  -> update OpsView (status=ACCEPTED, accepted_at=...)
```

Quy tắc: read model chỉ phục vụ query; không dùng để enforce invariants.

### 2.3 Rebuild strategy: khi projection bị lệch

Bạn cần cách rebuild read model:
- replay events từ outbox/event log,
- hoặc ETL batch từ source-of-truth.  

Nếu không có plan rebuild, read model sẽ “lệch dần” và khó debug.


---

## 3) Event Store: khi nào cần, khi nào không?

Event store (event sourcing) có lợi khi bạn cần:
- replay toàn bộ lịch sử để rebuild state,
- temporal queries (“trạng thái tại thời điểm T”),
- audit/compliance cực nặng.

Nhưng event store tăng chi phí:
- schema evolution,
- projections,
- tooling.

Trong ADLP MVP, thường đủ với:
- domain events cho integration,
- audit log append-only,
- versioning cho transcript/dataset.

Khi enterprise yêu cầu traceability cực sâu, bạn cân nhắc event store cho một số contexts (Quality/Payment).

### 3.1 Example Event Store Schema (Postgres)

Nếu xây build-in event store đơn giản:

```sql
CREATE TABLE event_store (
    sequence_id BIGSERIAL PRIMARY KEY,
    stream_id UUID NOT NULL, -- Aggregate ID
    stream_type VARCHAR(50), -- 'Batch', 'Wallet'
    version INT NOT NULL,
    event_type VARCHAR(100), -- 'BatchSubmitted', 'QualityEvaluated'
    payload JSONB NOT NULL,
    occurred_at TIMESTAMP DEFAULT NOW(),
    metadata JSONB,
    UNIQUE(stream_id, version) -- Optimistic Locking
);
```


---

## 4) Audit Trail & Versioning: cách làm “đúng mà không quá đắt”

ADLP cần trả lời câu hỏi enterprise:
- ai sửa nhãn nào, khi nào, vì sao?
- vì sao batch accepted/rejected?
- dataset version nào chứa batch nào?

Bạn có thể đáp ứng bằng:
- transcript versions (immutable snapshots),
- review decision logs,
- policy_version gắn với quality decision,
- export versioning.

Đây là “audit by design” mà không cần event sourcing toàn hệ thống.

### 4.1 Audit log tối thiểu (ADLP)

```sql
CREATE TABLE quality_decision_audit (
    decision_id UUID PRIMARY KEY,
    batch_id UUID NOT NULL,
    reviewer_id UUID NOT NULL,
    decision VARCHAR(30) NOT NULL, -- Accepted/Rejected/Rework
    policy_version VARCHAR(20) NOT NULL,
    reason TEXT,
    decided_at TIMESTAMP NOT NULL
);
```

Audit log giúp bạn trả lời câu hỏi “ai quyết định gì, theo policy nào”.

---

## 5) Caching & performance: cache đúng chỗ để không phá domain

Cache sai chỗ có thể phá invariants. Nguyên tắc:

1) Cache chủ yếu cho reads (query/search/list), không cache decision logic.  
2) Không dựa vào cache để enforce invariants (assign/accept).  
3) Với “điểm đắt tiền”, ưu tiên correctness hơn micro-optimizations.

Ví dụ ADLP:
- Cache labeler profile search results để assignment nhanh hơn (ok).
- Nhưng assign batch phải atomic ở DB/lock layer, không dựa vào cache “available batches”.

### 5.1 Cache invalidation: quy tắc tối thiểu

- TTL ngắn cho dữ liệu thay đổi nhanh (queue depth, availability).  
- Invalidate theo event khi dữ liệu “đắt tiền” đổi (BatchAssigned/BatchAccepted).  
- Không cache “decision state” nếu bạn không có strategy invalidation rõ.  

---

## 6) Áp dụng vào ADLP: data ownership map (thực dụng)

Một mapping đơn giản:
- Ingestion: items metadata + hash dedupe; S3 pointers
- Prelabeling: segments + model_id/version + confidence
- Assignment: batches + assignments + lock TTL
- Labeling: transcript versions + autosave + submission id
- Quality: quality evaluations + reviews + policy_version
- Export: dataset versions + export jobs + download audit
- Wallet: transactions + payout state

Điểm quan trọng: mỗi context sở hữu sự thật của mình, contexts khác đọc qua events/read models hoặc API.

### 6.1 Ownership table (rút gọn)

| Context | Source of Truth | Read Model/Consumer |
|---|---|---|
| Assignment | batches, locks | OpsView, Labeling UI |
| Labeling | transcript versions | Quality, OpsView |
| Quality | decisions, scores | Export, Wallet |
| Export | dataset versions | Customer portal |
| Wallet | earnings, payouts | Finance dashboard |

### 6.2 Data exchange patterns

- **Events** cho workflow async (Quality → Wallet/Export).  
- **Query API** cho UI/view model (Labeling view).  
- **Read model** cho dashboard/ops (OpsView).  

---

## 7) Anti-patterns

### 7.1 Join xuyên context “cho nhanh”
Phá boundary, schema coupling, refactor đau.

### 7.2 Shared table “common”
Shared kernel phình to dưới dạng DB.

### 7.3 Cache như source of truth
Cache stale phá invariants, tạo ghost bugs.

### 7.4 Foreign key xuyên context
FK xuyên schema làm bạn “khóa” evolution.  
Dùng logical reference + event/API thay vì FK.

---

## 8) Exercise có hướng dẫn (45 phút): thiết kế read model cho Ops dashboard

### Bước 1: Chọn 5 chỉ số dashboard
Ví dụ: review backlog, batch throughput, accept rate, SLA breaches, queue depth.

### Bước 2: Chọn 5 events feed
BatchAssigned, BatchSubmitted, ReviewRequired, BatchAccepted, BatchRejected.

### Bước 3: Định nghĩa read model schema (chỉ cho reads)
OpsView: batch_id, status, assigned_at, submitted_at, review_required_at, accepted_at, tier, SLA_status.

### Bước 4: Xác định update rules
Mỗi event update một phần read model.

---

## 9) Artefacts/Deliverables sau chương này

- Data ownership map per bounded context.
- Read model plan cho 1–2 màn hình quan trọng.
- Quyết định audit/versioning strategy (v0).
- Cache policy: cái gì cache, cái gì không.

---

## Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Data ownership rõ theo bounded context (không join/read xuyên DB để “cho nhanh”)  
> - [ ] Query quan trọng có read model/CQRS-lite để giảm coupling và tăng performance  
> - [ ] Audit/retention strategy rõ cho dữ liệu nhạy cảm/financial (compliance by design)  
> - [ ] Cache đặt đúng chỗ (cache reads, không phá invariants)  
> - [ ] Có kế hoạch migration/rollback khi đổi schema/ownership  
