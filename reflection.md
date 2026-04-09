# Individual reflection — Lê Minh Khang (2A202600102)

## 1. Role
LangGraph orchestration owner. Phụ trách thiết kế workflow tổng thể: định nghĩa state contract (`AgentState`), wiring graph giữa các nodes, và logic routing giữa các path (normal / watch / critical). Đây là lớp điều phối trung tâm đảm bảo các node hoạt động đúng thứ tự và giao tiếp đúng dữ liệu.

## 2. Đóng góp cụ thể
- Định nghĩa `AgentState` làm shared contract giữa các nodes, đảm bảo mỗi node đọc/ghi đúng field và không phá vỡ pipeline
- Thiết kế và wiring LangGraph với 4 nodes chính: ingestion → severity → explanation → guardrails, đảm bảo flow rõ ràng và dễ debug
- Implement conditional routing: tách rõ 2 luồng xử lý (normal vs critical escalation) dựa trên severity level
- Viết hàm `run_workflow()` làm entry point duy nhất cho toàn bộ hệ thống, giúp UI chỉ cần gọi 1 interface
- Align interface giữa các node: thống nhất input/output format để tránh mismatch khi integrate

## 3. SPEC mạnh/yếu
- **Mạnh nhất:** Scope orchestration rõ ràng ngay từ đầu — flow được define trước khi code nên khi implement không phải refactor lớn. Các node có boundary rõ, dễ chia việc song song.
- **Yếu nhất:** Chưa formalize rõ confidence/fallback policy trong routing. Ví dụ khi explanation không chắc chắn thì route thế nào chưa được encode thành rule rõ ràng, dẫn đến phải xử lý heuristic ở node thay vì orchestration layer.

## 4. Đóng góp khác
- Hỗ trợ unblock integration khi các node bị mismatch về field (ví dụ thiếu `severity` hoặc sai key), debug trực tiếp trên graph thay vì từng node riêng lẻ
- Sync contract giữa các member: đảm bảo mọi người dùng cùng naming convention trong `AgentState`
- Fix lỗi routing khi graph ban đầu không handle đúng nhánh critical → thêm explicit condition thay vì implicit assumption

## 5. Điều học được
Trước hackathon nghĩ orchestration chỉ là nối các function lại với nhau. Sau khi làm mới thấy: orchestration là nơi enforce contract và đảm bảo tính determinism cho system. Nếu state không rõ ràng hoặc routing mơ hồ thì bug sẽ lan khắp pipeline và rất khó trace. Việc đặt một lớp orchestration rõ ràng giúp giảm phụ thuộc vào hành vi không ổn định của LLM và giữ system predictable hơn.

## 6. Nếu làm lại
Sẽ freeze interface (`AgentState`) sớm hơn trước khi các node bắt đầu code, và viết contract test ngay từ đầu để đảm bảo mọi node tuân thủ schema. Ngoài ra sẽ áp dụng naming/typing chặt hơn (tránh string tự do) để giảm lỗi integration. Routing logic cũng sẽ được define bằng rule rõ ràng thay vì để implicit trong code.

## 7. AI giúp gì / AI sai gì
- **Giúp:** Dùng AI để brainstorm kiến trúc LangGraph ban đầu và debug flow khi graph không chạy đúng thứ tự. AI giúp nhanh chóng identify chỗ sai trong routing condition và gợi ý cách tách node cho rõ responsibility.
- **Sai/mislead:** AI có xu hướng đề xuất kiến trúc quá phức tạp (multi-layer graph, retry logic, async handling) không phù hợp với scope hackathon 1 ngày. Nếu làm theo sẽ mất nhiều thời gian mà không tăng giá trị demo. Bài học: chỉ lấy phần phù hợp, không follow toàn bộ suggestion.

