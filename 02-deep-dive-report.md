# 🏗️ Pha 3 & 5 — Deep-Dive Report & Evaluation (Nhóm)

## 👥 Thông tin nhóm

| Thông tin | Chi tiết |
|-----------|----------|
| **Tên nhóm** | VinSmart AI Product Lab |
| **Dự án chọn** | VinFast AI Customer Support Copilot |
| **Thành viên 1** | Nguyễn Văn A — MSSV: 2024xxxxx |
| **Thành viên 2** | Trần Thị B — MSSV: 2024xxxxx |
| **Thành viên 3** | Lê Văn C — MSSV: 2024xxxxx |

---

## 🗳️ Quyết định lựa chọn

Nhóm quyết định chọn bài toán **Card #1 — VinFast AI Customer Support Copilot** để thực hiện Deep-Dive.

### Lý do lựa chọn:
| Tiêu chí | Đánh giá |
|----------|----------|
| **Tác động kinh doanh** | CSKH là mặt trận đầu tiên của VinFast. Mỗi phút chậm trễ ảnh hưởng trực tiếp đến trải nghiệm khách hàng và doanh thu. |
| **Tính khả thi kỹ thuật** | RAG (Retrieval Augmented Generation) trên knowledge base có sẵn là giải pháp proven, chi phí thấp, triển khai nhanh. |
| **Ranh giới an toàn** | Dễ kiểm soát: AI chỉ draft phản hồi, nhân viên CSKH phải duyệt trước khi gửi (Human-in-the-loop). |
| **Dữ liệu có sẵn** | VinFast có sẵn FAQ, sổ tay bảo dưỡng, tài liệu kỹ thuật cho từng dòng xe (VF5, VFe34, VF8, VF9). |

### Lý do loại bỏ các thẻ khác:
| Thẻ | Lý do loại bỏ |
|-----|---------------|
| **Card #2 (Vinhomes)** | Bài toán phân loại phản ánh có thể giải quyết tốt bằng Rule-based classifier + deterministic routing, không cần LLM. Chi phí vận hành LLM cho ~200 lượt/ngày không tối ưu. |
| **Card #3 (Vinmec)** | Rủi ro y tế rất cao. AI sai sót trong tóm tắt bệnh án có thể dẫn đến hậu quả pháp lý nghiêm trọng. Cần thêm thời gian thu thập dữ liệu và xây dựng guardrails trước khi triển khai. |

---

# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Current-State Workflow Mapping

### Quy trình xử lý ticket CSKH hiện tại tại VinFast:

```text
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ Bước 1          │     │ Bước 2          │     │ Bước 3          │     │ Bước 4          │
│ Khách hàng      │     │ CSKH nghe &     │     │ CSKH tra cứu    │     │ CSKH soạn       │
│ gọi điện/chat   │ ──→ │ ghi chú nội     │ ──→ │ FAQ + tài liệu  │ ──→ │ phản hồi và      │
│ mô tả vấn đề    │     │ dung sự cố      │     │ kỹ thuật        │     │ gửi cho khách   │
│                 │     │                 │     │ (nhiều tab)      │     │                   │
│ Ai: Khách hàng  │     │ Ai: CSKH        │     │ Ai: CSKH        │     │ Ai: CSKH         │
│ ⏱ 3 phút        │     │ ⏱ 3 phút       │     │ ⏱ 7 phút 🔴    │     │ ⏱ 5 phút 🔴     │
│ In: Điện thoại  │     │ In: Lời nói    │     │ In: Ghi chú     │     │ In: Thông tin    │
│ Out: Yêu cầu    │     │ Out: Ghi chú   │     │ Out: Dữ liệu    │     │ Out: Phản hồi    │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
                                                                                 │
                                                                                 ▼
                                                                        ┌─────────────────┐
                                                                        │ Bước 5          │
                                                                        │ CSKH đọc lại &  │
                                                                        │ xác nhận với    │
                                                                        │ khách hàng      │
                                                                        │ Ai: CSKH        │
                                                                        │ ⏱ 2 phút        │
                                                                        └─────────────────┘
```

**Ký hiệu:**
- 🔴 **Bottleneck:** Bước 3 (tra cứu) và Bước 4 (soạn phản hồi) — chiếm 12/18 phút (~67% tổng thời gian).
- 🔄 **Handoff:** Bước 1→2 (Khách hàng → CSKH), Bước 4→5 (Hệ thống → CSKH đọc lại).

**Tổng thời gian xử lý thủ công: 18 phút/ticket.**

### Thống kê khối lượng:
- ~150 ticket/ngày tại trung tâm CSKH VinFast Hà Nội.
- 18 phút × 150 = 45 giờ làm việc/ngày.
- ~8 CSKH full-time chỉ để tra cứu và soạn phản hồi.

---

## 3.2. Problem Statement (6-field)

