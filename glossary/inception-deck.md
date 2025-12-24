Đây là bản mẫu **Inception Deck** (Bộ khung khởi tạo dự án) được tinh chỉnh dành riêng cho kiến trúc **Microservices**. Nó tập trung vào việc thống nhất giữa Biz (Nghiệp vụ) và Dev (Kỹ thuật) để đảm bảo hệ thống không bị "chia sai" ngay từ đầu.

---

# INCEPTION DECK: SMARTACCOUNTING SAAS

**Mục tiêu:** Xác lập khung dự án cho hệ thống Kế toán Microservices.

## 1. Tại sao chúng ta ở đây? (The "Why")

Xây dựng một nền tảng kế toán đám mây cho phép doanh nghiệp:

* Tự động hóa 80% luồng hạch toán từ hóa đơn.
* Cung cấp báo cáo tài chính thời gian thực thay vì đợi đến cuối tháng.
* Dễ dàng mở rộng tính năng mới (ví dụ: Payroll) mà không ảnh hưởng đến phần lõi Sổ cái.

## 2. Bản vẽ "Cái hộp sản phẩm" (The Product Box)

*Nếu hệ thống này là một món hàng trên kệ, nó sẽ ghi gì?*

* **Mặt trước:** "Kế toán không sai sót - Tự động hóa hoàn toàn luồng tiền".
* **Mặt sau:** "Hỗ trợ đa doanh nghiệp (Multi-tenant), Bảo mật chuẩn ngân hàng, Tích hợp mọi ngân hàng lớn".

## 3. Danh sách "KHÔNG LÀM" (The "NOT" List)

Xác định biên giới để tránh Microservices phình to vô tội vạ.

* **KHÔNG:** Xây dựng hệ thống quản lý kho chuyên sâu (WMS).
* **KHÔNG:** Xây dựng hệ thống Chat nội bộ.
* **KHÔNG:** Hỗ trợ các doanh nghiệp sản xuất quá phức tạp (chỉ tập trung Thương mại/Dịch vụ).

## 4. Gặp gỡ các Stakeholders (The Neighbors)

Xác định ai cung cấp kiến thức cho buổi **Event Storming**:

* **Kế toán trưởng:** Chuyên gia về Sổ cái và Thuế (Domain Expert).
* **Giám đốc tài chính (CFO):** Người cần báo cáo (Query Side).
* **DevOps Lead:** Người lo về hạ tầng Microservices (Infrastructure).

## 5. Thanh trượt đánh đổi (The Trade-off Sliders)

Đây là phần quan trọng nhất để đưa ra quyết định kiến trúc:

| Yếu tố | Cố định (Fixed) | Linh hoạt (Flexible) |
| --- | --- | --- |
| **Tính chính xác (Accuracy)** | [ X ] | [ ] |
| **Thời gian ra mắt (Time to Market)** | [ ] | [ X ] |
| **Chi phí hạ tầng (Cost)** | [ ] | [ X ] |
| **Khả năng chịu tải (Scalability)** | [ X ] | [ ] |

> **Hệ quả kiến trúc:** Vì "Tính chính xác" là tuyệt đối, chúng ta chọn **Saga Pattern** thay vì gọi API trực tiếp để đảm bảo không mất dữ liệu.

## 6. Sơ đồ kiến trúc mức cao nhất (C4 Model - Level 1)

Trước khi chia nhỏ, hãy vẽ bức tranh hệ thống tương tác với thế giới:

* **Người dùng:** Kế toán, Chủ doanh nghiệp.
* **Hệ thống bên ngoài:** Tổng cục Thuế, Ngân hàng (Vietcombank, Techcombank), Cổng thanh toán.

## 7. Những nỗi sợ ban đêm (The Risks)

* "Làm sao để đảm bảo dữ liệu Tenant A không bao giờ rò rỉ sang Tenant B?"
* "Nếu hệ thống chia quá nhiều service, việc vận hành sẽ cực kỳ tốn kém."
* "Liệu đội ngũ có đủ trình độ để xử lý giao dịch phân tán (Saga) không?"

---

### Quy trình tiếp theo sau khi có Inception Deck:

1. **Bước 1 (Framing):** Hoàn thiện bản Inception Deck này (mất 1-2 ngày).
2. **Bước 2 (Discovery):** Tổ chức buổi **Event Storming** dựa trên các Stakeholders đã xác định.
3. **Bước 3 (Mapping):** Vẽ **Context Map** để xác định các Microservices dựa trên kết quả Event Storming.
4. **Bước 4 (Detailing):** Viết **DDS (Detailed Design Specification)** cho từng service như chúng ta đã làm ở trên.
