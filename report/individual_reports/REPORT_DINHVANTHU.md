# Individual Report: Lab 3 - Chatbot vs ReAct Agent

- **Student Name**: Đinh Văn Thư
- **Student ID**: 2A202600035
- **Date**: 2026-04-06

---

## I. Technical Contribution (15 Points)

Tôi chịu trách nhiệm phát triển **Weather Tool** - một tool quan trọng hỗ trợ use case “Tôi muốn đi cafe” của nhóm.

- **Modules Implemented**: [`src/tools/weather_tool.py` (chứa hàm chính `get_weather_forecast()`)
- Tích hợp và tối ưu module web crawler `weather_forecast.py`]

- **Code Highlights**:
  [

```python
def get_weather_forecast():
    """
    Use cached weather file if it exists, otherwise crawl fresh data.
    """
    cached_file = _find_latest_cached_weather_file()
    if cached_file and cached_file.exists():
        with open(cached_file, "r", encoding="utf-8") as f:
            return json.load(f)
    return crawl_thoitiet_hourly()
```

]

- **Documentation**:
  [ Các phần tôi đã triển khai:
  Web crawler sử dụng requests + BeautifulSoup để lấy dữ liệu thời tiết theo giờ tại Hà Nội từ thoitiet.vn.
  Cơ chế cache file JSON để giảm số lần crawl, tránh bị block và tăng tốc độ phản hồi.
  Trích xuất các thông tin hữu ích: nhiệt độ, cảm giác nhiệt, độ ẩm, tốc độ gió, UV index, tình trạng thời tiết.
  Hàm trả về dữ liệu dưới dạng JSON chuẩn, dễ dàng để LLM parse và sử dụng trong ReAct loop.

Cách tương tác với ReAct loop:
Tool trả về dữ liệu thời tiết chi tiết → Agent nhận Observation → Thực hiện Thought → Đưa ra đề xuất trang phục phù hợp và gợi ý thời gian đi cafe lý tưởng dựa trên điều kiện thực tế.]

---

# II. Debugging Case Study (10 Points)

- **Problem Description**:
  [Ban đầu, khi agent gọi tool get_weather_forecast(), nó thường không hiểu rõ cấu trúc JSON trả về, dẫn đến hallucination thông tin thời tiết hoặc không đưa ra được đề xuất trang phục hợp lý.]
  **Log Source**:
  [Các dòng log cho thấy Observation rất dài, nhưng agent vẫn trả lời sai, ví dụ: “Nhiệt độ hôm nay khoảng 25°C” trong khi dữ liệu thực tế chỉ từ 18–22°C.]
- **Diagnosis**:
  [LLM chưa được cung cấp few-shot examples tốt về format output của weather tool.
  System prompt ban đầu quá chung chung, không hướng dẫn cụ thể cách đọc các field như temperature, feels_like, condition.
  Model đôi khi bỏ qua dữ liệu cache và cố gắng suy luận dựa trên kiến thức cũ.]

- **Solution**:
  [Tôi đã cập nhật system prompt bằng cách bổ sung few-shot examples cụ thể cho weather tool:
  MarkdownExample:
  Observation: {"temperature": "19°C", "feels_like": "17°C", "condition": "Mưa nhẹ"}
  `Thought`: Thời tiết mát và có mưa nhẹ, nên khuyên mặc áo khoác mỏng và mang ô khi đi cafe.
  Sau khi bổ sung few-shot examples, tỷ lệ agent sử dụng đúng dữ liệu từ weather tool tăng rõ rệt, giảm đáng kể hiện tượng hallucination.]

---

# III. Personal Insights: Chatbot vs ReAct (10 Points)

1. **Reasoning**: Khối Thought trong ReAct giúp agent suy nghĩ logic và từng bước rõ ràng. Thay vì chatbot trả lời chung chung như “Hôm nay trời mát”, agent sẽ phân tích: “Nhiệt độ 19°C, có mưa nhẹ → nên mặc áo dài tay, chọn quán cafe có mái che hoặc indoor”. Nhờ đó, câu trả lời trở nên cụ thể, logic và hữu ích hơn nhiều so với chatbot thông thường.
2. **Reliability**: Agent đôi khi hoạt động tệ hơn chatbot trong một số trường hợp, đặc biệt khi tool thất bại (crawl lỗi hoặc cache quá cũ). Lúc này agent dễ hallucinate mạnh. Trong khi đó, chatbot chỉ dựa vào kiến thức đã huấn luyện nên đôi khi an toàn hơn với các câu hỏi đơn giản.
3. **Observation**: Observation từ tool là yếu tố quan trọng nhất quyết định chất lượng của agent. Nhờ có dữ liệu thực tế (nhiệt độ, tình trạng mưa, UV index…), agent có thể đưa ra quyết định cá nhân hóa cao (đề xuất trang phục và thời gian đi cafe phù hợp). Nếu không có Observation chất lượng, agent gần như không khác biệt so với chatbot thông thường.

---

# IV. Future Improvements (5 Points)

- **Scalability**: [Chuyển từ web crawling sang sử dụng API thời tiết chính thức (OpenWeatherMap hoặc Visual Crossing) để có dữ liệu ổn định, nhanh chóng và hỗ trợ nhiều thành phố.]
- **Safety**: [Thêm guardrail kiểm tra độ tươi mới của dữ liệu (ví dụ: nếu cache cũ hơn 3 giờ thì bắt buộc crawl lại).]
- **Performance**: [Triển khai cơ chế caching thông minh theo thời gian (cache 1 giờ/lần) và mở rộng hỗ trợ nhiều thành phố (Hà Nội, Hải Phòng…).
  Nâng cao use case: Kết hợp Weather Tool với tool tìm quán cafe gần nhất (khoảng cách + đánh giá) để agent đưa ra gợi ý toàn diện hơn, ví dụ: “Nên đi cafe lúc 15h tại quán ABC cách 1.2km vì trời mát mẻ”.]

---
