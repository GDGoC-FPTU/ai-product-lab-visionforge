# 📝 Phase 6 — AI Log & Reflection (Cá nhân)

## Thành viên 1: Nguyễn Văn A

### AI đã giúp tôi những gì?

Trong buổi lab này, tôi sử dụng Gemini để brainstorm các bài toán tiềm năng cho Vingroup. Ban đầu tôi chỉ nghĩ ra được 2-3 bài toán. Khi tôi yêu cầu Gemini gợi ý thêm các quy trình vận hành thủ công trong ngành ô tô, nó đã đưa ra 5 ý tưởng mới có căn cứ thực tế. Ngoài ra, tôi dùng AI để kiểm tra prompt prototype — nó phát hiện system prompt của tôi còn thiếu một số ràng buộc quan trọng về an toàn.

### AI đã sai ở điểm nào?

AI gợi ý rằng chúng tôi nên xây dựng một multi-agent system phức tạp với 4 agent riêng biệt để xử lý ticket CSKH — một agent phân loại, một agent tra cứu, một agent soạn thảo, và một agent kiểm duyệt. Điều này là over-engineering. Thực tế, một LLM Feature với RAG đơn giản là đủ, vì quy trình CSKH có cấu trúc cố định và rủi ro khi để agent tự trị quá nhiều có thể gây ra lỗi dây chuyền.

### Tôi đã sửa như thế nào?

Tôi yêu cầu Gemini giải thích lý do tại sao cần multi-agent và chỉ ra điểm yếu. Khi nó không đưa ra được lý do thuyết phục, tôi quyết định bỏ qua đề xuất đó và giữ kiến trúc LLM Feature đơn giản. Tôi cũng thêm vào prompt: *"Chỉ đề xuất kiến trúc tối giản nhất có thể giải quyết bài toán"* để tránh over-engineering trong tương lai.

---

## Thành viên 2: Trần Thị B

### AI đã giúp tôi những gì?

Tôi sử dụng ChatGPT để hỗ trợ viết quick problem cards. Tôi mô tả bài toán VinFast, và yêu cầu AI cấu trúc lại workflow thành 5 bước với thời gian ước tính. Nó giúp tôi tiết kiệm thời gian format. Tôi cũng dùng AI để tìm cách tấn công prompt injection vào prototype — nó đưa ra 3 kịch bản tấn công thú vị mà tôi chưa nghĩ tới, như cố tình yêu cầu AI bỏ qua bước kiểm tra kiến thức.

### AI đã sai ở điểm nào?

Khi tôi yêu cầu AI soạn một phản hồi mẫu cho khách hàng VinFast hỏi về lịch bảo dưỡng VF8, AI đã trả lời với thông tin sai: nó nói rằng VF8 cần bảo dưỡng sau mỗi 3.000 km — nhưng thực tế VinFast khuyến cáo 5.000 km cho VF8. Đây là một hallucination điển hình. Nếu phản hồi này được gửi cho khách hàng thật, nó có thể gây hiểu lầm và ảnh hưởng đến uy tín.

### Tôi đã sửa như thế nào?

Tôi thêm vào system prompt một quy tắc: *"Nếu thông tin không có trong knowledge base được cung cấp, TUYỆT ĐỐI không được tự suy luận hoặc bịa đặt — chỉ trả lời dựa trên dữ liệu có sẵn và trích dẫn nguồn."* Tôi cũng thêm trường `source_reference` vào JSON output để bắt buộc AI phải kèm trích dẫn. Sau khi thêm quy tắc này, các phản hồi đều có nguồn gốc rõ ràng.

---

## Thành viên 3: Lê Văn C

### AI đã giúp tôi những gì?

Tôi tập trung vào phần code prototype. Tôi dùng Gemini để support viết và debug hàm `evaluate_prompt()`. Ban đầu tôi gặp lỗi với SDK — tôi import nhầm thư viện `google.generativeai` thay vì `google-genai`. AI đã chỉ ra lỗi và cung cấp code mẫu đúng cú pháp. Tôi cũng dùng AI để thiết kế adversarial test cases — nó gợi ý thêm test case thứ ba về việc yêu cầu AI đưa ra hướng dẫn an toàn vượt quá phạm vi knowledge base.

### AI đã sai ở điểm nào?

AI gợi ý tôi sử dụng `os.environ.get()` để lấy API key, nhưng nó viết sai tên biến môi trường là `GEMINI_API` (thiếu `_KEY`). Khi tôi chạy code, nó không tìm thấy key và báo lỗi. Tôi mất 5 phút để debug trước khi nhận ra tên biến sai. Đây là lỗi nhỏ nhưng cho thấy AI code không phải lúc nào cũng chính xác.

### Tôi đã sửa như thế nào?

Tôi tự kiểm tra lại tên biến môi trường dựa trên documentation của Google và README của lab, sửa thành `GEMINI_API_KEY`. Tôi cũng thêm fallback `GOOGLE_API_KEY` để dự phòng. Về sau, tôi rút kinh nghiệm: không copy code từ AI một cách mù quáng, mà luôn kiểm tra chéo với tài liệu chính thức.

---

## Bài học chung của nhóm

| Bài học | Mô tả |
|---------|-------|
| **AI là công cụ, không phải chuyên gia** | AI giúp brainstorming và tăng tốc độ soạn thảo, nhưng kiến thức chuyên ngành (thông số kỹ thuật, chi phí API) phải do con người kiểm chứng. |
| **Hallucination là rủi ro thực tế** | Cả ba thành viên đều gặp ít nhất một lần AI đưa ra thông tin sai lệch. Điều này khẳng định tầm quan trọng của HITL và trích dẫn nguồn. |
| **Prompt engineering là kỹ năng then chốt** | Cách đặt câu hỏi và ràng buộc ảnh hưởng trực tiếp đến chất lượng output. Chúng tôi đã học cách thêm các quy tắc phủ định ("không được làm gì") để kiểm soát AI. |
| **Kiến trúc tối giản là tốt nhất** | AI thường đề xuất giải pháp phức tạp. Chúng tôi học được rằng "LLM Feature + HITL" đơn giản thường hiệu quả và an toàn hơn multi-agent. |
