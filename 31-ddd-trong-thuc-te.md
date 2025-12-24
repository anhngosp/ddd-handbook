# Chương 31 — DDD trong môi trường thực tế: khi business đổi, khi team đổi, khi legacy kéo chân

Nếu bạn chỉ học DDD ở mức “vẽ bounded context, vẽ aggregate”, bạn sẽ có cảm giác: mọi thứ rất hợp lý. Nhưng khi bước vào đời thật, thứ làm bạn mệt không phải là khái niệm — mà là thực tế:
- Business đổi liên tục (và thường đổi theo cách “không đẹp”).
- Team đổi người, đổi nhịp, đổi deadline.
- Legacy system vẫn phải sống, vẫn phải tích hợp, vẫn phải “đẻ thêm feature”.
- Production có incident, và mỗi incident bẻ cong kiến trúc một chút.

Ở chương này, ta không nói “DDD đúng nhất”. Ta nói “DDD sống sót”. Mục tiêu là giúp bạn:
1) giữ được ngôn ngữ và ranh giới khi hệ thống tiến hóa,
2) biết lúc nào refactor strategic (bounded context) và lúc nào chỉ refactor tactical,
3) xử lý legacy mà không phá domain model,
4) vận hành DDD như một kỷ luật — không phải một buổi workshop rồi thôi.

Ví dụ xuyên suốt: ADLP (AI Data Labeling Platform). Hãy tưởng tượng sau MVP audio STT, business muốn thêm:
- multi-modal (ảnh/text),
- tier pricing phức tạp (premium 48h, “rush”, “enterprise review”),
- compliance nâng cấp,
- và mở marketplace lớn hơn (nhiều labelers, nhiều dự án).

Nếu bạn không có cơ chế tiến hóa, mọi thứ sẽ trượt về “chạy được là được”.

---

## Bạn sẽ nhận được gì sau chương này?

1) Một cách làm “Continuous Discovery” để domain model luôn bám business reality.  
2) Dấu hiệu nào cho thấy bounded context đang sai (và cần refactor).  
3) Các chiến thuật refactor an toàn: rename, split, merge, ACL, published language, strangler.  
4) Cách làm việc với legacy: anti-corruption layer, translation, song song hệ thống, migration từng lát.  
5) Áp dụng vào ADLP: tình huống mở rộng từ audio sang image, và tác động lên contexts.  
6) Best practices + trade-offs + anti-patterns + exercise có hướng dẫn.

---

## 1) Business thay đổi là mặc định — DDD tồn tại để bạn đổi “ít đau hơn”

Nhiều người hiểu nhầm: “làm DDD là để thiết kế đúng ngay từ đầu”. Thực tế, giá trị lớn nhất của DDD là:
> **khi business đổi, bạn đổi đúng chỗ**.

Nếu hệ thống không có boundaries rõ, business đổi sẽ kéo bạn vào:
- sửa 12 bảng,
- sửa 9 services,
- sửa UI theo cách không hiểu vì sao,
- và mỗi lần sửa là tạo bug “liên hoàn”.

DDD không ngăn business đổi. DDD giúp bạn:
- localize change,
- bảo vệ invariants,
- và giữ ngôn ngữ không bị méo.

> **NOTE**  
> “Đúng ngay từ đầu” là ảo tưởng. “Đổi có kỷ luật” là kỹ năng.

---

## 2) Continuous Discovery: bạn không “chốt domain” một lần rồi thôi

### 2.1 Discovery không phải một phase — nó là một nhịp làm việc

Trong ADLP, ngay cả khi bạn đã chốt 9 contexts v0.2, bạn vẫn sẽ phát hiện:
- thuật ngữ mới (RushOrder, GoldReviewer, DatasetFreeze),
- rule mới (rework policy, dispute policy),
- và “điểm nóng” mới (quality fairness, bias detection).

Continuous Discovery nghĩa là:
- bạn duy trì glossary như một tài sản sống,
- bạn cập nhật event catalog,
- bạn xem lại hotspots định kỳ,
- và bạn có cơ chế “đưa domain expert quay lại bàn”.

### 2.2 Một nhịp discovery thực dụng (2 tuần / 1 tháng)

