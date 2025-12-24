**Project Framing** (Xác lập khung dự án) là giai đoạn "định hình" ngay từ khi khởi đầu, nhằm mục tiêu thống nhất tầm nhìn, phạm vi và các giới hạn cốt lõi của dự án giữa các bên liên quan (Stakeholders).

Nếu coi dự án là một ngôi nhà, thì Project Framing chính là bước phác thảo bản vẽ và xác định diện tích đất, thay vì bắt tay vào đặt từng viên gạch.

---

## 1. Mục tiêu của Project Framing

Trong kiến trúc phần mềm và đặc biệt là Microservices, Framing giúp trả lời 4 câu hỏi "Tại sao":

* **Why:** Tại sao chúng ta cần làm hệ thống này? (Vấn đề kinh doanh).
* **Who:** Ai là người hưởng lợi và ai là người thực hiện?
* **What:** Những tính năng nào nằm trong (In-scope) và nằm ngoài (Out-of-scope)?
* **How:** Chúng ta đo lường thành công bằng cách nào? (KPIs).

---

## 2. Các thành phần chính trong một Project Framing

Một buổi Framing chuẩn thường bao gồm các hoạt động sau:

### A. Tầm nhìn dự án (Elevator Pitch)

Tóm tắt dự án trong 1-2 câu để bất kỳ ai cũng có thể hiểu.

* *Ví dụ:* "Xây dựng hệ thống kế toán SaaS tự động hóa hạch toán giúp doanh nghiệp SME giảm 50% thời gian quyết toán."

### B. Business Objectives (Mục tiêu kinh doanh)

Xác định các mục tiêu SMART. Với hệ thống Microservices, mục tiêu có thể là khả năng mở rộng (scalability) hoặc tính linh hoạt khi thay đổi luật thuế.

### C. In & Out (Phạm vi dự án)

Sử dụng biểu đồ "In and Out" để tránh hiện tượng **Scope Creep** (Phình đại phạm vi).

* **In:** Tạo hóa đơn, hạch toán tự động, báo cáo thuế.
* **Out:** Quản lý lương nhân sự, quản lý kho nâng cao.

### D. Xác định Stakeholders

Vẽ bản đồ các bên liên quan để biết ai là người ra quyết định, ai là người cung cấp nghiệp vụ (Domain Experts).

---

## 3. Mối quan hệ giữa Framing và Event Storming

Trong thiết kế Microservices theo phương pháp mà chúng ta đang thảo luận:

1. **Project Framing** diễn ra trước để xác định: "Chúng ta đang làm hệ thống Kế toán cho ai?".
2. **Event Storming** diễn ra ngay sau đó để cụ thể hóa: "Bên trong hệ thống Kế toán đó, các sự kiện nghiệp vụ diễn ra như thế nào?".

---

## 4. Công cụ hỗ trợ Framing (Inception Deck)

Để làm Framing hiệu quả, người ta thường dùng bộ công cụ **Inception Deck** (được phổ biến bởi Spotify/ThoughtWorks) gồm 10 câu hỏi/bài tập nhỏ, tiêu biểu như:

* **The Product Box:** Nếu dự án này là một sản phẩm đóng hộp bày bán trên kệ, trên hộp sẽ ghi những ưu điểm gì?
* **The "NOT" List:** Chúng ta rõ ràng sẽ KHÔNG làm gì?
* **Trade-off Sliders:** Ưu tiên cái nào hơn giữa: *Thời gian*, *Phạm vi*, *Chất lượng*, và *Ngân sách*?

---

## 5. Tại sao thiết kế Microservices lại "chết" nếu thiếu Framing?

* **Chia sai Service:** Nếu không rõ tầm nhìn, bạn sẽ chia service theo cảm tính kỹ thuật thay vì ranh giới nghiệp vụ thực tế.
* **Lãng phí nguồn lực:** Đội ngũ sẽ sa đà vào việc thiết kế những service "Generic" (như Notification, Chat) mà quên tập trung vào "Core Domain" (như General Ledger).
* **Xung đột ưu tiên:** Các team Microservices sẽ không biết nên ưu tiên tính năng nào để kịp tiến độ chung của toàn bộ hệ thống.

**Tóm lại:** Project Framing giúp mọi người cùng nhìn về một hướng trước khi sa vào các chi tiết kỹ thuật phức tạp như Bounded Context hay Kafka.
