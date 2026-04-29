# Prompt — Gen bài Đoạn Văn Dài (JLPT 長文読解)

## Cách dùng

Copy prompt bên dưới, thay `{số}` rồi paste vào Claude hoặc Gemini.
SKILL.md chứa workflow + checklist QC. rules/ chứa chi tiết. Prompt chỉ cần nói **cái gì** và **bao nhiêu**.

**⛔ Scope chỉ 2 level**: **N1** (3 câu/bài, ~1000–1150 chars) và **N3** (4 câu/bài, ~550–700 chars). N2/N4/N5 KHÔNG có kind này — KHÔNG gen.

---

## Prompt ngắn (khuyên dùng)

```
Đọc .claude/skills/jlpt-reading-long-passage/SKILL.md rồi gen bài đọc hiểu đoạn văn dài:
- N3: {số} bài (4 câu/bài, ~550–700 chars)
- N1: {số} bài (3 câu/bài, ~1000–1150 chars)

⛔ CHỈ N1 và N3. Không gen N2/N4/N5 (kind này không tồn tại ở các level đó).

Lưu CSV vào sheets/samples_v1.csv. HTML lưu vào assets/html/doan_van_dai/{LEVEL}_{uuid}.html.
Làm đúng theo SKILL.md — từng bài một, đọc rules/ trước khi gen.

⛔ Q COUNT BẮT BUỘC:
- N1 = 3 câu (fill question_{1,2,3}, slot 4–5 empty)
- N3 = 4 câu (fill question_{1,2,3,4}, slot 5 empty)

⛔ COVERAGE RULE: mỗi câu test đoạn/ý KHÁC NHAU. Marker ①②③ trong HTML khớp câu hỏi tương ứng. Câu cuối nên test thesis/main idea (label `question_author_opinion` hoặc `question_content_match`).

⛔ ĐA DẠNG — BẮT BUỘC:
1. Đọc rules/rule_doc_hieu.md (rule giáo viên — section 3-5 áp dụng trực tiếp) + rules/content.md (chủ đề + char range) + rules/questions.md (label combo).
2. Scan sheets/samples_v1.csv xem topic + label combo đã dùng.
3. Trong cùng level: KHÔNG trùng topic; mỗi bài dùng ≥ 2 question_label khác nhau.
4. Tag **tiếng Anh** từ cột `en` của `rules/topic.json` (vd: philosophy, science, economics). TUYỆT ĐỐI không tiếng Việt/Nhật.

⛔ FURIGANA — chỉ cho từ VƯỢT level. Cấm dạng "Ab". Tra rules/jlpt_kanji.csv.

⛔ SOURCE LINE: N1 có thể có (tự chế tên tác giả/báo, KHÔNG dùng tên thật như 朝日/読売/村上春樹). N3 thường KHÔNG có.

Sau khi gen xong mỗi bài, tự QC checklist trong SKILL.md (HTML + CSV + multi-question coverage → log PASS/FAIL). 1 FAIL = sửa → QC lại. Tất cả PASS mới sang bài tiếp.
Điền Q&A bằng scripts/fill_qa.py (KHÔNG sửa CSV bằng tay).
Sửa HTML = chạy lại process_html.py --refresh.
Verify cuối: python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_dai
```

---

## Prompt có thêm ràng buộc (khi cần kiểm soát chất lượng)

```
Đọc .claude/skills/jlpt-reading-long-passage/SKILL.md rồi gen bài đọc hiểu đoạn văn dài:
- N3: {số} bài | N1: {số} bài

⛔ CHỈ N1 và N3. KHÔNG gen N2/N4/N5.

Lưu CSV vào sheets/samples_v1.csv. HTML lưu vào assets/html/doan_van_dai/{LEVEL}_{uuid}.html.
Trước khi gen:
1. Đọc rules/rule_doc_hieu.md (rule giáo viên — source-of-truth cho vocab/grammar/distractor)
2. Đọc rules/content.md + rules/vocabulary.md + rules/technical.md + rules/questions.md
3. Đọc rules/jlpt_kanji.csv để tra level kanji
4. Đọc 1-2 sample: scripts/load_references.py --level {N1|N3} --count 2
5. Scan sheets/samples_v1.csv xem topic + label combo nào đã dùng

⛔ Q COUNT BẮT BUỘC: N1 = 3 câu, N3 = 4 câu. Slot dư phải empty.

⛔ MULTI-QUESTION COVERAGE:
- Mỗi câu test đoạn/ý KHÁC NHAU (không 2 câu trùng đoạn)
- Marker ①②③ khớp câu hỏi reference. Không marker dư
- Câu cuối nên là `question_author_opinion` hoặc `question_content_match` (test thesis tổng thể, KHÔNG có marker)

⛔ ĐA DẠNG CHỦ ĐỀ + LABEL:
- Trong cùng level: KHÔNG trùng topic; mỗi bài ≥ 2 question_label khác nhau
- Cross-level: ưu tiên topic chưa xuất hiện
- Tag **tiếng Anh** từ cột `en` của `rules/topic.json` (TUYỆT ĐỐI không tiếng Việt/Nhật)

⛔ FURIGANA ZERO-TOLERANCE:
- Kanji vượt level PHẢI có <ruby><rt>
- Cấm dạng "Ab". Chọn 1 trong 2: full kanji + furigana HOẶC full hiragana
- N1 ưu tiên ít ruby (1-3 cái). N3 nhiều hơn vì có nhiều kanji vượt level

⛔ SOURCE LINE — chỉ N1, tên tác giả TỰ CHẾ:
- KHÔNG dùng tên thật (朝日/読売/毎日/日経/村上春樹/夏目漱石)
- Format: （著者名「タイトル」による） hoặc tương tự

⛔ ANNOTATION (注): N1 nên có 1-2 chú thích cho thuật ngữ. Giải thích bằng tiếng Nhật đơn giản, KHÔNG tiếng Anh/Việt.

⛔ ĐÁP ÁN — 4 options newline-separated, KHÔNG prefix "1.", "①", "1)".

Yêu cầu chất lượng câu hỏi:
- Question_label dùng prefix `question_`. ≥ 2 unique labels per bài
- Distractor đa dạng ≥ 3 loại bẫy. Mỗi distractor PHẢI dùng info thật từ bài
- Paraphrase: đáp án đúng KHÔNG copy nguyên văn ≥ 4 từ liên tiếp (N1) / 5 từ (N3)
- Self-solve verify: tự giải từng câu, KHỚP correct_answer_i
- Explanation 3 phần (VN + EN)

Sau khi gen xong mỗi bài, BẮT BUỘC tự QC theo checklist trong SKILL.md:
- Phần A HTML + B Content + C Questions/Answers + D Multi-Q Coverage
- 1 FAIL = sửa → refresh CSV (nếu sửa HTML) → QC lại

Lưu ý kỹ thuật:
- Điền Q&A bằng scripts/fill_qa.py (KHÔNG edit CSV tay)
- Refresh CSV sau sửa HTML: process_html.py --refresh
- Verify cuối: process_html.py --validate --html-dir assets/html/doan_van_dai
```