Bạn không cần workshop lớn mỗi lần. Một nhịp đơn giản:
1) Chọn 1 kịch bản đắt tiền mới hoặc đang “đau” (incident/bug/feature).
2) Làm mini Event Storming 60–90 phút (big picture mini hoặc process-level).
3) Chốt 3–5 thuật ngữ và 1–2 invariants mới.
4) Viết 1 ADR nếu quyết định ảnh hưởng dài hạn (integration, SLO, retention).
5) Đẩy artefacts vào repo: glossary, event catalog, context map (nếu đổi).

> **BEST PRACTICE**  
> Dùng incident/postmortem như đầu vào discovery. Incident thường là “domain truth bị hiểu sai” hoặc “NFR bị bỏ quên”.

---

## 3) Khi nào domain model cần tiến hóa? (và đừng nhầm “tiến hóa” với “đập đi làm lại”)

Bạn sẽ gặp hai loại thay đổi:

### 3.1 Thay đổi tactical (bên trong context): chỉnh aggregate, VO, rule

Ví dụ ADLP:
- lock TTL thay đổi theo tier (premium TTL khác standard),
- policy scoring thay đổi (quality score v2),
- thêm idempotency cho submit/accept.

Đây là thay đổi bên trong context, thường không cần đổi context map.

### 3.2 Thay đổi strategic (ranh giới context): split/merge/rename/ownership change

Ví dụ ADLP:
- Prelabeling mở rộng sang image/text → policy model routing phức tạp, pipeline mới.
- Export tách thành “Dataset Packaging” và “Distribution/Access” vì compliance/permissions.
- Quality phát triển thành “Quality + Dispute Resolution” (khi marketplace lớn).

Đây là thay đổi làm bạn phải xem lại boundaries, contracts, và team ownership.

> **WARNING**  
> Nếu bạn cố “nhét” thay đổi strategic vào tactical refactor, bạn sẽ tạo một context phình to và cuối cùng thành monolith nghiệp vụ.

---

## 4) Dấu hiệu bounded context đang sai (và bạn đang trả giá)

Hãy coi đây là các “triệu chứng lâm sàng”. Nếu 2–3 cái xuất hiện cùng lúc, bạn nên xem lại strategic boundaries.

1) **Ngôn ngữ bị mâu thuẫn ngay trong cùng module**  
   Ví dụ: “Batch” lúc thì nghĩa là “job”, lúc thì nghĩa là “dataset chunk”, lúc thì nghĩa là “payment unit”.

2) **Đổi một rule mà phải sửa nhiều contexts**  
   Ví dụ: thay đổi “premium deadline” mà bạn phải sửa Assignment, Labeling, Quality, Export, Wallet cùng lúc vì rule nằm rải rác.

3) **Integration trở thành call chain sync dài**  
   Bạn luôn phải gọi A→B→C để hoàn thành một request. Đây là dấu hiệu boundary không đúng hoặc orchestration bị leak.

4) **Một context trở thành “nơi chứa mọi thứ”**  
   Code có một module tên “core” hoặc “common” chứa business rules của nhiều contexts.

5) **Team ownership mơ hồ**  
   Khi incident xảy ra, không ai chắc “ai chịu trách nhiệm”. Ownership mơ hồ thường đi kèm boundary mơ hồ.

> **EXAMPLE (ADLP)**  
> Nếu “Export” vừa lo đóng gói dataset, vừa lo permission của customer, vừa lo billing của premium tier, bạn sẽ thấy bug và tranh cãi vì “export là ai?”.

---

## 5) Refactor bounded context: làm thế nào để “đổi ranh giới” mà không vỡ hệ thống

Strategic refactor khó vì nó đụng:
- contracts (API/events),
- data ownership,
- team boundaries.

Bạn cần chiến thuật “đổi dần”.

### 5.1 Rename trước khi split (để sửa ngôn ngữ)

Nếu thuật ngữ đang sai, rename giúp cả team nói đúng.

Ví dụ: thay vì “Task” (mơ hồ), trong ADLP có thể cần chốt rõ:
- `SegmentTask` (đơn vị label)
- `Batch` (đơn vị giao việc)

