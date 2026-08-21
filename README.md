# Bài 3: Tối Ưu Truy Vấn RAG Phòng Thủ & Tránh Ảo Tưởng (Defensive RAG System)

## Mô tả
Đây là giải pháp RAG phòng thủ toàn diện (Defensive Retrieval-Augmented Generation) thuộc chương trình **[PTIT - K24 - AI_Integration] Session 02**.

Hệ thống được thiết kế theo kiến trúc nhiều lớp (Multi-layer Security & Optimization Architecture) nhằm giải quyết hai vấn đề cốt lõi trong ứng dụng AI Doanh nghiệp:
1. **Phòng chống tấn công Prompt Injection & Jailbreak.**
2. **Tránh hiện tượng ảo tưởng (Hallucination)** khi LLM đưa ra câu trả lời không có căn cứ trong tài liệu nội bộ.

---

## Kiến Trúc RAG Phòng Thủ (5 Lớp Bảo Vệ)

```
[User Query]
     │
     ▼
┌─────────────────────────┐
│ 1. Input Guardrails     │ ──(Phát hiện Injection/Off-topic)──► [Block Query]
└────────────┬────────────┘
             │ Safe
             ▼
┌─────────────────────────┐
│ 2. Defensive Retrieval  │ ──(Score < Threshold)────────────► [Empty Context]
└────────────┬────────────┘
             │ High Score Contexts
             ▼
┌─────────────────────────┐
│ 3. Strict System Prompt │ ──(Ràng buộc nghiêm ngặt chỉ dựa trên Context)
└────────────┬────────────┘
             │ Draft Response
             ▼
┌─────────────────────────┐
│ 4. Output Verification  │ ──(Low Groundedness Score)────────► [Fallback Response]
└────────────┬────────────┘
             │ Passed Verification
             ▼
[Final Safe Response]
```

---

## Các Tính Năng Nổi Bật

1. **Input Guardrail Filter**: 
   - Kiểm tra Regex & Keyword patterns đối với các cuộc tấn công Prompt Injection (`ignore instructions`, `show system prompt`, `admin rights`...).
   - Giới hạn độ dài câu hỏi để ngăn chặn Buffer Overflow/Token Exhaustion attacks.

2. **Defensive Vector Retrieval**:
   - Sử dụng thuật toán vector TF-IDF / Cosine Similarity đơn giản, nhẹ, tối ưu.
   - Thiết lập **Similarity Score Threshold** (Ngưỡng tin cậy tối thiểu = 0.15). Nếu không có chunk nào vượt ngưỡng, hệ thống lập tức loại bỏ ngữ cảnh nhiễu.

3. **Anti-Hallucination Prompt Engineering**:
   - System Prompt đóng khung cứng (Rigid Frame).
   - Yêu cầu LLM bắt buộc trả lời `"Tôi không tìm thấy thông tin này trong tài liệu được cung cấp."` khi ngữ cảnh không hỗ trợ.

4. **Hậu kiểm Output (Faithfulness & Groundedness Checker)**:
   - Tính toán tỷ lệ từ khóa (Keyword Coverage/Faithfulness Score) giữa câu trả lời tạo ra và ngữ cảnh gốc.
   - Tự động chặn và vô hiệu hóa câu trả lời nếu phát hiện độ trung thực < 60%.

---

## Cấu Trúc File

- `solution.txt`: Mã nguồn Python hoàn chỉnh (bao gồm Vector Store, Guardrails, LLM Engine, Verification và Test Cases).
- `README.md`: Tài liệu hướng dẫn sử dụng và giải thích kiến trúc.

---

## Cách Chạy Chương Trình

Hệ thống được viết bằng Python chuẩn (Standard Python Engine), **không phụ thuộc thư viện ngoài** như LangChain hay ChromaDB để đảm bảo khả năng chạy tức thì trên mọi môi trường.

### Lệnh chạy:
```bash
# Đổi tên file solution.txt thành solution.py nếu chạy trên terminal cục bộ
cp solution.txt solution.py
python solution.py
```

---

## Kết Quả Kiểm Thử (Expected Output)

1. **Query Hợp Lệ**: Tìm kiếm chính xác thông tin từ tài liệu và trả về câu trả lời có trích dẫn.
2. **Query Out-of-Scope**: Trả về thông báo từ chối an toàn thay vì tự bịa câu trả lời.
3. **Query Tấn Công (Prompt Injection)**: Lớp Guardrail chặn ngay từ đầu và ghi log cảnh báo bảo mật.

---

## Giải Thích Chuyên Sâu (Developer Notes)

- **Tại sao cần Thresholding?** Trong các hệ thống RAG thông thường, Vector Search luôn trả về Top-K kết quả dù điểm tương đồng rất thấp. Điều này làm nhiễu LLM và dẫn đến ảo tưởng. Lớp phòng thủ của chúng ta loại bỏ triệt để các kết quả rác này.
- **Tính mở rộng:** Lớp `SimpleTFIDFVectorStore` có thể dễ dàng thay thế bằng `Chroma`, `Pinecone`, hoặc `FAISS` trong môi trường Production mà không ảnh hưởng tới logic của các lớp Guardrail còn lại.