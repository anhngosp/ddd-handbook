# Chương 4 — Hiểu domain trước khi vẽ kiến trúc

Một hệ thống có thể đạt điểm tuyệt đối ở kỹ thuật nhưng vẫn thất bại ở business. Lý do thường không nằm ở “chọn sai công nghệ”, mà nằm ở việc **đặt sai trọng tâm**: bạn xây quá sâu ở phần generic, và bỏ qua phần core tạo lợi thế cạnh tranh. Chương này là “phanh an toàn” để tránh việc bạn vẽ kiến trúc trước khi hiểu domain.

Trong ADLP, bài học này đặc biệt rõ: nếu bạn đầu tư 3 tuần vào Auth/MFA/Permissions nhưng bỏ qua Quality và Task Assignment, bạn sẽ có một hệ thống bảo mật tuyệt đối… nhưng dataset vẫn rác, consumer vẫn phàn nàn, và marketplace vẫn thua.

---

## Bạn sẽ nhận được gì sau chương này?

1) Phân biệt rõ **Domain** và **Solution** (miền nghiệp vụ vs hệ thống phần mềm).  
2) Hiểu sâu bộ phân loại **Core / Supporting / Generic**, kèm trade-off và chiến lược đầu tư.  
3) Biết cách ưu tiên nguồn lực: **đầu tư chất xám vào đâu để thắng**.  
4) Có bộ **Subdomain Mapping Table** và **Discovery Checklist** để dùng ngay.  
5) Có một bài tập có hướng dẫn (không mơ hồ) để tự phân loại subdomain của dự án.

---

## 1) Domain là gì (và vì sao nhiều team hiểu sai)?

Trong DDD, **Domain** là thế giới nghiệp vụ mà phần mềm đang phục vụ. Domain tồn tại ngay cả khi chưa có hệ thống. Domain là quy tắc vận hành, là logic của business, là cách tổ chức tạo ra giá trị.

Ngược lại, **Solution** là hệ thống phần mềm mà bạn xây để giải quyết domain. Solution có thể thay đổi (framework mới, DB mới, microservice mới), còn domain thì khó thay đổi hơn nhiều — vì nó là cách business vận hành.

Nếu bạn nhầm lẫn domain với solution, bạn sẽ:
- quyết định kiến trúc dựa trên kỹ thuật (DB/UI/CRUD),
- tối ưu thứ dễ thấy (Auth, storage) và bỏ qua thứ tạo giá trị thật (quality, assignment),
- và cuối cùng bạn có một hệ thống “chạy được” nhưng không “đúng”.

> **WARNING**  
> Domain tồn tại trước phần mềm. Phân tích domain là phân tích cách business vận hành, **không phải** phân tích schema DB.

---

## 2) Vì sao phải phân loại subdomain?

Business không đồng nhất. Một nền tảng như ADLP vừa có:
- phần tạo lợi thế cạnh tranh (quality + routing),
- phần hỗ trợ cần có (user profile, ingestion),
- phần phổ quát (auth, notification).

Nếu bạn đối xử tất cả các phần như nhau, bạn sẽ phân tán nguồn lực và “tự hủy” lợi thế.

DDD đưa ra 3 loại subdomain để giúp bạn **đầu tư đúng chỗ**:

### 2.1 Core Domain (miền lõi)
Đây là nơi tạo lợi thế cạnh tranh. Nếu bạn làm tốt, bạn thắng. Nếu bạn làm tệ, bạn thua. Core domain thường có:
- độ phức tạp cao,
- thay đổi thường xuyên theo chiến lược kinh doanh,
- khó mua được từ bên ngoài.

### 2.2 Supporting Subdomain (miền bổ trợ)
Phần này cần thiết để core domain chạy được, nhưng không tạo ra lợi thế cạnh tranh lớn. Nó vẫn đặc thù cho business, nhưng không “lõi”.

### 2.3 Generic Subdomain (miền phổ quát)
Phần này mọi business đều cần, và thị trường đã có giải pháp tốt. Ở đây “tự viết” gần như luôn là waste.

---

## 3) Trade-offs: sai phân loại sẽ trả giá thế nào?

### 3.1 Gọi nhầm Generic là Core
Bạn sẽ:
- dựng một đội dev lớn để viết lại thứ thị trường đã làm tốt hơn,
- mất thời gian và cơ hội ở nơi tạo lợi thế thật,
- làm chậm time-to-market.

Ví dụ trong ADLP: nếu bạn dành 3 sprint để tự xây hệ thống Auth “tối thượng”, bạn sẽ mất 3 sprint không cải thiện quality routing — thứ khiến consumer trả tiền.

