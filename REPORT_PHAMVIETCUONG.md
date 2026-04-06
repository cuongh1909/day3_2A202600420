# Individual Report: Lab 3 - Chatbot vs ReAct Agent

- **Student Name**: Phạm Việt Cường
- **Student ID**: 2A202600420
- **Date**: 6/4/2026

---

## I. Technical Contribution (15 Points)

*Describe your specific contribution to the codebase (e.g., implemented a specific tool, fixed the parser, etc.).*

- **Modules Implementated**:  
  `src/agent/agent.py`

- **Code Highlights**:  

```python
spec = self._tool_index.get(tool_name)
if not spec:
    logger.log_event(
        "AGENT_ERROR",
        {
            "code": "HALLUCINATION_TOOL",
            "tool": tool_name,
            "known": list(self._tool_index.keys()),
        },
    )
    return (
        f"Error: unknown tool '{tool_name}'. "
        f"Valid tools: {', '.join(self._tool_index.keys())}."
    )
    ## Đoạn code trên xử lý trường hợp LLM tự sinh ra tool không tồn tại. Hệ thống sẽ log lỗi HALLUCINATION_TOOL và trả về danh sách tool hợp lệ để agent có thể retry.
```
```python
spec = self._tool_index.get(tool_name)
if spec.get("uses_kwargs"):
    kwargs_dict = kwargs_from_blob(inner)
    out = run_fn(**kwargs_dict)
    ## Đoạn code này hỗ trợ parse JSON arguments thành keyword arguments (**kwargs) để gọi tool theo dạng:
    search_hotels(city="Da Lat", budget=800000)
else:
    parts = normalize_arg_tokens(split_csv_args(inner))
    out = run_fn(parts)
    ##Đoạn này hỗ trợ tool nhận positional arguments hoặc CSV-style arguments như:
    search_hotels("Da Lat", 800000)
    except TypeError as e:
    logger.log_event(
        "AGENT_ERROR",
        {
            "code": "TOOL_ARG_MISMATCH",
            "tool": tool_name,
            "detail": str(e),
        },
    )
    ##Đoạn code trên xử lý lỗi khi LLM truyền sai số lượng hoặc sai kiểu arguments cho tool.
    logger.log_event(
    "TOOL_CALL",
    {"tool": tool_name, "args": inner[:500], "result_preview": str(out)[:300]},
)
```
- **Documentation**: Phần code này đóng vai trò là lớp trung gian giữa LLM và các tool thực tế trong vòng lặp ReAct.

---

## II. Debugging Case Study (10 Points)

*Analyze a specific failure event you encountered during the lab using the logging system.*

- **Problem Description**: AttributeError: Sau khi thay đổi mã nguồn của hàm run để tạo vòng lặp vô hạn (Infinite Loop)
- **Log Source**: 
```python
{"timestamp": "2026-04-06T09:29:48.229244", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 667, "completion_tokens": 189, "total_tokens": 856, "latency_ms": 2800, "cost_estimate": 0.003557}}
{"timestamp": "2026-04-06T09:29:49.937885", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 880, "completion_tokens": 132, "total_tokens": 1012, "latency_ms": 1708, "cost_estimate": 0.00352}}
{"timestamp": "2026-04-06T09:29:51.648087", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 1036, "completion_tokens": 132, "total_tokens": 1168, "latency_ms": 1710, "cost_estimate": 0.00391}}
{"timestamp": "2026-04-06T09:29:52.906839", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 1192, "completion_tokens": 132, "total_tokens": 1324, "latency_ms": 1258, "cost_estimate": 0.0043}}
{"timestamp": "2026-04-06T09:29:54.278465", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 1348, "completion_tokens": 132, "total_tokens": 1480, "latency_ms": 1371, "cost_estimate": 0.00469}}
{"timestamp": "2026-04-06T09:29:55.981953", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 1504, "completion_tokens": 132, "total_tokens": 1636, "latency_ms": 1703, "cost_estimate": 0.00508}}
{"timestamp": "2026-04-06T09:29:57.382297", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 1660, "completion_tokens": 132, "total_tokens": 1792, "latency_ms": 1386, "cost_estimate": 0.00547}}
{"timestamp": "2026-04-06T09:29:58.989404", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 1816, "completion_tokens": 132, "total_tokens": 1948, "latency_ms": 1607, "cost_estimate": 0.00586}}
{"timestamp": "2026-04-06T09:30:01.409484", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 1972, "completion_tokens": 132, "total_tokens": 2104, "latency_ms": 2420, "cost_estimate": 0.00625}}
{"timestamp": "2026-04-06T09:30:03.748015", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 2128, "completion_tokens": 132, "total_tokens": 2260, "latency_ms": 2338, "cost_estimate": 0.00664}}
{"timestamp": "2026-04-06T09:30:05.510048", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 2284, "completion_tokens": 132, "total_tokens": 2416, "latency_ms": 1762, "cost_estimate": 0.00703}}
{"timestamp": "2026-04-06T09:30:07.342841", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 2440, "completion_tokens": 132, "total_tokens": 2572, "latency_ms": 1832, "cost_estimate": 0.00742}}
{"timestamp": "2026-04-06T09:30:08.980082", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 2596, "completion_tokens": 132, "total_tokens": 2728, "latency_ms": 1637, "cost_estimate": 0.00781}}
{"timestamp": "2026-04-06T09:30:11.857621", "event": "LLM_METRIC", "data": {"provider": "openai", "model": "gpt-4o", "prompt_tokens": 2752, "completion_tokens": 132, "total_tokens": 2884, "latency_ms": 2877, "cost_estimate": 0.0082}}

```
- **Diagnosis**: Lỗi loop trong đoạn mã này,cần quan sát sự tương quan giữa Action và Observation.
- **Solution**: Nếu thấy Agent bắt đầu lặp đi lặp lại một hành động hoặc nói nhảm, dừng chương trình ngay lập tức. Chú ý đến các dòng logger.log_event, nếu thấy steps tăng lên con số hàng chục mà kết quả lặp lại, xem lại Prompt hoặc các Tool cung cấp cho nó.

