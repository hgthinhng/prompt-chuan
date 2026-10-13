# Bẫy kỹ thuật của Claude đời 5.x

Những điểm prompt hoặc cấu hình API hay làm sai trên model mới. Phần này đổi nhanh theo đời model: ghi chú cập nhật lần cuối tháng 10/2026; trước khi dựa vào một tham số, kiểm trang prompting của đúng model đích (https://platform.claude.com/docs , mục "Prompting best practices" và trang riêng của từng model).

Mục lục: 1. Suy luận và effort · 2. Prefill và ép tool · 3. Định dạng output · 4. Tài liệu dài và caching · 5. Hành vi mặc định cần biết

## 1. Suy luận và effort

- Các model đời mới có thinking (adaptive, hoặc luôn bật trên một số model như Opus 5.5). Prompt không cần thẻ `<thinking>`, không cần "hãy nghĩ từng bước".
- Chỉ dẫn cấp cao thường cho kết quả tốt hơn kế hoạch suy luận viết tay; kế hoạch cứng giới hạn model trong trí tưởng tượng của người viết. Cho mục tiêu, ràng buộc, tiêu chí đúng.
- Độ sâu suy nghĩ chỉnh bằng tham số `effort` (`low` tới `max`, mức nào có tuỳ model), không chỉnh bằng lời "nghĩ kỹ" hay "đừng nghĩ nhiều". Mặc định khác nhau giữa các model (ví dụ docs 10/2026: Opus 5.5 là `medium`, Fable 5.1 là `high`). Trong Claude Code: cờ `--effort` hoặc trường `effort` trong frontmatter.
- Không đòi model chép suy luận ra output (trường `reasoning` trong JSON, "show your thinking"). Trên các model 5.x, yêu cầu này có thể bị từ chối. Cần lý do thì xin giải thích ngắn của kết quả; cần nội dung thinking thì đọc qua API.
- `budget_tokens` và `thinking: {type: "disabled"}` lỗi trên một số model mới; dùng adaptive kèm `effort`.
- Thinking tính vào `max_tokens`; chừa đủ chỗ cho lượt agentic dài.
- Trong hội thoại có tool, truyền lại khối thinking nguyên vẹn; sửa hay lọc khối này làm vỡ giao thức.
- Tự kiểm vẫn có ích: "Trước khi kết thúc, kiểm lại theo [tiêu chí] và sửa chỗ sai." Kiểm độc lập tốt nhất là một phiên có context sạch.

## 2. Prefill và ép tool

- Prefill lượt assistant cuối không còn được hỗ trợ từ đời 4.6 trở đi (lỗi 400). Thay bằng chỉ dẫn rõ, thẻ output, hoặc Structured Outputs. Câu assistant ở giữa hội thoại không bị ảnh hưởng.
- Ép dùng tool (`tool_choice` là `any` hoặc một tool cụ thể) bị hạn chế trên một số model mới; kiểm docs model đích. Cách chắc ăn: `tool_choice: auto`, nói trong prompt tool nào dùng khi nào, thêm `strict: true` để giữ input đúng schema.

## 3. Định dạng output

- Nói điều nên làm: "viết văn xuôi liền mạch" tốt hơn "đừng dùng markdown".
- Phong cách của prompt kéo phong cách câu trả lời: prompt ít markdown thì câu trả lời cũng ít markdown.
- Thẻ XML cho output khi cần cắt phần trả lời bằng code: "đặt kết quả trong `<answer>`".
- JSON cho máy đọc: dùng Structured Outputs với schema, rồi validate bằng code. Lời dặn "chỉ xuất JSON" không đủ tin cậy.
- Model đời mới mặc định súc tích hơn và ít bullet hơn; luật chống định dạng viết cho model cũ có thể bỏ. Muốn có tóm tắt sau khi chạy tool thì xin rõ.
- Toán mặc định ra LaTeX; không muốn thì nói.

## 4. Tài liệu dài và caching

- Đặt tài liệu dài ở trên, câu hỏi và chỉ dẫn ở dưới. Mỗi tài liệu bọc trong `<document>` với `<source>` và `<document_content>`.
- Với tài liệu dài: bảo model trích nguyên văn đoạn liên quan trước khi trả lời, để lọc nhiễu và để kiểm được.
- Prompt caching nhận prefix theo thứ tự tools, system, messages. Nội dung tĩnh (vai, chỉ dẫn, ví dụ) đặt trước, nội dung đổi mỗi lần (input, thời gian) đặt sau; điểm cache đặt ở khối cuối cùng còn giữ nguyên giữa các request.
- Tách phần cố định khỏi phần biến; biến bọc trong thẻ XML để ranh giới rõ.

## 5. Hành vi mặc định cần biết

- **Làm đúng từng chữ.** "Bạn có thể gợi ý thay đổi" cho ra gợi ý; "hãy sửa" cho ra bản sửa. Nói rõ mức hành động.
- **Nhạy với chữ nhấn.** Chữ nhấn viết cho model cũ ("CRITICAL: luôn dùng tool X", "nếu phân vân thì dùng X") nay làm tool bị gọi quá đà. Viết "Dùng X khi...".
- **Làm thừa.** Model hay thêm file, lớp trừu tượng, tính năng không ai nhờ. Nêu phạm vi khi cần gọn.
- **Tự song song và tự giao việc.** Model tự gọi tool song song và tự spawn subagent, đôi khi cả khi một lệnh trực tiếp nhanh hơn. Nếu muốn tiết chế: "Việc nhỏ, sửa một file, hoặc việc cần trạng thái chung thì làm trực tiếp."
- **Mặc định thẩm mỹ.** Với frontend không có định hướng, model rơi về vài phong cách quen thuộc; câu "tránh vẻ AI" chỉ đổi mặc định này sang mặc định khác. Gọi tên cụ thể những gì cần tránh, hoặc đưa định hướng thiết kế rõ.
- **Chỉ dẫn viết cho model cũ** thường quá áp đặt và kéo chất lượng xuống. Khi đổi model, rà lại chỉ dẫn, tool và guardrail còn cần không.

## Nguồn

- Prompting best practices: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Extended và adaptive thinking, effort: https://platform.claude.com/docs/en/build-with-claude/extended-thinking
- Prompt caching: https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- Structured outputs: https://platform.claude.com/docs/en/build-with-claude/structured-outputs
