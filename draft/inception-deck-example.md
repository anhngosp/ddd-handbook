Chúng ta sẽ thực hiện bài tập **Thanh trượt đánh đổi (Trade-off Sliders)**. Đây là thời điểm chúng ta "ngửa bài" để xem dự án này thực sự ưu tiên điều gì. Trong kiến trúc Microservices, bạn không thể có tất cả cùng lúc (định lý CAP là một ví dụ).

Hãy lấy ví dụ cụ thể là module **General Ledger (Sổ cái)** của hệ thống kế toán SaaS mà chúng ta đang thiết kế.

---

### Quy tắc bài tập:

Tôi sẽ đưa ra 4 thanh trượt. Mỗi thanh có mức độ từ 1 đến 5. **Tổng điểm của 4 thanh này không được vượt quá 12**. Điều này bắt buộc chúng ta phải hy sinh cái này để lấy cái kia.

## 1. Các thanh trượt đánh đổi

### Thanh 1: Tính nhất quán dữ liệu (Data Consistency)

* **Mức 1:** Chấp nhận dữ liệu lệch nhau vài phút, báo cáo có thể trễ (Eventual Consistency).
* **Mức 5:** Dữ liệu phải khớp ngay lập tức ( trong 1 mili giây), không chấp nhận bất kỳ sai lệch nào.

### Thanh 2: Tốc độ ra mắt (Time to Market)

* **Mức 1:** Làm kỹ, làm chuẩn, thiết kế Microservices xịn sò, Test Coverage 100%. Mất 6 tháng mới ra bản Beta.
* **Mức 5:** Càng nhanh càng tốt để chiếm thị trường. Chấp nhận nợ kỹ thuật (Technical Debt), có thể làm Monolith trước rồi tách sau.

### Thanh 3: Khả năng mở rộng & Chịu tải (Scalability)

* **Mức 1:** Chạy tốt cho 100 khách hàng đầu tiên trên 1 server duy nhất là đủ.
* **Mức 5:** Thiết kế để chịu tải cho 100,000 doanh nghiệp cùng lúc, scale ngang tự động trên Kubernetes.

### Thanh 4: Chi phí hạ tầng & Vận hành (Cost)

* **Mức 1:** Ngân sách eo hẹp, dùng VPS rẻ, không dùng các dịch vụ Cloud đắt đỏ (Managed Kafka, Managed DB).
* **Mức 5:** Sẵn sàng chi tiền cho hạ tầng cao cấp nhất để đảm bảo hệ thống tự động hóa hoàn toàn và bảo mật tối đa.

---

## 2. Lựa chọn của tôi (Mẫu cho hệ thống Kế toán Enterprise)

Nếu là tôi thiết kế hệ thống này để bán cho các doanh nghiệp lớn, tôi sẽ chọn như sau:

* **Tính nhất quán:** **5** (Kế toán mà sai số dư dù chỉ 1 đồng là mất uy tín hoàn toàn).
* **Khả năng mở rộng:** **4** (SaaS mà không scale được khi khách tăng là tự sát).
* **Tốc độ ra mắt:** **2** (Hệ thống tài chính cần sự cẩn trọng hơn là tốc độ).
* **Chi phí:** **1** (Để có Nhất quán và Scale, ta phải chấp nhận tốn tiền cho hạ tầng xịn).

**Tổng điểm:**  (Vừa đủ khung).

---

## 3. Hệ quả kiến trúc từ lựa chọn trên

Từ bảng đánh đổi này, chúng ta suy ra các quyết định kỹ thuật "đắt giá":

1. **Vì Nhất quán = 5:** Chúng ta **bắt buộc** dùng **Transactional Outbox** và **Saga Orchestration**. Tuyệt đối không gọi API lỏng lẻo. Database phải chọn PostgreSQL với mức Isolation cao nhất.
2. **Vì Scale = 4:** Chúng ta chọn **Microservices ngay từ đầu** và triển khai trên **Kubernetes**. Dữ liệu mỗi Tenant phải nằm ở Schema riêng để không bị nghẽn cổ chai.
3. **Vì Tốc độ = 2:** Chúng ta dành 1 tháng đầu chỉ để xây dựng **Hạ tầng dùng chung** (Chassis framework, Logging tập trung, CI/CD) trước khi viết dòng code nghiệp vụ đầu tiên.

---

### Bây giờ đến lượt bạn:

Giả sử bạn đang làm bản **MVP (Minimum Viable Product)** để gọi vốn, nguồn lực có hạn và cần ra mắt trong 2 tháng tới. Bạn sẽ phân bổ 12 điểm này như thế nào?

*(Hãy điền số từ 1-5 cho 4 thanh trượt sao cho tổng = 12, tôi sẽ phân tích kiến trúc tương ứng cho bạn).*