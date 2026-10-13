# Đích trong Claude Code

Cách viết cho từng loại file chỉ dẫn mà Claude Code đọc. Docs gốc: https://code.claude.com/docs/en/best-practices , https://code.claude.com/docs/en/skills , https://code.claude.com/docs/en/sub-agents . Các trường frontmatter và giới hạn ký tự đổi theo phiên bản, nên kiểm docs khi viết.

Mục lục: 1. CLAUDE.md · 2. SKILL.md và slash command · 3. Subagent · 4. Prompt giao cho subagent · 5. Prompt cho CLI khác · 6. Lời dặn hay hook

## 1. CLAUDE.md (và AGENTS.md)

Được nạp vào mọi phiên, nên mỗi dòng là chi phí lặp lại. Test cho từng dòng: bỏ dòng này thì Claude có mắc lỗi không? Không thì cắt.

Nên ghi:
- Lệnh build, test, lint mà Claude không tự đoán được.
- Quy ước code khác với mặc định phổ biến.
- Cách chạy test, quy ước nhánh và commit, quy trình review.
- Quyết định kiến trúc và lý do, những bẫy môi trường (biến env, dịch vụ cần bật).

Không nên ghi:
- Điều Claude tự thấy khi đọc code, quy ước chuẩn của ngôn ngữ.
- Tài liệu API dài: để link, hoặc dùng `@path/to/file` để import khi cần.
- Thông tin đổi thường xuyên, mô tả từng file một, lời giải thích dài.

Hình dạng: gạch đầu dòng ngắn, câu mệnh lệnh, nhóm theo chủ đề. Có thể đặt ở `~/.claude/CLAUDE.md` (mọi dự án), gốc repo (chia sẻ với nhóm), hoặc thư mục con (nạp khi làm việc ở đó). Nếu Claude vẫn làm sai một điều dù đã ghi, thường là file quá dài và điều đó bị chìm; tỉa file trước khi thêm chữ nhấn.

## 2. SKILL.md và slash command

Cấu trúc: `skills/<tên>/SKILL.md` cùng các file phụ. Slash command cũ ở `.claude/commands/<tên>.md` vẫn chạy, nhưng skill làm được mọi thứ command làm và hơn.

Frontmatter thường dùng:
- `name`: chữ thường, gạch ngang, trùng tên thư mục.
- `description`: câu then chốt quyết định skill có được gọi đúng lúc không. Nêu skill làm gì và khi nào dùng, kể cả những cách người dùng hay diễn đạt yêu cầu; nêu cả khi nào không dùng nếu dễ nhầm. Viết ở ngôi thứ ba. Spec chung giới hạn 1.024 ký tự.
- `argument-hint`: gợi ý tham số khi gõ lệnh. Trong thân dùng `$ARGUMENTS` để nhận phần người dùng gõ sau lệnh.
- `disable-model-invocation: true`: chỉ chạy khi người dùng gõ lệnh, model không tự gọi. Hợp cho việc có tác dụng phụ (deploy, gửi tin).
- `allowed-tools`, `model`, `context: fork`: giới hạn tool, chọn model, chạy trong context riêng.

Thân:
- Dưới 500 dòng. Claude vốn đã giỏi; chỉ ghi điều nó chưa biết hoặc hay làm sai.
- Chọn mức tự do theo độ mong manh: việc có nhiều đường đúng thì nêu mục tiêu và nguyên tắc; thao tác dễ vỡ thì ghi đúng lệnh hoặc để sẵn script.
- Tài liệu chi tiết tách ra file phụ và chỉ dẫn rõ khi nào đọc file nào. Chỉ tham chiếu một cấp (SKILL.md trỏ tới file, file không trỏ tiếp). File dài hơn 100 dòng thì có mục lục ở đầu.
- Thao tác cần kết quả xác định (đổi định dạng, kiểm schema) thì viết script và bảo Claude chạy, thay vì mô tả bằng lời.
- Thử với ít nhất 3 tình huống, có mốc so sánh là không dùng skill.

Khi skill nằm trong plugin, lệnh gọi có tiền tố tên plugin: `/<plugin>:<skill>`.

## 3. Subagent (`.claude/agents/<tên>.md`)

Frontmatter: `name`, `description` (khi nào nên giao việc cho agent này; câu này quyết định agent có được chọn đúng không), `tools` (chỉ những tool cần), `model`. Thân file là system prompt của subagent.

Subagent không thấy cuộc hội thoại của phiên chính. Thân file cần tự đủ: vai trò, phạm vi, cách làm việc, định dạng báo cáo trả về (ngắn, kết luận trước, vì báo cáo sẽ đổ vào context phiên chính).

## 4. Prompt giao cho subagent (lúc spawn)

Viết như spec cho người không có mặt trong cuộc trò chuyện:
- Mục tiêu và lý do; điều quan trọng nhất.
- Bối cảnh cần thiết, kèm đường dẫn tuyệt đối tới file và thư mục.
- Ranh giới: chỉ đọc hay được sửa, không đụng tới đâu.
- Định dạng và độ dài báo cáo trả về; yêu cầu tách điều đã kiểm với điều suy luận.
- Cách kiểm kết quả (test, lệnh chạy thử).

Phiên chính tự tổng hợp kết quả. Tránh giao luôn phần "hiểu" kiểu "dựa vào kết quả tìm được, hãy sửa đi".

## 5. Prompt cho CLI agent khác (Codex, Gemini CLI, Kimi...)

- Tự đủ hoàn toàn: agent kia không có skill, memory hay CLAUDE.md của bạn.
- Prompt dài thì ghi ra file rồi đưa qua stdin, tránh truyền qua tham số dòng lệnh.
- Nêu rõ được đọc hay ghi, thư mục làm việc, và đường dẫn tuyệt đối.
- Chỉ định định dạng trả về cụ thể (Markdown có mục cố định, hoặc JSON với schema đầy đủ). Ví dụ JSON phải điền đủ trường thật, không dùng `...`, vì một số model chép nguyên văn ví dụ.
- Model khác hãng thường cần cấu trúc rõ hơn Claude; xem mục "Khi nào vẫn cần giàn giáo" trong SKILL.md.

## 6. Lời dặn hay hook

CLAUDE.md, SKILL.md và prompt chỉ là lời khuyên; model có thể bỏ qua, nhất là khi context dài. Điều phải đúng 100% thì dùng cơ chế cưỡng chế:
- Hook (`PreToolUse`, `PostToolUse`, `Stop`...) để chặn hoặc tự chạy lệnh, ví dụ chặn ghi vào thư mục cấm, tự chạy formatter sau khi sửa file.
- `permissions.deny` trong settings để cấm hẳn một lệnh hay đường dẫn.
- Sandbox hoặc chạy chỉ đọc khi giao việc cho agent ngoài.

Nếu Claude đã tự làm đúng một điều mà không cần dặn, bỏ dòng dặn đó.
