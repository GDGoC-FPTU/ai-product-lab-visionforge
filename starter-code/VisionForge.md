# 02 Deep-Dive Report — VisionForge

## Thông tin nhóm

**Tên nhóm:** VisionForge

| STT | Họ và tên | MSSV |
|---|---|---|
| 1 | Bùi Thị Như Ngọc | [Điền MSSV] |
| 2 | [Điền tên thành viên 2] | [Điền MSSV] |
| 3 | [Điền tên thành viên 3] | [Điền MSSV] |
| 4 | [Điền tên thành viên 4] | [Điền MSSV] |

---

# 1. Bài Toán Nhóm Chọn

Nhóm chọn bài toán:

**Trợ lý AI hỗ trợ điều phối sự cố xe Xanh SM gần hết pin**

Trong quy trình vận hành hiện tại, khi tài xế Xanh SM báo xe gần hết pin hoặc không đủ pin để tiếp tục chuyến, điều phối viên phải xử lý thủ công nhiều bước: kiểm tra vị trí xe, tra cứu trạm sạc gần nhất, kiểm tra loại cổng sạc phù hợp, xem trạm còn trụ trống hay không, rồi soạn hướng dẫn gửi lại cho tài xế. Quy trình này tốn thời gian, dễ sai sót và ảnh hưởng trực tiếp đến trải nghiệm của tài xế lẫn khách hàng.

---

# 2. Lý Do Chọn Bài Toán

Nhóm chọn bài toán này vì:

- Bài toán có workflow rõ ràng, dễ mô tả và dễ đo lường.
- Bottleneck nằm ở các bước cụ thể: tra cứu trạm sạc và soạn hướng dẫn cho tài xế.
- Có metric định lượng được: thời gian xử lý/lượt, tỷ lệ hướng dẫn đúng, tỷ lệ cần điều xe cứu hộ.
- AI phù hợp ở vai trò hỗ trợ điều phối viên, không thay thế hoàn toàn con người.
- Rủi ro có thể kiểm soát bằng cơ chế Human-in-the-loop: điều phối viên phải duyệt trước khi gửi hướng dẫn.

## Lý do loại bỏ các bài toán khác

**Vinhomes phân loại phản ánh cư dân:**  
Bài toán cũng phù hợp với LLM, nhưng phạm vi phản ánh cư dân khá rộng, có nhiều tình huống liên quan đến phí dịch vụ, tranh chấp, khiếu nại nhạy cảm. Cần thêm dữ liệu lịch sử và quy trình escalation rõ hơn trước khi prototype.

**Vinmec tóm tắt hồ sơ xuất viện:**  
Bài toán có giá trị cao nhưng thuộc lĩnh vực y tế, rủi ro sai sót lớn. AI cần truy cập dữ liệu bệnh án và bắt buộc có kiểm duyệt nghiêm ngặt từ bác sĩ. Với phạm vi lab, bài toán này phức tạp và rủi ro hơn.

**VinFast đối chiếu hóa đơn sạc điện:**  
Bài toán có tính lặp lại cao, nhưng nhiều phần có thể giải quyết tốt bằng rule-based system hoặc data matching truyền thống. Không nhất thiết cần LLM ở giai đoạn đầu.

---

# 3. Current-State Workflow

Quy trình xử lý thủ công hiện tại:

```text
Tài xế báo xe gần hết pin
        |
        v
Điều phối viên nhận cuộc gọi/ticket
Thời gian: 2 phút
Handoff: Tài xế -> Điều phối viên
        |
        v
Điều phối viên tra vị trí GPS của xe
Thời gian: 2 phút
Handoff: Điều phối viên -> Hệ thống định vị
        |
        v
Điều phối viên tra dashboard trạm sạc VinFast
Thời gian: 5 phút
Bottleneck
        |
        v
Chọn trạm phù hợp và soạn hướng dẫn cho tài xế
Thời gian: 5 phút
Bottleneck
        |
        v
Gửi hướng dẫn cho tài xế hoặc gọi cứu hộ nếu cần
Thời gian: 1 phút
Handoff: Điều phối viên -> Tài xế / Đội cứu hộ