| Field | Nội dung chi tiết |
|-------|-------------------|
| **1. Actor / Operator** | Nhân viên Chăm sóc Khách hàng (CSKH) tại Trung tâm Hỗ trợ VinFast, xử lý các cuộc gọi/chat từ chủ xe điện VF5, VFe34, VF8, VF9. |
| **2. Current Workflow** | Khi khách hàng gọi điện hoặc chat, CSKH ghi chú thủ công nội dung sự cố (lỗi kỹ thuật, lịch bảo dưỡng, câu hỏi về trạm sạc, thủ tục đăng ký). CSKH sau đó mở nhiều tab trình duyệt để tra cứu FAQ nội bộ, sổ tay bảo dưỡng theo từng dòng xe, thông số kỹ thuật, và chính sách bảo hành. Sau khi thu thập đủ thông tin, CSKH tự soạn thảo phản hồi bằng ngôn ngữ tự nhiên và đọc lại cho khách hàng. Toàn bộ quy trình mất trung bình 18 phút/ticket, hoàn toàn thủ công. |
| **3. Bottleneck** | Bước 3 & 4 (mất 12 phút): Tra cứu thủ công FAQ và tài liệu kỹ thuật phân tán trên nhiều hệ thống (cổng nội bộ, SharePoint, file PDF), sau đó soạn thảo phản hồi từ đầu. CSKH thường xuyên không tìm được tài liệu phù hợp hoặc soạn phản hồi thiếu chính xác về thông số kỹ thuật. |
| **4. Business Impact** | Mỗi ngày 150+ ticket CSKH tại Hà Nội. Chi phí nhân công lãng phí: ~45 giờ/ngày cho việc tra cứu và soạn thảo. Tỉ lệ CSAT hiện tại 82% — giảm 5% so với cùng kỳ do thời gian chờ đợi kéo dài. Khoảng 8% ticket phải xử lý lại (reopen) do phản hồi không chính xác. Tổn thất ước tính ~2.4 tỷ VND/năm do hiệu suất CSKH thấp và khách hàng không hài lòng. |
| **5. Success Metric** | **Hiệu suất:** Giảm thời gian xử lý trung bình từ 18 phút xuống dưới 4 phút/ticket (giảm 78%).<br>**Chất lượng:** Tỉ lệ phản hồi chính xác ngay lần đầu (FCR) tăng từ 82% lên 95%.<br>**Hài lòng:** CSAT tăng từ 82% lên trên 92%.<br>**Năng suất:** Một CSKH xử lý được 15+ ticket/giờ (thay vì 3-4 ticket như hiện tại). |
| **6. Operational Boundary** | **AI ĐƯỢC PHÉP:** Truy xuất knowledge base (FAQ, sổ tay kỹ thuật, chính sách bảo hành), tự động soạn thảo phản hồi dạng nháp (draft), đề xuất hướng xử lý dựa trên dữ liệu có sẵn.<br><br>**AI TUYỆT ĐỐI KHÔNG ĐƯỢC PHÉP:** Tự động gửi phản hồi cho khách hàng khi chưa có sự phê duyệt của CSKH (bắt buộc [DRAFT_ONLY] tag). Không được tự chẩn đoán lỗi kỹ thuật ngoài phạm vi knowledge base. Không được đưa ra cam kết bảo hành hoặc chi phí sửa chữa nếu không có dữ liệu chính xác. Không được tư vấn các vấn đề an toàn ngoài hướng dẫn của nhà sản xuất.<br><br>**YÊU CẦU BẮT BUỘC:** Mọi phản hồi AI soạn phải được CSKH review và phê duyệt trước khi gửi. Mọi đề xuất về kỹ thuật phải kèm trích dẫn nguồn từ knowledge base. |

---

## 3.3. Future-State Flow & AI Fit

### AI-Fit Matrix

| Mức độ AI | Phù hợp? | Lý do |
|-----------|----------|-------|
| **Rule / State-Machine** | ❌ | Câu hỏi của khách hàng đa dạng, ngôn ngữ tự nhiên phong phú. Rule không thể cover edge cases và câu hỏi phức tạp về kỹ thuật. |
| **LLM Feature (Chọn)** | ✅ | RAG (Retrieval Augmented Generation) cho phép tra cứu knowledge base và sinh phản hồi tự động. Dễ triển khai, dễ kiểm soát chất lượng qua Human-in-the-loop. |
| **Agentic Loop** | ❌ Rủi ro cao | Agent tự quyết định và hành động có thể gửi phản hồi sai cho khách hàng. Rủi ro về an toàn thông tin và uy tín thương hiệu. Không cần thiết cho quy trình CSKH có cấu trúc. |

### Quy trình tương lai (Future-State Flow)

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │     │ Bước 5       │
│ Khách hàng   │     │ 🔵 AI        │     │ 🔵 Gemini    │     │ 🟢 CSKH      │     │ CSKH gửi     │
│ gọi điện/    │ ──→ │ Nhập ghi chú │ ──→ │ 2.5 Flash    │ ──→ │ duyệt &      │ ──→ │ phản hồi     │
│ chat         │     │ + Phân loại  │     │ RAG → Draft  │     │ chỉnh sửa   │     │ cho khách   │
│              │     │ intent       │     │ phản hồi     │     │              │     │               │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                                           │
                                                                                           ▼
                                                                                  ↩️ Fallback:
                                                                                  Nếu AI không tự tin
                                                                                  (confidence < 85%),
                                                                                  tự động chuyển ticket
                                                                                  lên CSKH cấp cao.
