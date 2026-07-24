# 📋 Pha 1 & 2 — Problem Scan & Quick Assessment (Cá nhân)

## 🏛️ Bối cảnh

Tôi là AI Engineer tại **Vin Smart Future**. Nhiệm vụ của tôi là quét qua các hoạt động vận hành của các công ty thành viên Vingroup để tìm kiếm cơ hội tối ưu hóa bằng AI.

---

# 🔍 Phase 1 — SCAN: Tìm kiếm cơ hội

Sử dụng **4 Lenses** (Lặp lại, Tốn thời gian, AI-upgrade, Stakeholder Pain) để quét vận hành của các công ty thành viên Vingroup.

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **VinFast** | AI-upgrade | Nhân viên CSKH VinFast mất 15-20 phút mỗi lượt tra cứu sổ tay kỹ thuật, FAQ và soạn phản hồi cho khách hàng về lỗi xe điện, lịch bảo dưỡng, và trạm sạc. |
| 2 | **Xanh SM** | Tốn thời gian | Điều phối viên xử lý thủ công các phản hồi khẩn cấp từ tài xế về sự cố sạc pin hoặc va chạm thực địa (mất 15-20 min/lượt). |
| 3 | **Vinhomes** | Lặp lại | Phân loại và điều hướng thủ công các phản ánh của cư dân (mất nước, hỏng thang máy, ồn ào) đến đúng ban quản lý tòa nhà (~200 phản ánh/ngày). |
| 4 | **Vinmec** | Pain từ người khác | Bác sĩ mất 20-30 phút mỗi bệnh nhân để viết tóm tắt hồ sơ xuất viện (Discharge Summary), khiến bác sĩ quá tải và chậm luân chuyển bệnh nhân. |
| 5 | **Vinpearl** | Tốn thời gian | Nhân viên lễ tân kiểm tra thủ công quỹ phòng trống qua nhiều hệ thống (email, PMS) khi xử lý booking đoàn (Group Booking) phức tạp. |
| 6 | **VinFast** | Stakeholder Pain | Kỹ thuật viên đại lý VinFast mất nhiều thời gian tra cứu mã lỗi từ mô tả bằng tiếng Việt của khách hàng, dẫn đến chẩn đoán sai và tái bảo hành. |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

Chọn **top 3 bài toán** từ danh sách SCAN để phân tích nhanh:

---

## Quick Card #1 — VinFast AI Customer Support Copilot

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Nhân viên CSKH VinFast mất quá nhiều thời gian    │
│ tra cứu tài liệu kỹ thuật và soạn phản hồi cho khách hàng.  │
│ Công ty thành viên: [x] VinFast                             │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên Chăm sóc Khách hàng (CSKH)   │
│ tại tổng đài VinFast hỗ trợ xe điện.                       │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Khách hàng gọi điện/chat mô tả vấn đề                  │
│   → 2. CSKH nghe và ghi chú nội dung sự cố                  │
│   → 3. CSKH mở nhiều tab tra cứu FAQ, sổ tay bảo dưỡng,    │
│        thông số kỹ thuật từng dòng xe (VF5/VF8/VF9)         │
│   → 4. CSKH tổng hợp và soạn phản hồi bằng tay              │
│   → 5. CSKH đọc lại cho khách hàng và chốt phương án        │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3-4 (⏱ 12-15 phút)    │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3-4              │
│ (Tự động Retrieval từ Knowledge Base → Draft phản hồi)      │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│ Giảm thời gian xử lý 1 ticket từ 18 phút → dưới 4 phút.     │
│ Tỉ lệ hài lòng (CSAT) tăng từ 82% lên trên 92%.            │
│                                                             │
│ Quick Architecture: [x] LLM Feature (RAG + Draft Response)   │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Card #2 — Vinhomes Phân loại & Điều hướng Phản ánh Cư dân

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Phân loại và route thủ công ~200 phản ánh/ngày    │
│ của cư dân đến đúng ban quản lý tòa nhà.                    │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên trực tổng đài Vinhomes       │
│ Resident (CSKH).                                            │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Cư dân gửi phản ánh qua App Vinhomes Resident          │
│   → 2. CSKH đọc và phân loại thủ công loại phản ánh          │
│   → 3. CSKH tra cứu sổ tay để xác định ban quản lý đúng    │
│   → 4. CSKH chuyển tiếp phản ánh qua email nội bộ           │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 (⏱ 5 phút/lượt,    │
│ sai sót ~12% do phân loại nhầm)                              │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2-3              │
│ (Tự động phân loại intent và gán đúng ban quản lý)          │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│ Giảm thời gian xử lý từ 8 phút → dưới 1 phút.              │
│ Tỉ lệ route đúng ban quản lý đạt trên 95%.                 │
│                                                             │
│ Quick Architecture: [x] Rule + LLM (Router Classifier)      │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Card #3 — Vinmec AI-Assisted Discharge Summary

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Bác sĩ Vinmec mất 20-30 phút viết tóm tắt xuất    │
│ viện cho mỗi bệnh nhân từ dữ liệu lâm sàng phân tán.        │
│ Công ty thành viên: [x] Vinmec                              │
│                                                             │
│ Ai đang đau (Actor)? Bác sĩ điều trị tại các bệnh viện      │
│ Vinmec trên toàn quốc.                                      │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                        │
│   1. Bác sĩ kết thúc điều trị, quyết định cho xuất viện    │
│   → 2. Bác sĩ mở nhiều hồ sơ: kết quả xét nghiệm,           │
│        phác đồ điều trị, ghi chú hằng ngày                   │
│   → 3. Bác sĩ tổng hợp thủ công và soạn thảo bản tóm tắt   │
│   → 4. Bác sĩ in, ký, và chuyển cho bệnh nhân              │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3 (⏱ 20 phút,      │
│ dễ bỏ sót thông tin lâm sàng quan trọng)                    │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2-3              │
│ (Trích xuất và tổng hợp từ EHR → Draft tóm tắt)             │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│ Giảm thời gian soạn thảo từ 25 phút → dưới 5 phút.         │
│ Bác sĩ duyệt và chỉnh sửa dưới 2 phút (Acceptance rate).   │
│                                                             │
│ Quick Architecture: [x] LLM Feature (Summarization + HITL)  │
└─────────────────────────────────────────────────────────────┘
```
