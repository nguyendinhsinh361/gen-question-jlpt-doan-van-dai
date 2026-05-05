# Prompt — Gen bài Đoạn Văn Dài (JLPT 長文読解)

## Cách dùng

Copy prompt bên dưới, thay `{số}` rồi paste vào Claude hoặc Gemini.

**⛔ Scope chỉ 2 level**: **N1** (3 câu/bài, ~1000–1150 chars) và **N3** (4 câu/bài, ~550–700 chars). N2/N4/N5 KHÔNG có kind này — KHÔNG gen.

> **🚨 ZERO-TOLERANCE QC**: Chỉ cần **1 tiêu chí FAIL** trong checklist QC của SKILL.md → **fix ngay hoặc gen lại** trước khi sang bài tiếp.

---

## Prompt

```
Đọc .claude/skills/jlpt-reading-long-passage/SKILL.md rồi gen bài đọc hiểu đoạn văn dài:
- N3: {số} bài (4 câu/bài, ~550–700 chars)
- N1: {số} bài (3 câu/bài, ~1000–1150 chars)

⛔ CHỈ N1 và N3. Không gen N2/N4/N5.

Lưu CSV: sheets/samples_v1.csv. HTML: assets/html/doan_van_dai/{LEVEL}_{uuid}.html.

═══ BƯỚC 0 — CHUẨN BỊ (1 lần) ═══
1. Đọc rules/rule_doc_hieu.md (rule giáo viên — source-of-truth, 11 phần). Áp dụng đặc biệt:
   - Phần 2.4 (Thể chia 文体の統一): N1/N3 → 普通形 (だ・である). Văn bản + câu hỏi + 4 đáp án thống nhất 普通形 toàn bộ.
   - Phần 3 (Furigana), Phần 4 (8 loại Q), Phần 5 (5 loại bẫy chuẩn).
2. Đọc rules/content.md + vocabulary.md + technical.md + questions.md.
3. Đọc rules/kanji_jlpt_sensei.csv (2495 kanji) để tra furigana.
4. Load 2 sample/level: scripts/load_references.py --level {N1|N3} --count 2.
5. Scan sheets/samples_v1.csv xem topic + label combo đã dùng.

═══ BƯỚC 1→5 — LẶP CHO TỪNG BÀI ═══
1. Gen _id = {LEVEL}_{uuid32}; chọn format + topic + label combo chưa/ít dùng.
2. Tag = **tiếng Anh** từ cột `en` của rules/topic.json (philosophy, science, economics...). TUYỆT ĐỐI không tiếng Việt/Nhật.
3. Gen HTML: container theo spec, <p> thuần, marker ①②③ khớp Q reference, furigana chỉ vượt level (cấm "Ab"), **toàn bộ 普通形 (Phần 2.4)**.
   - N1: source line tự chế (CẤM tên thật 朝日/読売/村上春樹...). N3 thường KHÔNG có source.
   - N1 nên có 1-2 注 cho thuật ngữ — giải thích bằng tiếng Nhật đơn giản.
4. Gen Q + 4 đáp án (newline \n, KHÔNG prefix); mỗi câu test đoạn/ý KHÁC NHAU; ≥ 2 unique label per bài; distractor ≥ 3 loại bẫy dùng info THẬT. Câu cuối nên là question_author_opinion / question_content_match (test thesis tổng thể, KHÔNG marker).
5. Tạo CSV row bằng scripts/process_html.py. Fill Q&A bằng scripts/fill_qa.py (KHÔNG sửa CSV tay).

═══ BƯỚC 2 — QC ZERO-TOLERANCE (BẮT BUỘC) ═══
Tự đánh giá checklist 4 phần trong SKILL.md, log PASS/FAIL:
- A. HTML + B. Content (chủ đề, từ vựng level, **toàn bộ 普通形**) + C. Q&A (label, đáp án, explain VN+EN, self-solve khớp correct) + D. Multi-Q Coverage
- **1 FAIL = fix ngay hoặc gen lại → refresh CSV (nếu sửa HTML) → QC lại từ đầu**. CẤM bỏ qua.

═══ HARD REJECT (gen lại ngay) ═══
- Q count sai: N1=3 (slot 4-5 empty), N3=4 (slot 5 empty)
- Char range ngoài: N1 1000–1150 | N3 550–700
- 2 câu cùng test 1 đoạn (vi phạm coverage); câu cuối KHÔNG test thesis
- Marker ①②③ trong HTML không khớp Q reference (hoặc marker dư)
- <ruby> thiếu <rt> hoặc <rt> rỗng; furigana dạng "Ab"
- Thể chia trộn lẫn (xuất hiện です・ます trong bài N1/N3)
- N1 dùng tên thật cho source; N4/N5 có 注 (cấm)
- Tag tiếng Việt/Nhật; trong cùng level: trùng topic hoặc <2 unique label per bài

═══ CUỐI BATCH ═══
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_dai --csv sheets/samples_v1.csv
```
