# Checklist — Domain Discovery (phỏng vấn + artefacts + rủi ro)

Checklist này dùng trước hoặc trong tuần 1 của dự án để đảm bảo **hiểu domain trước khi vẽ kiến trúc**.  
Mục tiêu: chốt được **vấn đề thật**, **core domain**, và **rủi ro domain** ở mức đủ để bước vào Event Storming.

---

## 1) Câu hỏi phỏng vấn (dành cho Domain Expert)

### Vấn đề & giá trị kinh doanh
- [ ] Vấn đề lớn nhất mà hệ thống phải giải quyết là gì?
- [ ] Thành công được đo bằng chỉ số nào (KPI/SLA)?
- [ ] Nếu giải sai, thiệt hại lớn nhất là gì (tiền/uy tín/pháp lý)?

### Quy tắc nghiệp vụ (invariants)
- [ ] Có quy tắc nào “không được sai” không? (ví dụ: payout chỉ khi batch accepted)
- [ ] Quy tắc nào thay đổi theo tier/khách hàng/điều kiện?
- [ ] Ngoại lệ nào thường xảy ra nhất?

### Workflow & điểm nóng
- [ ] Workflow “đắt tiền” nhất là gì? (đặt deadline/SLA cụ thể)
- [ ] Điểm nào thường gây tranh cãi giữa các bên?
- [ ] Phần nào đang gây đau nhất cho ops/QA/finance?

### Trade-offs
- [ ] Khi xung đột giữa speed và quality, ưu tiên gì?
- [ ] Khi deadline gấp, có hạ threshold không?
- [ ] Khi batch bị nghi gian lận, ai quyết định và trong bao lâu?

---

## 2) Artefacts tối thiểu (đầu ra cần có)

- [ ] Problem statement (1–2 đoạn, rõ “vì sao dự án tồn tại”)
- [ ] Success metrics (KPI/SLA cụ thể)
- [ ] Danh sách 3–5 rủi ro domain lớn nhất
- [ ] Workflow “đắt tiền” (mô tả bằng lời)
- [ ] Glossary seed 10–30 thuật ngữ
- [ ] Danh sách 10–20 domain events thô (ở thì quá khứ)

---

## 3) Rủi ro thường gặp (và tín hiệu cảnh báo)

- [ ] **Sai domain expert:** người tham gia không có quyền quyết định → output mơ hồ.
- [ ] **Thiếu workflow cụ thể:** chỉ nói chung chung → event storming không ra kết quả.
- [ ] **Nhầm core domain:** team giỏi đang làm generic → tín hiệu phân loại sai.
- [ ] **Không ghi lại trade-offs:** tranh luận lặp lại → thiếu ADR.
- [ ] **Thiếu ngôn ngữ chung:** cùng một từ hiểu khác nhau → glossary chưa chốt.

---

## 4) Áp nhanh cho ADLP (gợi ý)

### Workflow “đắt tiền”
- [ ] Premium order 48h (SLA + quality + payout)

### Invariants mẫu
- [ ] Export chỉ lấy dữ liệu accepted
- [ ] Batch chỉ assign cho một labeler tại một thời điểm (lock TTL)
- [ ] Payout chỉ khi quality gate pass

### Trade-offs mẫu
- [ ] Speed vs Quality khi deadline gấp
- [ ] Escalation policy khi phát hiện anomaly/bias