### 3.2 Coi thường Core (gọi Core là Supporting/Generic)
Bạn sẽ:
- mua hoặc dùng giải pháp đóng gói cho phần lõi,
- bị khóa vào vendor,
- không thay đổi được khi chiến lược business đổi.

Ví dụ: nếu bạn coi “quality gate + escalation” là generic và mua tool bên ngoài, khi business muốn thay đổi chính sách “premium” hoặc SLA, bạn sẽ bị kẹt.

---

## 4) Heuristics để nhận diện Core / Supporting / Generic

Thay vì tranh luận cảm tính, hãy dùng các câu hỏi thực dụng:

1) **Nếu làm tốt phần này, ta có thắng đối thủ không?**  
2) **Nếu làm kém phần này, ta có thua không?**  
3) **Phần này có thay đổi theo chiến lược kinh doanh không?**  
4) **Thị trường có giải pháp “đủ tốt” chưa?**  

Nếu câu 1–2 trả lời “có”, câu 3 “có”, câu 4 “không”, thì đó là **Core**.  
Nếu câu 4 “có” và phần này không tạo lợi thế, đó là **Generic**.  
Phần còn lại thường là **Supporting**.

> **EXAMPLE (ADLP)**  
> “Quality Assurance” trả lời “có” cho câu 1–2–3 và “không” cho câu 4 → Core.  
> “Identity & Access” trả lời “không” cho 1–2–3 và “có” cho 4 → Generic.

---

## 5) Áp dụng vào ADLP theo Strategic Design v0.2

Dựa trên Strategic Design v0.2, phân loại ADLP như sau:

### Core Domains (đầu tư chất xám sâu)
1) **Quality Assurance** — WER/agreement, automated review, bias detection, escalation; đây là nơi quyết định “dataset có đáng tiền không”.  
2) **Task Assignment & Routing** — skill-matching, priority, deadline, TTL lock, reassign; đây là nơi quyết định throughput và fairness.  
3) **Prelabeling Orchestration** — segmentation, confidence, model routing/versioning; đây là nơi quyết định giảm chi phí gán nhãn.

### Supporting Subdomains (đủ tốt, không over-engineer)
1) **User Profile** — quản lý skills/ratings, cần nhưng không tạo lợi thế nếu làm quá cầu kỳ.  
2) **Wallet & Payment** — quan trọng về chính xác, nhưng pattern thanh toán đã khá chuẩn.  
3) **Data Ingestion** — cần ổn định và chuẩn hóa, nhưng phần “đột phá” không nằm ở đây.

### Generic Subdomains (mua/dùng giải pháp chuẩn)
1) **Identity & Access** — login/logout/RBAC; ưu tiên dùng giải pháp chuẩn.  
2) **Notification** — email/push; dùng dịch vụ có sẵn.  
3) **(Một phần) Export** — đóng gói file, signed URL; không nên tự “phát minh lại bánh xe”.

> **NOTE**  
> “Export” có thể có phần core nếu business value nằm ở versioning hoặc watermarking, nhưng ở MVP thì thường là generic/supporting. Đây là quyết định cần xem lại theo chiến lược.

---

## 6) Best practices khi phân loại subdomain (kèm giải thích)

### 6.1 Luôn bắt đầu từ “lợi thế cạnh tranh”
Đừng hỏi “phần nào khó kỹ thuật nhất”, hãy hỏi “phần nào nếu làm tốt sẽ tạo khác biệt trên thị trường”.  
Trong ADLP, thuật toán routing và quality gate khó hơn cả chuyện storage — nhưng đó cũng là nơi tạo ra lợi thế.

### 6.2 Đừng để team giỏi nhất làm Generic
Đây là sai lầm phổ biến: kỹ sư mạnh bị cuốn vào Auth/infra vì “nhìn kỹ thuật”. Kết quả là core domain thiếu người giỏi.  
Rule thực dụng: **core domain phải có người giỏi nhất**.

### 6.3 Generic = buy or leverage
Generic subdomain là nơi bạn nên mua hoặc dùng giải pháp chuẩn. Nếu bạn tự viết, bạn chỉ đang “đốt” thời gian mà không tăng lợi thế.

### 6.4 Supporting = đủ tốt, không “tối ưu hóa quá sớm”
Supporting domain cần đúng và ổn định, nhưng không cần đầu tư quá mức như core. DDD-lite là đủ.

---

## 7) Anti-patterns thường gặp

### 7.1 “Technical Core Trap”
Nhầm phần khó kỹ thuật (caching, HA, performance) là core domain. Core domain là **business core**, không phải technical core.

### 7.2 “Everything is Core”
Nếu mọi thứ đều core, thì không còn gì là core. Đó là dấu hiệu bạn chưa phân biệt được lợi thế cạnh tranh thật sự.

### 7.3 “In-house Generic”
Tự viết Notification/Logging/Auth từ đầu chỉ để “kiểm soát mọi thứ”. Đây là cách chậm nhất để thua.

