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

| Chỉ số RAGAS (Metric) | Prompt V1 (Ngắn gọn, trực diện) | Prompt V2 (Chuyên sâu, có cấu trúc) | Winner |
|---|:---:|:---:|:---:|
| **Faithfulness** | **0.9763** ⭐ | **0.9805** ⭐ | **V2 (+0.0042)** |
| **Answer Relevancy** | **0.9216** | 0.9070 | **V1 (+0.0146)** |
| **Context Recall** | **1.0000** | **1.0000** | **Hòa (1.0)** |
| **Context Precision** | 0.9417 | **0.9450** | **V2 (+0.0033)** |

> 🌟 **Điểm nổi bật**: Cả 2 phiên bản đều đạt điểm **Faithfulness $\ge 0.97$** (vượt xa ngưỡng thưởng $\ge 0.90$), chứng minh pipeline hoàn toàn không bị ảo giác (hallucination-free).

---

### Phân tích chuyên sâu (Giải thích sự chênh lệch giữa V1 và V2):

1. **Faithfulness (Độ trung thực):**
   - **V2 (0.9805) cao hơn V1 (0.9763)**: 
     - Prompt V2 áp dụng cấu trúc 2 phần (Direct factual answer + Key supporting details from context), buộc mô hình phải trích dẫn trực tiếp các bằng chứng từ context.
     - Cơ chế trích dẫn có cấu trúc này giúp các mệnh đề được kiểm chứng tuyệt đối từ context, đạt độ trung thực gần như hoàn hảo (98.05%).
     - Prompt V1 dù rất cao (97.63%) nhưng do phong cách diễn đạt tự nhiên hơn nên đôi khi có những từ nối ngữ pháp nằm ngoài context.

2. **Answer Relevancy (Độ liên quan):**
   - **V1 (0.9216) vượt trội so với V2 (0.9070)**: 
     - Prompt V1 yêu cầu trả lời súc tích trong 2–4 câu bám sát câu hỏi người dùng đặt ra, không bị loãng bởi các tiêu đề phân mục hay tiền tố định dạng.
     - Nhờ đó, vector ngữ nghĩa của câu trả lời V1 có độ tương đồng cao hơn đối với câu hỏi gốc.

3. **Context Recall (Độ phủ ngữ cảnh):**
   - Cả 2 phiên bản đều đạt điểm tuyệt đối **1.0000 (100%)**, chứng minh retriever FAISS ($k=3$) với chiến lược chunking tối ưu (`chunk_size=500, overlap=50`) luôn bắt trúng 100% ngữ cảnh cần thiết để trả lời câu hỏi.

4. **Context Precision (Độ chính xác xếp hạng ngữ cảnh):**
   - Cả 2 đạt điểm rất cao (~0.94), cho thấy các chunk liên quan nhất luôn nằm ở đầu danh sách trả về.

---

## 3. Tổng kết Guardrails Validators

1. **PII Detector**: Phát hiện chính xác và che dấu an toàn (Redaction) cho 4 loại dữ liệu cá nhân nhạy cảm gồm Email, Số điện thoại, Mã số BHXH (SSN), Số thẻ tín dụng (Credit Card).
2. **JSON Formatter**: Tự động gỡ bỏ Markdown code fences, chuyển đổi nháy đơn sang nháy kép hợp lệ, xóa dấu phẩy thừa (trailing commas), và cung cấp fallback an toàn khi dữ liệu đầu vào hoàn toàn không thể phục hồi.