```

### Giải thích quy trình tương lai:

1. **🔵 Bước 2 (AI — Intent Classification):** Khi CSKH nhập ghi chú sự cố, AI tự động phân loại intent (hỏi về sạc, lỗi kỹ thuật, bảo dưỡng, thủ tục...) để xác định knowledge base cần tra cứu.
2. **🔵 Bước 3 (AI — RAG Generation):** Gemini 2.5 Flash truy xuất các tài liệu liên quan từ knowledge base (FAQ, sổ tay kỹ thuật) và sinh phản hồi dạng nháp với thẻ `[DRAFT_ONLY]` ở đầu.
3. **🟢 Bước 4 (Human-in-the-loop):** CSKH đọc, kiểm tra, chỉnh sửa phản hồi nếu cần, sau đó bấm duyệt và gửi. Đây là bước bắt buộc không thể bỏ qua.
4. **↩️ Fallback:** Nếu AI đánh giá confidence dưới 85% (thiếu dữ liệu, câu hỏi phức tạp), hệ thống tự động gắn tag `[NEEDS_ESCALATION]` và chuyển lên CSKH cấp cao.

### So sánh Hiện tại vs Tương lai

| Tiêu chí | Hiện tại (Thủ công) | Tương lai (AI Copilot) |
|----------|---------------------|----------------------|
| Thời gian/ticket | 18 phút | 4 phút |
| Số ticket/CSKH/ngày | ~20 | ~60 |
| Tỉ lệ FCR | 82% | 95% |
| CSAT | 82% | 92%+ |
| Lỗi do tra cứu sai | 8% reopen rate | <2% reopen rate |

---

## 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist

| # | Câu hỏi | Trạng thái | Ghi chú |
|---|---------|-----------|---------|
| 1 | Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test? | ✅ Có | VinFast có sẵn 5000+ ticket logs từ trung tâm CSKH, FAQ chuẩn, sổ tay kỹ thuật PDF. |
| 2 | Rủi ro khi AI sai có nằm trong tầm kiểm soát (qua HITL hoặc Fallback)? | ✅ Có | Bước 4 bắt buộc có HITL. Fallback tự động khi confidence thấp. Mọi phản hồi đều có [DRAFT_ONLY] tag. |
| 3 | Stakeholders sẵn sàng thay đổi quy trình làm việc cũ? | ✅ Có | Ban lãnh đạo VinFast đã phê duyệt thí điểm AI Copilot tại trung tâm CSKH Hà Nội. |

### Đánh giá chi phí

| Hạng mục | Chi phí ước tính | Ghi chú |
|----------|-----------------|---------|
| **Gemini 2.5 Flash API** | ~1,200 USD/tháng | ~150 tickets/ngày × 22 ngày × ~2,000 tokens/ticket × $0.0001/token |
| **Phát triển & Tích hợp** | ~8,000 USD (one-time) | 2 tháng: 1 BE + 1 Prompt Engineer |
| **Knowledge Base xây dựng** | ~3,000 USD (one-time) | Chuẩn hóa FAQ và tài liệu kỹ thuật |
| **Vận hành & Bảo trì** | ~500 USD/tháng | Monitoring, prompt tuning, KB cập nhật |
| **ROI (12 tháng)** | **~380%** | Tiết kiệm ~45 giờ CSKH/ngày = ~450 triệu/năm. Chi phí vận hành ~35 triệu/năm. |

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future

✅ **GO (Bắt đầu xây dựng Prototype)**

### Justification (Lý giải quyết định)

**Luận điểm kỹ thuật:**
- Bài toán được xác định rõ ràng với quy trình hiện tại đã được mapping chi tiết (5 bước, 18 phút/ticket).
- Knowledge base có sẵn và được cấu trúc hóa, phù hợp với RAG architecture.
- LLM Feature (RAG + Gemini 2.5 Flash) là lựa chọn tối ưu — không quá phức tạp (Agentic), không quá cứng nhắc (Rule).
- Ranh giới an toàn được thiết lập chặt chẽ (HITL bắt buộc, [DRAFT_ONLY] tag, Fallback khi confidence <85%).

**Luận điểm chi phí:**
- Chi phí API Gemini 2.5 Flash rất thấp so với lợi ích thu được (~1,200 USD/tháng vs ~37,500 USD/tháng chi phí nhân công lãng phí hiện tại).
- ROI dự kiến ~380% trong năm đầu tiên.
- Không cần đầu tư hạ tầng lớn — tích hợp qua API.

**Luận điểm rủi ro:**
- Rủi ro thấp nhất trong số các bài toán được xem xét vì HITL được yêu cầu ở mọi bước gửi phản hồi.
- Fallback tự động đảm bảo ticket phức tạp luôn được xử lý bởi con người.
- Có thể triển khai theo pha: chỉ 10% ticket trong tháng đầu, sau đó mở rộng dần.

**Kết luận:** Dự án sẵn sàng để bắt đầu xây dựng prototype ngay với scope hẹp (VinFast Hà Nội, giới hạn 3 loại ticket phổ biến nhất)