---

## 8) Subdomain Mapping Table (template dùng ngay)

Dưới đây là template thực dụng để bạn dùng ngay trong workshop discovery:

| Subdomain | Type | Strategic Importance | Investment Level | Owner | Strategy |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

### Example: Filled Table for ADLP (9 bounded contexts)

| Subdomain | Type | Strategic Importance | Investment Level | Owner | Strategy |
|---|---|---|---|---|---|
| **Task Assignment** | Core | Throughput + fairness trực tiếp ảnh hưởng SLA Premium 48h | High | Assignment Team | In-house DDD + strong invariants |
| **Quality Assurance** | Core | Quyết định “giá trị dataset” và uy tín marketplace | High | Quality Team | In-house DDD + policy/versioning |
| **Prelabeling** | Core | Giảm cost label, tăng margin, lợi thế cạnh tranh | High | AI/ML Team | In-house + ML orchestration |
| **Labeling** | Supporting | UI/ops heavy, quan trọng nhưng không tạo lợi thế độc đáo | Medium | Labeling Ops | DDD-lite + strong UX/read models |
| **Data Ingestion** | Supporting | Cần ổn định pipeline nhưng pattern đã có | Medium | Data Platform | Modular monolith + queue |
| **Dataset Export** | Supporting | Quan trọng cho delivery nhưng ít logic độc đáo | Medium | Delivery Team | DDD-lite + job orchestration |
| **Wallet & Payment** | Generic | Tiền bạc quan trọng nhưng flow chuẩn hóa | Low/Medium | FinTech Team | Integrate 3rd-party + audit |
| **Identity & Access** | Generic | Solved problem (OAuth2/OIDC) | Low | Security Team | Buy/Use Service (Cognito/Auth0) |
| **User Profile** | Supporting | CRUD + rating/skills, không tạo lợi thế lớn | Low/Medium | Ops Team | Simple domain + validations |

**Gợi ý điền nhanh (ADLP):**
- Core contexts có Investment High + owner rõ, tránh outsource.  
- Generic contexts nên ưu tiên buy/integrate, tránh “tự build để kiểm soát”.  

---

## 9) Exercise có hướng dẫn (15 phút): phân loại subdomain cho ADLP

### Bước 1: Liệt kê 8–10 subdomain chính
Gợi ý ADLP: Identity, User Profile, Wallet, Ingestion, Prelabeling, Assignment, Labeling, Quality, Export, Notification.

### Bước 2: Áp câu hỏi 4 trục
Cho từng subdomain, trả lời 4 câu:
1) Có tạo lợi thế cạnh tranh không?  
2) Nếu làm kém có thua không?  
3) Có thay đổi theo chiến lược không?  
4) Thị trường đã có giải pháp tốt chưa?  

### Bước 3: Phân loại và ghi chiến lược đầu tư
Core → build in-house + DDD sâu; Supporting → DDD-lite; Generic → buy/standard.

### Bước 4: Tự kiểm
Nếu team giỏi nhất đang làm Generic, bạn phải điều chỉnh.

### Đáp án tham khảo (rút gọn)
Core: Quality, Assignment, Prelabeling  
Supporting: User Profile, Wallet, Ingestion  
Generic: Identity/Auth, Notification, Export (MVP)

---

## 10) Artefacts/Deliverables sau chương này

1) **Subdomain Mapping Table** hoàn chỉnh.  
2) **Core Domain Vision Statement** (1–2 đoạn mô tả “vì sao core domain tạo lợi thế”).  
3) **Ưu tiên nguồn lực**: ai làm core, ai làm supporting/generic.  
4) **Backlog tập trung**: 60–70% effort cho core domain.

---

## 11) Kết nối sang chương sau

Nếu bạn đã phân loại core/supporting/generic, chương tiếp theo sẽ trả lời một câu hỏi quan trọng: **Ai là domain expert, làm thế nào để khai thác tri thức đúng người, và tránh suy diễn nghiệp vụ?**

---

## Checklist (dùng ngay)

> **CHECKLIST**
> - [ ] Bạn có danh sách capability chính và đang dùng đúng ngôn ngữ business (không dùng tên bảng/tech)  
> - [ ] Bạn phân loại Core/Supporting/Generic và ghi rõ “vì sao” (driver: lợi thế cạnh tranh / rủi ro sai / tốc độ đổi)  
> - [ ] Core domain có 1 workflow “đắt tiền” làm trục để đầu tư sâu  
> - [ ] Bạn đã quyết định chỗ nào “buy/standardize” (generic) để không lãng phí người giỏi  
> - [ ] Bạn có 3 câu hỏi follow-up cho domain expert về phần core (để tránh suy diễn)  
