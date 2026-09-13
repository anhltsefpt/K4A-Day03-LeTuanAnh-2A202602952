# 📊 BÁO CÁO THU HOẠCH NGHIỆM THU BÀI LAB 3 (BƯỚC 3 — SUBMISSION ARTIFACT)

> **Họ và Tên Học viên:** Lê Tuấn Anh  
> **Mã Sinh Viên / Mã Học viên:** 2A202602952  
> **Chủ đề Lựa chọn:** Gợi ý 1.1 — Trợ lý Học vụ & Tra cứu Lịch thi VinUni (Tra cứu điểm GPA, lịch thi và đặt lịch tư vấn học vụ với Cố vấn)  

---

## 1. BẢNG CHẤM ĐIỂM AGENTIC FIT SCORING MATRIX (ĐÁNH GIÁ CHỦ ĐỀ)

| Tiêu chí Đánh giá | Mức độ (1 - 5) | Giải trình chi tiết lý do chọn điểm |
| :--- | :---: | :--- |
| **1. Multi-step Reasoning** | 5 / 5 | Có. Bài toán đòi hỏi chuỗi suy luận nối tiếp (rõ nhất ở TC04 — ReAct đa bước): Agent phải *tra cứu cố vấn học tập của sinh viên trước → sau đó mới đặt lịch hẹn*. Đây là chuỗi ≥ 2 bước phụ thuộc nhau, đúng bản chất multi-step reasoning. |
| **2. Tool Interaction** | 5 / 5 | Có. Hệ thống bắt buộc kết nối MCP Server / CSDL bên ngoài qua ít nhất 2 tool: `academic_query` (tra cứu GPA/lịch thi) và `schedule_appointment` (hành động đặt lịch). Không thể trả lời chỉ bằng tri thức nội tại của LLM. |
| **3. Dynamic Decision** | 5 / 5 | Có. Bước sau phụ thuộc trực tiếp vào observation của bước trước: kết quả tra cứu quyết định có/không đặt lịch và đặt với cố vấn nào. TC05 còn kiểm tra nhánh rẽ động khi nhận `NOT_FOUND` → phản hồi lịch sự, không bịa dữ liệu. |
| **4. Long Horizon Goal** | 4 / 5 | Có. Agent phải giữ mục tiêu "hoàn tất đặt lịch tư vấn cho đúng sinh viên" xuyên suốt nhiều lượt tool call, mang ngữ cảnh (student_id, tên cố vấn, thời gian) từ bước tra cứu sang bước đặt lịch. Chưa đạt 5 vì horizon còn ngắn (2–3 bước/phiên), không kéo dài qua nhiều phiên hay nhiều ngày. |
| **TỔNG ĐIỂM AGENTIC FIT** | **19 / 20** | *Tổng điểm > 12/20 → Bài toán rất phù hợp triển khai Agentic System.* |

---

## 2. TRÍCH XUẤT KẾT QUẢ WATERFALL TRACE LOG (SAU KHI CHẠY TEST SUITE TRÊN API THẬT)

> ⚠️ **YÊU CẦU NGHIỆM THU:** Mở tệp `.env` điền `GEMINI_API_KEY` (hoặc `OPENAI_API_KEY`) để kết nối LLM thật trước khi thực thi `python src/app.py --all`. Bài nộp chỉ dùng Mock Offline Provider sẽ không đạt điểm nghiệm thực tế.

Đoạn trích tiêu biểu dưới đây là **Test Case TC04 (multi_step_reasoning)** sinh ra từ **OpenAI API thật** (`LLM_PROVIDER=openai`, model `gpt-4o-mini`). Trace thể hiện rõ chuỗi ReAct đa bước: **Bước 1** Agent gọi `academic_query` để tra cứu cố vấn của sinh viên → **Bước 2** dùng chính kết quả quan sát được (`advisor = "PGS.TS Nguyễn Văn A"`) để gọi `schedule_appointment` đặt lịch với đúng cố vấn đó → **Bước 3** tổng hợp câu trả lời cuối cùng. `latency_ms` là độ trễ đo thực tế từng bước.