---

## III. Personal Insights: Chatbot vs ReAct

*Reflect on the reasoning capability difference.*

### 1. Reasoning (Suy luận)
Khối lệnh **Suy nghĩ (Reasoning)** đóng vai trò như một "bản đồ tư duy", giúp Agent chia nhỏ một yêu cầu phức tạp thành các nhiệm vụ đơn lẻ có thứ tự logic:

* **Xác định thứ tự ưu tiên:** Thay vì trả lời chung chung như Chatbot (chỉ gợi ý mẹo tìm kiếm), Agent v2 tự nhận thức được rằng cần phải có dữ liệu thời tiết và danh sách khách sạn thực tế *trước* khi đưa ra gợi ý trang phục.
* **Kết nối dữ liệu:** Suy luận giúp Agent nhận ra mối liên hệ giữa các dữ liệu rời rạc (ví dụ: "nhiệt độ 15-18°C + mưa" từ công cụ thời tiết) để đưa ra khuyến nghị cụ thể như "mang theo ô và áo khoác ấm", thay vì chỉ liệt kê trang phục phổ thông như Chatbot.
* **Kiểm chứng điều kiện:** Agent có khả năng so sánh giá thực tế từ Tool với ngưỡng ngân sách **800.000 VND** để loại bỏ các lựa chọn không phù hợp (như Sapa Lodge), điều mà Chatbot không thể thực hiện chính xác do thiếu dữ liệu thực tế.

---

### 2. Reliability (Độ tin cậy)
Mặc dù ReAct Agent thông minh hơn trong việc xử lý tác vụ, nó vẫn có thể kém hiệu quả hơn Chatbot truyền thống trong các trường hợp sau:

* **Lỗi định dạng (Parsing Error):** Nếu Model không tuân thủ đúng cấu trúc JSON hoặc Key-Value, hệ thống dễ bị kẹt trong vòng lặp chẩn đoán lỗi (Format Loop), dẫn đến việc không có câu trả lời cuối cùng hoặc kết quả bị cắt cụt.
* **Chi phí và Tốc độ (Latency):** Theo bảng thống kê, Agent v2 tốn gấp **~6 lần** số token và **~4 lần** số lượt request so với Chatbot. Đối với các yêu cầu đơn giản không cần dữ liệu thực, việc sử dụng Agent gây lãng phí tài nguyên và tăng độ trễ đáng kể.
* **Lỗi chuỗi (Cascade Failure):** Nếu một công cụ (ví dụ: Google Hotels) trả về kết quả sai hoặc rỗng, Agent có thể bị "lạc lối" hoặc suy luận sai lệch trong các bước kế tiếp. Trong khi đó, Chatbot vẫn có thể cung cấp các mẹo tổng quát hữu ích.

---

### 3. Observation (Quan sát)
Phản hồi từ môi trường (Observation) đóng vai trò là **"nguồn sự thật" (Ground Truth)** để Agent điều chỉnh hành động theo thời gian thực:

* **Dẫn dắt hành động tiếp theo:** Sau khi nhận danh sách khách sạn, Agent v2 không dừng lại ngay. Thông qua quan sát thấy có nhiều lựa chọn, nó tự động thực hiện thêm bước `get_hotel_reviews` để kiểm chứng chất lượng trước khi đề xuất — một bước "chăm sóc khách hàng" mà Chatbot không có.
* **Thay đổi nội dung câu trả lời:** Khi công cụ `get_weather` báo có "mưa nhỏ chiều tối", quan sát này ngay lập tức bẻ lái nội dung tư vấn ở bước cuối (thêm "ô/dù"). Điều này chứng minh Agent có khả năng cập nhật tri thức tức thời thay vì chỉ dựa vào kiến thức đóng băng (static knowledge) trong mô hình.

---

## IV. Những cải tiến trong tương lai

Để nâng cấp hệ thống lên cấp độ sản xuất (**Production-ready**), các cải tiến trọng tâm bao gồm:

* **Khả năng mở rộng (Scalability):** Triển khai kiến trúc **hàng đợi bất đồng bộ (Asynchronous Queues)** như Celery hoặc Redis để xử lý song song các lệnh gọi công cụ, giúp hệ thống không bị nghẽn khi quy mô người dùng tăng lớn.
* **An toàn (Safety):** Thiết lập một **LLM Giám sát (Supervisor)** đóng vai trò kiểm duyệt (Guardrails), rà soát các hành động và nội dung phản hồi của tác nhân trước khi gửi đến người dùng cuối để ngăn chặn sai lệch logic hoặc nội dung nhạy cảm.
* **Hiệu suất (Performance):** Tích hợp **Cơ sở dữ liệu Vector (Vector Database)** để thực hiện truy xuất công cụ theo ngữ nghĩa (Semantic Tool Retrieval). Điều này giúp chọn lọc chính xác công cụ cần thiết trong hệ thống có hàng trăm Tool, từ đó tối ưu hóa số lượng Token và giảm độ trễ (Latency).

---

> [!NOTE]
> Submit this report by renaming it to `REPORT_[YOUR_NAME].md` and placing it in this folder.
