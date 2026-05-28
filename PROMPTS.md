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

---

## Prompt QC hậu kỳ

Chạy QC trên CSV đã gen — auto-check scripts + LLM review + auto-fix tối đa 3 vòng.

```
Đọc .claude/skills/jlpt-reading-long-passage-post-qc/SKILL.md và chạy QC đầy đủ theo workflow.

CSV cần QC: sheets/samples_v1.csv

Phạm vi (chọn 1):
- ALL: toàn bộ CSV (CHỈ N1 và N3)
- LEVEL: chỉ rows có level = {N1|N3}
- ID: chỉ row có _id = {LEVEL}_{uuid}

Quy trình BẮT BUỘC:
1. BƯỚC 1 — Auto-check: chạy post_qc.py + check_furigana + check_spacing + check_csv_fields + check_answer_punctuation
2. BƯỚC 2 — LLM review: L1-L16 (đặc thù 長文: L10 N1 bẫy Criticized-view + 注-anchored, L11 pattern Q1 ref ~38% / Q3 author_opinion ~60%, L12 rate 注/による/(中略))
3. BƯỚC 3 — Cross-batch: B1-B5 (đặc biệt B5 (中略) rate N1 ~30-35%)
4. BƯỚC 4 — Auto-fix: row FAIL → sửa tối thiểu phần lỗi (KHÔNG gen lại toàn bộ), lặp tối đa 3 vòng

Báo cáo theo format trong SKILL.md.
```

---

## Prompt với topic chỉ định

Chỉ định topic cho từng level (CHỈ N1 và N3). Topic dùng tiếng Anh từ cột `en` của `rules/topic.json` (vd: `philosophy`, `science`, `social criticism`).

```
Đọc .claude/skills/jlpt-reading-long-passage/SKILL.md và tuân thủ đầy đủ workflow + 5 GATE.

Gen bài đọc hiểu đoạn văn dài với số bài + topic chỉ định cho từng level (CHỈ N1 và N3 — N3 = 4 câu/bài, N1 = 3 câu/bài):
- N3: 2 bài | topic: environment
- N1: 3 bài | topic: philosophy

Quy tắc:
- Topic PHẢI có trong cột `en` của `rules/topic.json` — kiểm tra trước, không có → DỪNG báo user.
- CSV field `tag` của mỗi row = topic của level đó.
- Nhiều bài cùng level → giữ chung topic NHƯNG mỗi bài khác góc nhìn + question coverage khác đoạn.
- N1 với topic philosophy/social criticism: có thể thêm 注 + source line tự chế.

Lưu CSV: sheets/samples_v1.csv
Lưu HTML: assets/html/doan_van_dai/{LEVEL}_{uuid}.html
```
