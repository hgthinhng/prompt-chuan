# prompt-chuan

Skill cho Claude Code giúp viết mới và tỉa chỉ dẫn cho Claude đời 5.x và các agent khác: prompt giao việc, system prompt, CLAUDE.md, SKILL.md, subagent, slash command, prompt cho CLI khác như Codex. Viết bằng tiếng Việt.

*A Claude Code skill (in Vietnamese) for writing and trimming instructions for Claude 5.x and other agents.*

## Ý tưởng

Model đời mới tự lập kế hoạch, tự suy luận và làm đúng từng chữ được viết. Prompt tốt chỉ chứa những thứ model không tự biết: mục tiêu thật, bối cảnh, chuẩn "xong" và cách kiểm. Skill này giúp viết đúng những thứ đó, ghi thẳng vào file đích, cắt chỉ dẫn thừa từ prompt cũ, và chạy thử prompt trước khi dùng.

Skill vẫn giữ cấu trúc chặt ở những chỗ cần: model nhỏ, thao tác khó đảo ngược, số liệu, input không tin cậy.

## Cài đặt

Qua plugin marketplace:

```
/plugin marketplace add hgthinhng/prompt-chuan
/plugin install prompt-chuan@prompt-chuan
```

Gọi bằng `/prompt-chuan:prompt-chuan <việc cần prompt>`, hoặc để Claude tự gọi khi bạn nhờ viết prompt. Cập nhật: `/plugin marketplace update prompt-chuan`.

Hoặc dùng như skill thường (lệnh ngắn `/prompt-chuan`, sửa là có hiệu lực ngay):

```
git clone https://github.com/hgthinhng/prompt-chuan.git
ln -s "$PWD/prompt-chuan/skills/prompt-chuan" ~/.claude/skills/prompt-chuan
```

Đừng dùng cả hai cách trên cùng một máy, vì skill sẽ bị nạp hai lần.

Trên claude.ai hoặc Claude Desktop: nén thư mục `skills/prompt-chuan/` thành zip (chạy `zip -r prompt-chuan.zip prompt-chuan/` từ trong `skills/`) rồi tải lên ở phần Skills trong cài đặt.

## Cấu trúc

```
skills/prompt-chuan/
  SKILL.md                         nguyên tắc, khung lõi, bảng đích, cách thử
  reference/claude-code-targets.md CLAUDE.md, SKILL.md, subagent, prompt cho CLI khác, hook
  reference/claude-5x-gotchas.md   effort, prefill, định dạng output, caching, hành vi mặc định
  reference/agents-and-safety.md   agent, tool, điều phối, input không tin cậy, giảm bịa, đo được
```

## Giấy phép

MIT
