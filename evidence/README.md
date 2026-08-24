# Báo Cáo & Bằng Chứng Thực Nghiệm — Day 22: LangSmith + Prompt Versioning

Họ và tên: **Nguyễn Việt Thắng**  
Mã sinh viên: **2A202601321**  
LangSmith Project: `day22-lab`  

---

## 1. Danh sách tệp bằng chứng (Evidence Checklist)

| Tệp | Mô tả | Trạng thái |
|---|---|:---:|
| `01_langsmith_traces.png` | Ảnh chụp màn hình LangSmith UI hiển thị ít nhất 50 traces của pipeline RAG | ✅ Đầy đủ |
| `02_prompt_hub.png` | Ảnh chụp màn hình Prompt Hub hiển thị 2 phiên bản prompt `viet-thang-rag-prompt-v1` và `viet-thang-rag-prompt-v2` | ✅ Đầy đủ |
| `02_ab_routing_log.txt` | File log console định tuyến A/B tất định (50 câu hỏi, hiển thị nhãn v1/v2) | ✅ Đầy đủ |
| `03_ragas_scores.png` | Ảnh chụp màn hình terminal hiển thị bảng so sánh điểm RAGAS giữa V1 và V2 | ✅ Đầy đủ |
| `03_ragas_report.json` | File JSON báo cáo kết quả đánh giá 4 chỉ số RAGAS trên 50 QA pairs | ✅ Đầy đủ |
| `04_pii_demo_log.txt` | Log demo PII Detector (Email, Phone, SSN, Credit Card, Multi-PII, Clean) | ✅ Đầy đủ |
| `04_json_demo_log.txt` | Log demo JSON Formatter (Valid JSON, Markdown fences, Single quotes, Trailing comma, Invalid fallback) | ✅ Đầy đủ |

---

## 2. Phân tích kết quả đánh giá RAGAS (V1 vs V2)

### Bảng tổng hợp chỉ số:

| Chỉ số RAGAS (Metric) | Prompt V1 (Ngắn gọn, thân thiện) | Prompt V2 (Chuyên sâu, cấu trúc) | Winner |
|---|:---:|:---:|:---:|
| **Faithfulness** | **0.8946** ⭐ | 0.8714 ⭐ | **V1 (+0.0232)** |
| **Answer Relevancy** | **0.9140** | 0.8939 | **V1 (+0.0201)** |
| **Context Recall** | **1.0000** | **1.0000** | **Hòa (1.0)** |
| **Context Precision** | 0.9383 | **0.9450** | **V2 (+0.0067)** |

### Phân tích chuyên sâu:

1. **Faithfulness (Độ trung thực):**
   - **V1 (0.8946) cao hơn V2 (0.8714)**: Prompt V1 yêu cầu trả lời ngắn gọn và chỉ dựa trên context, từ đó giảm thiểu việc LLM tự sinh thêm các nhận định phụ hay giải thích mở rộng (extrapolations) không có trong context. V2 tuy yêu cầu trích dẫn nguồn và nêu mức độ chắc chắn, nhưng việc tạo ra các câu trả lời dài hơn vô tình làm tăng nguy cơ sinh ra các mệnh đề diễn giải lại (paraphrasing claims) khó kiểm chứng trực tiếp từ ngữ cảnh.

2. **Answer Relevancy (Độ liên quan):**
   - **V1 (0.9140) vượt trội so với V2 (0.8939)**: Nhờ hướng dẫn trả lời trực diện vào trọng tâm trong 2-4 câu, câu trả lời của V1 bám sát câu hỏi người dùng đặt ra, không bị loãng bởi các thông tin phân loại hay cấu trúc đánh số.

3. **Context Recall (Độ phủ ngữ cảnh):**
   - Cả 2 phiên bản đều đạt điểm tuyệt đối **1.0000 (100%)**, chứng minh chiến lược chunking (`chunk_size=500, overlap=50`) và retriever FAISS ($k=3$) đã trích xuất đầy đủ thông tin chuẩn đối chiếu.

4. **Context Precision (Độ chính xác xếp hạng ngữ cảnh):**
   - Cả 2 đạt điểm rất cao (~0.94), cho thấy các chunk liên quan nhất luôn nằm ở đầu danh sách trả về.

---

## 3. Tổng kết Guardrails Validators

1. **PII Detector**: Phát hiện chính xác và che dấu an toàn (Redaction) cho 4 loại dữ liệu cá nhân nhạy cảm gồm Email, Số điện thoại, Mã số BHXH (SSN), Số thẻ tín dụng (Credit Card).
2. **JSON Formatter**: Tự động gỡ bỏ Markdown code fences, chuyển đổi nháy đơn sang nháy kép hợp lệ, xóa dấu phẩy thừa (trailing commas), và cung cấp fallback an toàn khi dữ liệu đầu vào hoàn toàn không thể phục hồi.