Rename là refactor rẻ nhất nhưng lợi nhất.

### 5.2 Split bằng Strangler Pattern (cắt theo lát use case)

Nếu bạn cần tách một context, đừng tách “theo data” trước. Hãy tách theo **use case slice**.

Ví dụ: tách “Distribution/Access” ra khỏi Export:
- bắt đầu bằng use case “Generate signed download URL”
- giữ interface cũ, bên trong route dần sang module mới
- khi ổn, migrate data ownership

### 5.3 Dùng ACL khi tích hợp legacy/external

Khi bạn buộc phải tích hợp hệ thống cũ (legacy) hoặc vendor, hãy để domain “sống sạch” bằng ACL:
- translate thuật ngữ,
- map trạng thái,
- che giấu quirks.

DDD v0.2 đã chốt ADR-006 cho ACL — đó là một quyết định để domain sống lâu.

### 5.4 Published Language + versioning: thay đổi mà không phá consumer

Khi boundary đổi, contract sẽ đổi. Bạn cần:
- schema versioning (semver cho events),
- deprecation plan,
- song song v1/v2.

Chương 24 và checklist integration giúp bạn giữ kỷ luật này.

---

## 6) Dealing with legacy: “không thể sạch ngay”, nhưng bạn có thể sạch dần

Legacy có thể là:
- một hệ thống cũ,
- một module trong cùng repo viết kiểu data-driven,
- hoặc một quyết định MVP bạn đã “nợ”.

Mục tiêu không phải là “rewrite”. Mục tiêu là:
> **đưa domain truth vào đúng chỗ, và cô lập phần bẩn**.

### 6.1 3 chiến thuật thực dụng

1) **Encapsulate & wrap**: bọc legacy bằng interface, đừng để nó lộ ra khắp nơi.  
2) **Translate**: dùng ACL để chuyển ngôn ngữ legacy → domain language.  
3) **Strangle**: cắt dần theo use cases, giữ hệ thống chạy song song.

### 6.2 Ví dụ ADLP: mở rộng sang image/text (legacy xuất hiện tự nhiên)

Ban đầu MVP audio STT có pipeline segmentation riêng. Khi thêm image/text, bạn sẽ có temptation:
- “đập chung” prelabeling thành một pipeline phức tạp,
- hoặc copy/paste pipeline cho mỗi modality.

DDD giúp bạn hỏi đúng:
- “Model routing” có phải core domain không?
- “Prelabeling” có nên tách theo modality không, hay theo capability (segmentation, inference, postprocess)?

Bạn có thể bắt đầu bằng:
- giữ Prelabeling context, nhưng thêm ACL cho model provider (SageMaker) và versioned policy
- tách dần thành modules bên trong prelabeling: segmentation, inference, postprocess
- khi modality bùng nổ, cân nhắc split strategic (tách sub-context) thay vì nhồi thêm.

---

## 7) Best practices (kèm giải thích)

### 7.1 Giữ artefacts sống: glossary, event catalog, context map

Artefacts không phải “tài liệu làm một lần”. Nó là bộ nhớ tổ chức.  
Nếu bạn không cập nhật, team sẽ lại nói bằng từ cũ và code sẽ lệch nghĩa.

### 7.2 Mọi “điểm đau production” phải map lại domain

Khi có incident:
- đừng chỉ fix kỹ thuật,
- hỏi: invariant nào bị phá? contract nào mơ hồ? NFR nào bị bỏ?

Chương 26 đã dạy NFR by design — hãy dùng nó để biến incident thành cải tiến kiến trúc.

### 7.3 Refactor strategic theo lát nhỏ, có đo lường

Mỗi lần đổi boundary:
- define success metrics (latency, error rate, lead time, incident count),
- rollout dần,
- có rollback strategy.

### 7.4 Đầu tư core domain, tối giản generic

Core contexts cần kỷ luật nhất (test, purity, contracts).  
Generic contexts nên dùng giải pháp chuẩn nhưng vẫn giữ boundaries.

---

## 8) Trade-offs: tiến hóa có chi phí, nhưng đứng yên còn đắt hơn

### 8.1 Mỗi boundary là một lời hứa vận hành

