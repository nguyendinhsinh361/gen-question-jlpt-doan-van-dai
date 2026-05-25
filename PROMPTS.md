# Prompt — Gen bài Đoạn Văn Dài (JLPT 長文読解)

Copy prompt bên dưới, thay `{số}` rồi paste vào Claude hoặc Gemini.

CHỈ N1 (3 câu, ~1000–1150 chars) và N3 (4 câu, ~550–700 chars).

## Prompt

```
Đọc .claude/skills/jlpt-reading-long-passage/SKILL.md và tuân thủ đầy đủ workflow + 5 GATE.

Gen bài đọc hiểu đoạn văn dài (CHỈ N1 và N3):
- N3: {số} bài (4 câu)
- N1: {số} bài (3 câu)

Lưu CSV: sheets/samples_v1.csv
Lưu HTML: assets/html/doan_van_dai/{LEVEL}_{uuid}.html
```