```json
[
  {
    "step": 1,
    "query": "Sinh viên SV2026001 muốn gặp cố vấn học tập của mình. Hãy tra cứu xem cố vấn học tập của sinh viên này là ai, rồi đặt lịch hẹn tư vấn với đúng cố vấn đó vào 14h00 ngày 22/09/2026.",
    "action_type": "TOOL_EXECUTION",
    "tool_name": "academic_query",
    "arguments": {
      "student_id": "SV2026001"
    },
    "observation": {
      "status": "SUCCESS",
      "student_id": "SV2026001",
      "data": {
        "full_name": "Nguyễn Văn An",
        "class": "AI-K4",
        "gpa": 3.85,
        "email": "an.nv@vinuni.edu.vn",
        "status": "Đang học",
        "advisor": "PGS.TS Nguyễn Văn A"
      }
    },
    "latency_ms": 977.73
  },
  {
    "step": 2,
    "query": "Sinh viên SV2026001 muốn gặp cố vấn học tập của mình. Hãy tra cứu xem cố vấn học tập của sinh viên này là ai, rồi đặt lịch hẹn tư vấn với đúng cố vấn đó vào 14h00 ngày 22/09/2026.",
    "action_type": "TOOL_EXECUTION",
    "tool_name": "schedule_appointment",
    "arguments": {
      "student_id": "SV2026001",
      "datetime_str": "14:00 22/09/2026",
      "advisor_name": "PGS.TS Nguyễn Văn A"
    },
    "observation": {
      "status": "SUCCESS",
      "booking_id": "BK-SV2026001-99",
      "student_id": "SV2026001",
      "datetime": "14:00 22/09/2026",
      "advisor": "PGS.TS Nguyễn Văn A",
      "message": "Đặt lịch thành công cho sinh viên SV2026001 với PGS.TS Nguyễn Văn A vào lúc 14:00 22/09/2026."
    },
    "latency_ms": 2250.13
  },
  {
    "step": 3,
    "query": "Sinh viên SV2026001 muốn gặp cố vấn học tập của mình. Hãy tra cứu xem cố vấn học tập của sinh viên này là ai, rồi đặt lịch hẹn tư vấn với đúng cố vấn đó vào 14h00 ngày 22/09/2026.",
    "action_type": "FINAL_ANSWER",
    "thought": "OpenAI phản hồi trực tiếp bằng văn bản (không cần gọi công cụ).",
    "output": "Sinh viên SV2026001, Nguyễn Văn An, đã được đặt lịch hẹn tư vấn với cố vấn học tập của mình, PGS.TS Nguyễn Văn A. Thời gian hẹn là vào lúc 14:00 ngày 22/09/2026.",
    "latency_ms": 1564.54
  }
]
```

---

## 3. TỔNG KẾT KẾT QUẢ NGHIỆM THU & NỘP BÀI

- [x] Đã điền API Key thật trong `.env` và xác nhận Agent chạy mượt mà trên LLM API thật (OpenAI — `LLM_PROVIDER=openai`, model `gpt-4o-mini`, KHÔNG dùng Mock).
- **Tổng số Test Cases đã chạy thành công:** 5 / 5 test cases.
- **Số lượt gọi Tool qua MCP Server chính xác:** 6 lượt (TC02: 1 × `academic_query`; TC03: `academic_query` → `schedule_appointment`; TC04: `academic_query` → `schedule_appointment`; TC05: 1 × `academic_query`; TC01 trả lời trực tiếp, không gọi Tool). Tổng cộng 11 sự kiện trace (6 TOOL_EXECUTION + 5 FINAL_ANSWER).
- **Ghi chú kiểm thử:** Đã xác minh chế độ đàm thoại trực tiếp `python src/app.py --interactive` hoạt động ổn định; TC05 xử lý đúng ca `NOT_FOUND` (mã SV9999999) mà không bịa dữ liệu (Anti-Hallucination); TC04 thực thi chuỗi ReAct đa bước thật sự (tra cứu → đặt lịch với đúng cố vấn).
- **Kết quả đẩy Repo nộp bài:** [ ] Đã Commit và Push mã nguồn thành công lên GitHub cá nhân.

---

> ✅ **HOÀN TẤT NỘP BÀI:** Sao chép đường link GitHub Repository cá nhân của bạn và dán vào ô nộp bài trên hệ thống LMS VLearn để hoàn tất Bài Lab 3!