Tách contexts/services làm bạn phải trả:
- observability,
- deployment,
- contract governance.

Nhưng boundary giúp bạn:
- cô lập rủi ro,
- scale độc lập,
- thay đổi đúng chỗ.

### 8.2 “Đừng refactor strategic vì code xấu”

Code xấu thường sửa bằng tactical refactor. Strategic refactor chỉ nên làm khi:
- ngôn ngữ mâu thuẫn,
- ownership mơ hồ,
- integration coupling quá cao,
- hoặc business capability thật sự đã đổi.

---

## 9) Anti-patterns (triệu chứng → hậu quả → cách tránh)

### 9.1 “Chốt domain một lần rồi thôi”

**Triệu chứng:** glossary cũ, event catalog không cập nhật, domain expert biến mất.  
**Hậu quả:** model lệch dần; bug tăng; tranh luận semantics.  
**Cách tránh:** continuous discovery cadence + ownership rõ.

### 9.2 “Rewrite” thay vì “strangle”

**Triệu chứng:** quyết định rewrite vì legacy xấu; dự án kéo dài; business đổi tiếp.  
**Hậu quả:** chết vì never-ending rewrite.  
**Cách tránh:** strangler theo use case slices; giữ production chạy.

### 9.3 Split service theo công nghệ, không theo domain

**Triệu chứng:** tách “DB service”, “Kafka service”, “API service”.  
**Hậu quả:** boundaries vô nghĩa; coupling tăng.  
**Cách tránh:** boundaries theo business capabilities + ownership.

---

## 10) Artefacts/Deliverables

Sau chương này, bạn nên có:
1) **Continuous Discovery plan** (cadence + ai tham gia + output).  
2) **Evolution log**: thay đổi thuật ngữ/invariants/contexts theo thời gian (ghi gọn).  
3) **Refactor strategy** cho 1 boundary đang đau (strangler + contracts + rollback).  
4) **Legacy integration strategy** (ACL + migration slices).  

---

## 11) Exercise (có hướng dẫn) — Refactor “Export” khi compliance siết chặt

### Bài toán
Giả sử ADLP nhận yêu cầu compliance mới: “Quyền truy cập dataset export phải được audit chi tiết, signed URLs phải có TTL ngắn, và phải có cơ chế revoke ngay lập tức”. Team hiện đang có Export context làm cả:
- đóng gói dataset,
- cấp quyền access,
- và ghi audit đơn giản.

Bạn cần thiết kế một kế hoạch tiến hóa trong 2–4 sprint mà không phá khách hàng hiện tại.

### Cách làm (gợi ý từng bước)

**Bước 1 — Viết lại capability bằng ngôn ngữ business**  
Tách rõ: “Packaging” vs “Access Control/Distribution” vs “Audit”.

**Bước 2 — Chẩn đoán boundary**  
Liệt kê các triệu chứng: nơi nào đang trộn ngôn ngữ? ai là owner của quyền truy cập?

**Bước 3 — Chọn chiến thuật**  
Gợi ý: strangler theo use case “generate signed URL”:
- tạo module mới `distribution-access`
- giữ API cũ nhưng route request sang module mới

**Bước 4 — Chốt contracts và versioning**  
Chọn event/API nào thay đổi, và plan chạy song song v1/v2.

**Bước 5 — Chốt đo lường**  
Define: số lần revoke thành công, audit completeness, p95 latency, incident rate.

### Output mong đợi
1) Một bản “before/after context map” (1 trang).  
2) Một kế hoạch 5–7 bước (theo sprint) với rollback strategy.  
3) Danh sách 3 risks lớn nhất và mitigation (outbox, idempotency, audit retention).  

---

## Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Có nhịp continuous discovery (mini workshop) và artefacts sống (glossary/event catalog/hotspots)  
> - [ ] Incident/postmortem được map lại thành “domain truth” và cập nhật artefacts/ADR  
> - [ ] Refactor strategic theo lát nhỏ (strangler) + contracts/versioning + rollback  
> - [ ] Legacy/external được cô lập bằng ACL (domain không bị kéo méo)  
> - [ ] Mọi thay đổi boundary có tiêu chí đo (SLO/lead time/incident rate)  
