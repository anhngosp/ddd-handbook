**Event Storming** là một kỹ thuật hội thảo (workshop) mang tính tương tác cao, dùng để khám phá và mô hình hóa các quy trình nghiệp vụ phức tạp. Nó được sáng tạo bởi **Alberto Brandolini** và là một công cụ cực kỳ phổ biến trong cộng đồng **Domain-Driven Design (DDD)**.

Thay vì ngồi viết tài liệu khô khan, Event Storming tập hợp cả **Domain Experts** (người hiểu nghiệp vụ) và **Developers** (người viết code) để cùng nhau "vẽ" lại luồng vận hành của hệ thống trên một bức tường lớn bằng những tờ giấy note màu sắc.

---

## 1. Tại sao Event Storming lại quan trọng?

Trong kiến trúc Microservices, sai lầm lớn nhất là chia service dựa trên Database (Data-driven). Event Storming giúp bạn chia service dựa trên **Hành vi (Behavior-driven)**, từ đó xác định chính xác các **Bounded Context**.

* **Tốc độ:** Có thể mô hình hóa toàn bộ quy trình kinh doanh của một doanh nghiệp lớn chỉ trong vài giờ hoặc vài ngày.
* **Ngôn ngữ chung (Ubiquitous Language):** Xóa bỏ rào cản giao tiếp giữa dân kỹ thuật và dân kinh doanh.
* **Phát hiện lỗ hổng:** Dễ dàng tìm thấy các điểm thắt nút (bottlenecks), các bước thiếu logic hoặc các mâu thuẫn trong quy trình.

---

## 2. Các thành phần chính (Màu sắc của các tờ giấy Note)

Mỗi loại thông tin trong Event Storming được quy định bằng một màu sắc cố định để mọi người nhìn vào là hiểu ngay:

1. **Màu Cam (Domain Event):** Một sự kiện đã xảy ra trong quá khứ (ví dụ: *Hóa đơn đã được tạo*, *Hàng đã xuất kho*). Luôn viết ở thì quá khứ.
2. **Màu Xanh Dương (Command):** Một hành động hoặc quyết định từ người dùng hoặc hệ thống (ví dụ: *Tạo hóa đơn*, *Xác nhận thanh toán*).
3. **Màu Vàng (Actor/User):** Người hoặc nhóm người thực hiện Command.
4. **Màu Hồng (Policy/Rule):** Các quy tắc nghiệp vụ tự động (ví dụ: *KHI hóa đơn đã thanh toán THÌ gửi email cho khách*).
5. **Màu Tím (External System):** Một hệ thống bên ngoài tương tác với quy trình (ví dụ: *Cổng thanh toán ngân hàng*, *Cơ quan thuế*).
6. **Màu Xanh Lá (Read Model):** Dữ liệu mà người dùng cần xem để đưa ra quyết định thực hiện Command.

---

## 3. Quy trình thực hiện Event Storming (Các bước)

### Bước 1: Chaotic Exploration (Khám phá hỗn loạn)

Mọi người cùng nhau ghi tất cả các **Domain Events** (màu cam) mà họ nghĩ ra và dán lên tường. Đừng lo lắng về thứ tự ở bước này.

### Bước 2: Enforce Timeline (Sắp xếp theo thời gian)

Sắp xếp các sự kiện theo trình tự thời gian từ trái sang phải. Nếu có các luồng chạy song song, hãy đặt chúng chồng lên nhau.

### Bước 3: Add Commands & Actors (Thêm hành động)

Xác định cái gì gây ra sự kiện đó (Command - màu xanh dương) và ai là người làm việc đó (Actor - màu vàng).

### Bước 4: Identify Bounded Contexts (Xác định ranh giới)

Sau khi bức tường đã đầy giấy note, bạn sẽ thấy các nhóm sự kiện có liên quan chặt chẽ với nhau. Hãy dùng bút vẽ một đường bao quanh các nhóm đó. Mỗi nhóm này thường là một **Bounded Context** (và sau này là một **Microservice**).

---

## 4. Áp dụng cho Hệ thống Kế toán SaaS (Ví dụ)

Hãy tưởng tượng luồng "Thanh toán hóa đơn":

1. **Command (Xanh dương):** `Nộp yêu cầu thanh toán` (bởi **Khách hàng - Vàng**).
2. **Event (Cam):** `Yêu cầu thanh toán đã nhận`.
3. **External System (Tím):** `Cổng thanh toán VnPay` xử lý.
4. **Event (Cam):** `Thanh toán thành công`.
5. **Policy (Hồng):** *KHI thanh toán thành công THÌ hạch toán sổ cái*.
6. **Event (Cam):** `Bút toán đã ghi sổ`.

---

## 5. Kết quả của Event Storming mang lại cho thiết kế

Sau buổi workshop, bạn sẽ có:

* **Hành trình khách hàng (User Journey)** rõ ràng.
* **Danh sách các Microservices** (dựa trên các cụm giấy note).
* **Danh sách các API** (chính là các Commands).
* **Danh sách các Event** để thiết kế hệ thống Event-Driven (Kafka/RabbitMQ).
