---
name: jlpt-doan-van-dai
description: >
  Generate JLPT "đoạn văn dài" (long-passage / 長文読解) reading comprehension passages as
  styled HTML files and output CSV training data for AI fine-tuning. Each passage is a long
  Japanese prose text (550–1150 characters depending on level) testing deep comprehension of
  outline, logical development, author's ideas, and reference phrases via 3-4 multiple-choice
  questions per passage. **CHỈ áp dụng N1 (3 câu, ~1000 chars) và N3 (4 câu, ~550 chars)** —
  N2/N4/N5 KHÔNG có dạng này.
  Skill này bao gồm TOÀN BỘ luồng: gen → QC loop (checklist PASS/FAIL) → sửa. Gen từng bài
  một, kiểm tra đến khi đạt chất lượng mới chuyển sang bài tiếp theo. Output chỉ gồm HTML +
  CSV (không có screenshot PNG).
  Skill này chỉ dành riêng cho dạng "đoạn văn dài" (長文読解).
  Use this skill whenever the user wants to: gen bài đoạn văn dài, tạo nội dung đoạn văn
  dài, generate long-passage reading comprehension, create JLPT 長文 passages, produce AI
  fine-tuning data for the đoạn văn dài section of JLPT N1/N3, kiểm tra chất lượng,
  quality check, review bài, QC.
  Also trigger when the user mentions: gen bài đoạn văn dài, tạo long passage, generate
  JLPT 長文読解, long reading passage N1 / N3.
---

# JLPT 長文 / Đoạn Văn Dài — Workflow

> **Nguyên tắc cốt lõi:**
> 1. **Chỉ N1 và N3** — N2/N4/N5 KHÔNG có dạng này (per `rules/question_format.json`)
> 2. **Gen từng bài một** — không batch rồi QC sau
> 3. **Multi-question** — mỗi bài có 3-4 câu hỏi phủ các đoạn/ý KHÁC NHAU (N1=3, N3=4)
> 4. **Agent tự QC** — đọc lại bài + toàn bộ câu hỏi, tự đánh giá từng mục, log PASS/FAIL
> 5. **1 FAIL = chưa xong** — sửa → QC lại → lặp đến khi ALL PASS
> 6. **KHÔNG có screenshot** — đoạn văn dài không cần PNG

## Cấu trúc file

| File | Nội dung | Đọc khi |
|------|----------|---------|
| `SKILL.md` (file này) | Workflow + QC Checklist | Luôn đọc đầu tiên |
| `rules/content.md` | R1 chủ đề N1/N3 + R2 layout/char counts/Q count/paragraph count + R7 formats + R8 visual multi-question | Gen HTML |
| `rules/vocabulary.md` | R3 từ vựng/ngữ pháp + R4 furigana (N1/N3) | Gen HTML + QC |
| `rules/questions.md` | R5 câu hỏi 3-4 câu + R6 đáp án/6 bẫy + coverage rule | Gen Q&A + QC |
| `rules/technical.md` | R9 HTML template 780px + R10 clean HTML + R11 CSV multi-question | Gen HTML + CSV |
| `references/html-patterns.md` | Template chi tiết per level + marker strategy 4 patterns | Tra cứu khi gen HTML |
| `references/sample-analysis.md` | Phân tích định lượng data mẫu N1/N3 | Hiểu tần suất pattern |
| `scripts/process_html.py` | Xử lý HTML → CSV + count + validate + multi-question support | Gen CSV + QC |
| `scripts/fill_qa.py` | Điền Q&A vào CSV (quote an toàn, 3-4 câu) | Sau khi gen Q&A |
| `scripts/load_references.py` | Load sample JSON để calibrate | BƯỚC 0 chuẩn bị |

## Outputs Per Passage

1. **Styled HTML** → `assets/html/doan_van_dai/{LEVEL}_{uuid}.html`
2. **Clean HTML** → CSV column `text_read` (no attributes, no `<rt>` content)
3. **CSV row** → `sheets/samples_v1.csv` với 3-4 câu hỏi populate (slot còn lại empty)

**KHÔNG có screenshot PNG.** CSV column `general_image` luôn `""`.

## Scope — CHỈ N1 và N3

| Level | Có "đoạn văn dài"? | Q Count | Char Range | Focus spec |
|-------|--------------------|---------|------------|------------|
| **N1** | ✅ có | **3** | **1000–1150** | outline / author's ideas |
| N2    | ❌ KHÔNG có        | —       | —          | — |
| **N3** | ✅ có | **4** | **550–700**   | summary / logical development |
| N4    | ❌ KHÔNG có        | —       | —          | — |
| N5    | ❌ KHÔNG có        | —       | —          | — |

> **⛔ KHÔNG gen N2, N4, N5 cho dạng đoạn văn dài** — sẽ bị reject bởi spec.

## Số câu hỏi per level (BẮT BUỘC)

| Level | Q Count | Slots populate | Slots empty | Câu cuối cần |
|-------|---------|----------------|-------------|--------------|
| **N1** | 3       | `question_{1,2,3}` | `question_{4,5}` | **`question_author_opinion`** (outline) |
| **N3** | 4       | `question_{1,2,3,4}` | `question_{5}` | **`question_content_match`** hoặc **`question_author_opinion`** (tổng thể) |

> **⛔ COVERAGE RULE**: 3-4 câu hỏi trong 1 bài PHẢI test các đoạn/ý KHÁC NHAU, không được trùng phủ 1 đoạn.
> **⛔ LABEL DIVERSITY**: ≥ 2 label khác nhau trong 1 bài, lý tưởng ≥ 3.
> **⛔ CÂU CUỐI RULE**: N1 câu 3 = `question_author_opinion`. N3 câu 4 = `question_content_match` hoặc `question_author_opinion` (test tổng thể).

---

# WORKFLOW

## BƯỚC 0: CHUẨN BỊ (1 lần cho batch)

1. **Đọc `rules/rule_doc_hieu.md`** — **Bộ Tiêu Chí Đánh Giá Đọc Hiểu JLPT toàn diện** từ giáo viên (source-of-truth, 11 phần: 4 tiêu chí, 程度 ±, 書き下ろし/による, ①② 下線 注, **文体の統一 (thể chia)**, furigana per level, 8 loại câu hỏi, 5 loại bẫy chuẩn (+ Single-side cho 統合理解), tiêu chí chi tiết per level).
   **Phần áp dụng trực tiếp cho dạng đoạn văn dài (長文)** — CHỈ N1 và N3:
   - Phần 1 (Tổng quan & Nguyên tắc 程度) — biên ± per level; 書き下ろし bắt buộc N3, có thể trích nguồn N1
   - Phần 2 (Hình thức) — ①②③ thường có; 注 bắt buộc khi có thuật ngữ
   - **Phần 2.4 (Thể chia nhất quán 文体の統一)** — N1/N3 dùng **普通形** (だ・である); văn bản + câu hỏi + 4 đáp án phải **thống nhất thể chia** (long-passage chỉ N1/N3 → toàn bộ 普通形).
   - Phần 3 (Furigana) — bảng quy tắc per level
   - Phần 4 (8 loại câu hỏi) — đặc biệt reference (×1-2), reason_explanation, author_opinion (N1), content_match/meaning_interpretation
   - Phần 5 (5 loại bẫy chuẩn)
   - **Phần 8.3 (N3 長文 ~550字, 4 câu)**, **Phần 10.3 (N1 長文 ~1000字, 3 câu)** — tiêu chí chi tiết 4 chiều
   - Phần 11 (Bảng so sánh tổng hợp) — tra cứu nhanh.
2. **Đọc rules skill**: `rules/content.md` + `rules/vocabulary.md` + `rules/technical.md` + `rules/questions.md`
3. **Đọc `rules/kanji_jlpt_sensei.csv`** — dùng để tra level từng kanji khi quyết định furigana
4. **Scan `sheets/samples_v1.csv` và `data/doan_van_dai_n{1,3}_clean.json`** — xem format, topic đã dùng → chọn format chưa/ít dùng
5. **Load 2-3 sample calibrate style**:
   ```bash
   python3 .claude/skills/jlpt-reading-long-passage/scripts/load_references.py --level N1 --count 2
   python3 .claude/skills/jlpt-reading-long-passage/scripts/load_references.py --level N3 --count 2
   ```
6. **Lập kế hoạch batch**: mỗi bài gán format + topic + combo question_label khác nhau theo distribution của level. Topic chọn **tiếng Anh** từ cột `en` của `rules/topic.json` (đa dạng ≥ 2 category trong batch > 3 bài).

---

## BƯỚC 1→5: LẶP CHO TỪNG BÀI

### BƯỚC 1: GEN HTML + CÁC CÂU HỎI

> Đọc: `rules/content.md` + `rules/vocabulary.md` + `rules/technical.md` + `rules/questions.md`
> Tham khảo: `references/html-patterns.md` cho template per level + 4 marker strategy patterns

1. **Gen `_id`** = `{LEVEL}_{uuid.uuid4().hex}` (full 32-char hex). Level ∈ {N1, N3}.
2. **Chọn format** từ R7 (`rules/content.md`) — 5 formats: essay/tiểu luận (N1), bài báo/thư dài/anecdote (N3).
3. **Chọn `tag`** (topic) — **tiếng Anh** từ cột `en` của `rules/topic.json`, đa dạng trong batch
4. **Chọn combo `question_label`** theo level (R5):
   - **N1** (3 câu): Combo 1 `reference(①) + reason + author_opinion` (phổ biến nhất)
   - **N1** khác: Combo 2 `meaning + reason + author_opinion` | Combo 3 `reference(①) + reference(②) + author_opinion`
   - **N3** (4 câu): Combo 1 `reference(①) + reference(②) + reason + content_match` (phổ biến nhất)
   - **N3** khác: Combo 2/3/4 — xem `rules/questions.md` R5
   - **Câu cuối**: N1 = `author_opinion`; N3 = `content_match`/`author_opinion`
5. **Gen HTML** theo rules → save `assets/html/doan_van_dai/{LEVEL}_{uuid}.html`
   - Container `max-width: 780px; margin: 0 auto; padding: 56px 64px; line-height: 1.95`
   - `word-break: keep-all` (đảm bảo xuống dòng sạch ở ranh giới từ)
   - `<p>` thuần, KHÔNG `<br>` giữa câu (đoạn văn dài KHÔNG có exception)
   - **Paragraph count**: N1 = 5-8, N3 = 4-6
   - Marker ①②③ khớp với các câu hỏi `question_reference`
   - Furigana chỉ cho từ vượt level (tra `rules/kanji_jlpt_sensei.csv`) — data 0% nên giữ ít
   - **Source line**: N1 rất nên có (data 70%), N3 optional (data 38%)
   - **Annotation 注**: N3 rất nên có (data 64%, 2-3 cái), N1 optional (25%, 1-2 cái)
6. **Gen 3-4 câu hỏi + 4 đáp án mỗi câu** theo `rules/questions.md`:
   - 4 options ngăn cách `\n`, KHÔNG số thứ tự `1.`, `①`, `1)`
   - `correct_answer_i` = integer 1-4
   - Mỗi distractor PHẢI dùng info/ý THẬT từ bài (1 trong 6 loại bẫy: Reversal/Detail swap/Scope/Misinterpretation/Part of truth/Over-generalization)
   - Paraphrase: đáp án đúng KHÔNG copy > 4 từ (N1) hoặc > 5 từ (N3) liên tiếp
   - Giải thích `explain_vn_i` + `explain_en_i` theo format 3 phần
7. **Tạo CSV row** bằng `process_html.py` hoặc `fill_qa.py` (⚠️ **dùng script, KHÔNG sửa CSV tay**):
   ```bash
   # Recommended cho 3-4 câu: JSON
   python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
     --file assets/html/doan_van_dai/{LEVEL}_{uuid}.html \
     --csv sheets/samples_v1.csv \
     --tag "{topic_vn}" \
     --questions-json /tmp/qs.json
   ```
   Hoặc dùng `fill_qa.py` với flags per-question:
   > **⛔ KHÔNG ĐƯỢC sửa CSV bằng tay. Commas + newlines trong nội dung (`100,000円`, `A\nB\nC\nD`) sẽ làm vỡ cột.**
   ```bash
   # Ví dụ N1 (3 câu)
   python3 .claude/skills/jlpt-reading-long-passage/scripts/fill_qa.py \
     --csv sheets/samples_v1.csv --row-id {LEVEL}_{uuid} --level N1 \
     --q1-label question_reference --q1 "..." --a1 "A\nB\nC\nD" --ca1 2 --evn1 "..." --een1 "..." \
     --q2-label question_reason_explanation --q2 "..." --a2 "A\nB\nC\nD" --ca2 3 --evn2 "..." --een2 "..." \
     --q3-label question_author_opinion --q3 "..." --a3 "A\nB\nC\nD" --ca3 1 --evn3 "..." --een3 "..."
   ```

---

### BƯỚC 2: ⛔ QC — AGENT TỰ ĐÁNH GIÁ CHECKLIST

> **ĐÂY LÀ BƯỚC QUAN TRỌNG NHẤT. KHÔNG ĐƯỢC BỎ QUA.**
>
> Agent phải **đọc lại** file HTML vừa gen + **TOÀN BỘ** câu hỏi/đáp án trong CSV,
> rồi **tự đánh giá từng mục** bên dưới. Log kết quả theo format:
>
> ```
> QC: {_id}  |  Level: N1  |  Q count: 3/3  |  Labels: [reference, reason, author_opinion]
> ────────────────────────────────
> [ 1] ✅ PASS — Char count (1087 chars, range 1000-1150)
> [ 2] ❌ FAIL — Flow text (found 2x 。<br>)
> [ 3] ✅ PASS — Container CSS (780px, margin auto)
> ...
> ────────────────────────────────
> ⚠️ 1 FAIL → sửa rồi QC lại
> ```
>
> **⛔ KHÔNG ĐƯỢC tự PASS mà không đọc lại nội dung. Phải confirm từng mục cho TẤT CẢ câu hỏi.**

---

### BƯỚC 3: ⛔ CHECKLIST — TẤT CẢ PHẢI PASS

> **Quy tắc: 1 FAIL = chưa xong. Sửa → QC lại từ đầu → lặp đến khi ALL PASS.**
> **Tổng: 32 checks ở 4 phần (A HTML 11, B content 6, C questions/answers + C2 verify 12, D multi-question coverage 3).**

#### PHẦN A: HTML (11 checks)

Agent đọc lại file HTML và kiểm tra:

| # | Check | Cách verify | PASS nếu |
|---|-------|-------------|----------|
| 1 | **Scope level** | Đọc filename `{LEVEL}_...` | Level ∈ {N1, N3}. **N2/N4/N5 = FAIL ngay, không apply** |
| 2 | **Char count** | Chạy `process_html.py --count-only --file ...` | Trong Target Range: N1 1000-1150, N3 550-700 |
| 3 | **Không Hard Reject** | So với Hard Reject threshold | ≥ N1:900, N3:500 |
| 4 | **Flow text** | Tìm `。<br>` trong HTML | Không có `。<br>` nào (đoạn văn dài KHÔNG có exception) |
| 5 | **Container CSS** | Xem CSS | `max-width: 780px`, `margin: 0 auto`, `padding: 56px 64px`, `line-height: 1.95`, `word-break: keep-all` |
| 6 | **`.passage` div** | Xem HTML structure | Có `<div class="passage">` bọc nội dung |
| 7 | **White background** | Xem CSS | `.passage` có `background: white`, body `#f9fafb` |
| 8 | **Paragraph count** | Đếm `<p>` trong `.passage` | N1: 5-8 đoạn. N3: 4-6 đoạn |
| 9 | **Furigana format** | Tìm ngoặc `漢字(かんじ)` hoặc `漢字【かんじ】` | Không có — tất cả furigana dùng `<ruby><rt>` |
| 10 | **Ruby có `<rt>` không rỗng** | Xem mọi `<ruby>...</ruby>` | Tất cả đều có `<rt>` chứa furigana **không rỗng** (vd `<ruby>褒<rt>ほ</rt></ruby>める`). CẤM `<ruby>褒</ruby>` (thiếu rt) hoặc `<ruby>褒<rt></rt></ruby>` (rt rỗng). Auto-check: `process_html.py --validate` |
| 11 | **Ruby count + Visual đúng level** | Đếm `<ruby>` + xem marker/annotation/source | Ruby: N1 ≤ 6, N3 ≤ 15. Visual phù hợp: N1 source nên có (70%), N3 annotation nên có (64%), marker khớp với câu reference |

#### PHẦN B: NỘI DUNG & TỪ VỰNG (6 checks)

| # | Check | Cách verify | PASS nếu |
|----|-------|-------------|----------|
| 12 | **Chủ đề đúng level** | Đọc nội dung, đối chiếu R1 | N1: tiểu luận/phê bình triết học/văn hóa. N3: essay đời sống/bài báo/thư dài/anecdote |
| 13 | **Format đúng level** | Đối chiếu R7 | N1: essay triết học/tiểu luận văn hóa. N3: essay/thư dài/anecdote |
| 14 | **Nội dung logic + đủ depth cho 3-4 câu hỏi** | Đọc toàn bài | Ý nhất quán, đủ nội dung phủ 3-4 câu hỏi khác nhau, có intro + body + (counter) + conclusion |
| 15 | **Không mơ hồ (test 2 cách hiểu)** | Đọc từng câu, thử hiểu theo cách 2 | Chỉ có DUY NHẤT 1 cách hiểu hợp lý cho từng câu hỏi |
| 16 | **Từ vựng đúng level** | Đọc từng từ, đối chiếu R3 | Key terms ≤ level, không dùng ngữ pháp vượt level |
| 17 | **Furigana đúng từ (tra CSV)** | Tra từng kanji trong `rules/kanji_jlpt_sensei.csv` | Mọi từ có kanji vượt level đều có `<ruby><rt>`. Không thừa furigana cho từ đúng level. Không dạng "Ab" (構ちく) |

#### PHẦN C: CÂU HỎI & ĐÁP ÁN (11 checks — áp dụng cho TỪNG câu hỏi)

Agent đọc TOÀN BỘ câu hỏi + 4 đáp án từ CSV và đánh giá từng câu (Q1..Q4):

| # | Check | Cách verify | PASS nếu |
|----|-------|-------------|----------|
| 18 | **Số câu hỏi đúng level** | Xem CSV đếm slot đã fill | N1: Q1+Q2+Q3 có content, Q4+Q5 empty. N3: Q1+Q2+Q3+Q4 có content, Q5 empty |
| 19 | **question_label đúng intent (mỗi câu)** | Đối chiếu R5 với nội dung từng câu | Label có `question_` prefix, khớp với dạng câu hỏi thực tế |
| 20 | **≥ 2 label khác nhau trong bài** | Đếm unique labels | ≥ 2 labels khác nhau (lý tưởng ≥ 3) |
| 21 | **Câu cuối đúng spec level** | Xem `question_label_{n}` | N1 câu 3 = `question_author_opinion`. N3 câu 4 = `question_content_match` hoặc `question_author_opinion` |
| 22 | **Marker khớp câu hỏi (mỗi câu)** | So marker trong HTML với câu hỏi | Mọi `question_reference` có `<u>` + marker ①/②/③ trong HTML tương ứng |
| 23 | **Answer format (mỗi câu)** | Xem 4 đáp án trong CSV | Đúng 4 options ngăn cách `\n`, KHÔNG `1.`/`①`/`1)` prefix. Độ dài tương đương (ratio < 2.0) |
| 24 | **correct_answer (mỗi câu + batch)** | Xem giá trị `correct_answer_i` | Integer 1-4. Scan batch: không lặp cùng vị trí ≥ 3 bài liên tiếp |
| 25 | **Paraphrase đáp án đúng (mỗi câu)** | So đáp án đúng với bài gốc | KHÔNG trùng cụm ≥ 4 từ liên tiếp (N1) hoặc ≥ 5 từ (N3) |
| 26 | **Distractor đa dạng bẫy (mỗi câu)** | Phân loại 3 distractor | ≥ 3 loại bẫy khác nhau trong (6: Reversal/Detail swap/Scope/Misinterpretation/Part of truth/Over-generalization) |
| 27 | **Distractor có căn cứ trong bài (mỗi câu)** | Với mỗi đáp án sai: trích được câu/vị trí trong bài để bác bỏ | KHÔNG bịa. Mỗi distractor dùng info/concept từ bài nhưng sai ngữ cảnh |
| 28 | **Explanations 3 phần (mỗi câu)** | Đọc `explain_vn_i` + `explain_en_i` | Có đủ 3 phần: đáp án đúng (trích vị trí) + đáp án sai (nêu loại bẫy) + tóm tắt. Cả VN và EN đầy đủ |

#### PHẦN C2: VERIFY ĐÁP ÁN (⛔ QUAN TRỌNG NHẤT) — 2 checks

> **Agent tự giải từng câu từ đầu — KHÔNG nhìn đáp án đã gen.**
> Đây là bước bắt lỗi distractor bịa, câu hỏi ambiguous, sai `correct_answer`.

| # | Check | Cách verify | PASS nếu |
|----|-------|-------------|----------|
| 29 | **Tự giải Q1→Q{n}** | Đọc bài + từng câu hỏi, tự chọn đáp án từ đầu (KHÔNG nhìn `correct_answer_i`) | TẤT CẢ kết quả tự chọn KHỚP với `correct_answer_i` trong CSV |
| 30 | **Distractor self-test (toàn bộ câu)** | Với TỪNG đáp án sai trong TỪNG câu: trích dẫn chính xác câu/vị trí trong bài dùng để bác bỏ | Mọi distractor đều trích được. Không trích được = BỊA → FAIL |

#### PHẦN D: MULTI-QUESTION COVERAGE — 2 checks

> **Đặc biệt quan trọng cho đoạn văn dài**: 3-4 câu hỏi phải test các đoạn/ý khác nhau.

| # | Check | Cách verify | PASS nếu |
|----|-------|-------------|----------|
| 31 | **Mỗi câu hỏi test đoạn/ý KHÁC nhau** | Xác định paragraph/ý mà mỗi câu hỏi chỉ vào | Không 2 câu cùng hỏi 1 paragraph/ý/cụm từ reference. Q1..Q{n} phủ ≥ 3 paragraph khác nhau |
| 32 | **Marker khớp câu hỏi tương ứng** | Với mỗi `①`/`②`/`③` trong HTML, confirm có câu hỏi hỏi về nó | Không có marker dư (không câu hỏi hỏi) và không câu hỏi nào hỏi marker không tồn tại |

---

### BƯỚC 4: SỬA & LẶP LẠI

> **⛔ Khi sửa HTML, CẬP NHẬT CSV — chạy lại `process_html.py --refresh` để cập nhật `text_read`, `jp_char_count` trong CSV.**
>
> **🚨 ĐẶC BIỆT khi sửa `<ruby>` thiếu/rỗng `<rt>`:** Đây là lỗi PHỔ BIẾN — agent hay chỉ sửa HTML mà QUÊN refresh CSV → CSV cột `text_read` vẫn chứa ruby hỏng → AI fine-tuning data BỊ HỎNG.
> Workflow BẮT BUỘC khi sửa ruby:
> 1. Sửa HTML: thay `<ruby>褒</ruby>` → `<ruby>褒<rt>ほ</rt></ruby>める`
> 2. **BẮT BUỘC** chạy: `python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py --refresh --html-dir assets/html/doan_van_dai --csv sheets/samples_v1.csv`
> 3. Verify: `python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_dai --csv sheets/samples_v1.csv` — output PHẢI có dòng `✅ CSV ...: 0 row với broken ruby`. Nếu vẫn báo `🚫 CSV ... có N row với broken ruby` → CSV chưa sync, chạy lại `--refresh`.
>
> Không có screenshot nên KHÔNG cần chạy lại screenshot script.

| Nếu FAIL | Hành động | Sau đó |
|-----------|-----------|--------|
| #1 (scope level) | Level sai → REJECT, không gen lại — đoạn văn dài chỉ có N1/N3 | Không tiếp tục |
| #2, #3 (chars) | Bổ sung/cắt nội dung. Nếu Hard Reject → gen lại hoàn toàn | Chạy `--refresh` → QC lại |
| #4 (flow text) | Sửa `<br>` → `</p><p>` | Chạy `--refresh` → QC lại |
| #5, #6, #7 (CSS/structure) | Sửa CSS/structure HTML (780px, 56px 64px, line-height 1.95) | Chạy `--refresh` → QC lại |
| #8 (paragraph count) | Chia/gộp paragraph đạt N1:5-8, N3:4-6 | Chạy `--refresh` → QC lại |
| #9, #10, #11 (ruby/visual) | Sửa ruby tags hoặc thêm marker/source/annotation theo level | Chạy `--refresh` → QC lại |
| #12-#16 | Gen lại nội dung (giữ _id) | Chạy `--refresh` → QC lại |
| #17 (furigana tra CSV) | Sửa ruby tags (tra lại `rules/kanji_jlpt_sensei.csv`) | Chạy `--refresh` → QC lại |
| #18 (số câu hỏi) | Thêm/xóa câu bằng `fill_qa.py` để đúng số slot (N1=3, N3=4) | QC lại |
| #19, #20 (labels) | Sửa label trong `fill_qa.py` (dùng đủ `question_` prefix + đa dạng) | QC lại |
| #21 (câu cuối label) | Sửa câu cuối = `question_author_opinion` (N1) / `question_content_match`/`question_author_opinion` (N3) | QC lại |
| #22 (marker ko khớp) | Sửa HTML (thêm/bớt marker) hoặc sửa câu hỏi | Chạy `--refresh` nếu sửa HTML → QC lại |
| #23, #24, #25 (đáp án) | Sửa đáp án bằng `fill_qa.py` | QC lại |
| #26 (distractor bẫy) | Viết lại distractor dùng 1 trong 6 loại bẫy | QC lại |
| #27 (distractor bịa) | Viết lại distractor dùng info thật từ bài | QC lại |
| #28 (explanation) | Viết lại explain 3 phần đầy đủ cho từng câu | QC lại |
| #29, #30 (self-solve) | Đáp án có thể sai → xem lại bài vs. đáp án | Sửa đáp án hoặc bài. QC lại |
| #31 (coverage trùng) | Sửa các câu để hỏi đoạn khác nhau | QC lại |
| #32 (marker dư/thiếu) | Thêm/bớt marker trong HTML hoặc sửa câu hỏi | Chạy `--refresh` → QC lại |

**Lệnh refresh CSV sau khi sửa HTML:**
```bash
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
  --refresh \
  --file assets/html/doan_van_dai/{LEVEL}_{uuid}.html \
  --csv sheets/samples_v1.csv
```

> **Vòng lặp: sửa → refresh CSV (nếu sửa HTML) → quay lại BƯỚC 2 (QC lại TẤT CẢ) → nếu còn FAIL thì lặp lại.**
> **Tối đa 5 vòng. Sau 5 vòng vẫn FAIL → báo lỗi cho user, KHÔNG bỏ qua.**

---

### BƯỚC 5: ✅ HOÀN THÀNH → BÀI TIẾP THEO

Chỉ khi **TẤT CẢ 32 checks PASS** → log:
```
🎉 ALL PASSED (32/32) — {_id} hoàn thành — {n} câu hỏi ({labels})
```
→ Chuyển sang bài tiếp theo (quay lại BƯỚC 1).

**Batch size**: 3-5 bài/lần (bài dài + nhiều câu hỏi, workload cao nhất trong series).

---

## BƯỚC CUỐI: VERIFY BATCH (sau khi gen xong TẤT CẢ bài)

Sau khi hoàn thành toàn bộ batch, chạy verify toàn bộ:

```bash
# 1. Validate tất cả file HTML (char count + broken ruby)
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
  --validate --html-dir assets/html/doan_van_dai

# 2. Đếm số rows trong CSV + check số câu hỏi + last-label rule
python3 -c "
import csv
expected_q = {'N1': 3, 'N3': 4}
last_rules = {'N1': ['question_author_opinion'],
              'N3': ['question_content_match', 'question_author_opinion']}
with open('sheets/samples_v1.csv', 'r', encoding='utf-8') as f:
    rows = list(csv.DictReader(f))
print(f'Total rows: {len(rows)}')
bad = 0
for r in rows:
    lv = r.get('level')
    if lv not in expected_q:
        continue
    want = expected_q[lv]
    got = sum(1 for i in range(1, 6) if r.get(f'question_{i}', '').strip())
    if got != want:
        bad += 1
        print(f\"  ❌ {r['_id']} ({lv}): {got} câu, expected {want}\")
        continue
    last_lbl = r.get(f'question_label_{want}', '')
    if last_lbl not in last_rules[lv]:
        bad += 1
        print(f\"  ❌ {r['_id']} ({lv}): câu cuối = {last_lbl}, expected {last_rules[lv]}\")
print(f'Multi-question OK: {len(rows) - bad}/{len(rows)}')
for level in ['N1','N3']:
    n = sum(1 for r in rows if r.get('level') == level)
    print(f'  {level}: {n}')
"
```

### Batch-level checklist

- [ ] Mỗi bài có `_id` unique, đúng format `{LEVEL}_{uuid}`, Level ∈ {N1, N3}
- [ ] `kind` = `đoạn văn dài` trong tất cả rows
- [ ] `level` chỉ là N1 hoặc N3 (KHÔNG có N2/N4/N5)
- [ ] `general_image` = `""` (empty) — KHÔNG có PNG
- [ ] `general_audio` = `""` (empty)
- [ ] Char count trong Target Range (N1:1000-1150, N3:550-700)
- [ ] Không bài nào dưới Hard Reject threshold (N1:900, N3:500)
- [ ] Paragraph count N1:5-8, N3:4-6
- [ ] Furigana chỉ cho từ vượt level, không dạng "Ab", mọi `<ruby>` có `<rt>`
- [ ] Ruby tags count ≤ expected (N1:6, N3:15)
- [ ] **Mỗi bài có đúng số câu hỏi theo level** (N1=3, N3=4) — các slot khác empty
- [ ] **N1 câu 3 = `question_author_opinion`. N3 câu 4 = `question_content_match`/`question_author_opinion`**
- [ ] `question_label_i` có `question_` prefix (7 labels hợp lệ)
- [ ] Trong 1 bài có ≥ 2 label khác nhau (lý tưởng ≥ 3)
- [ ] Mỗi câu hỏi có 4 đáp án ngăn cách `\n` (KHÔNG số thứ tự)
- [ ] `correct_answer_i` phân bố đều 1-4 trong batch
- [ ] Distractor dùng info từ bài (không bịa), ≥ 3 loại bẫy khác nhau per câu
- [ ] `explain_vn_i` + `explain_en_i` đủ 3 phần cho mọi câu
- [ ] Marker trong text khớp câu hỏi (`①` ↔ Q reference)
- [ ] **Multi-question coverage**: 3-4 câu test các đoạn/ý khác nhau trong từng bài
- [ ] `text_read` clean — không attribute, không class, không `<rt>` content
- [ ] `<p>` thuần, không `<br>` giữa câu
- [ ] Annotation (注) giải thích bằng **tiếng Nhật đơn giản**, KHÔNG tiếng Anh/Việt
- [ ] Source line dùng tên tác giả tự chế (KHÔNG dùng tên tác giả thật)
- [ ] Trong batch, tag đa dạng ≥ 2 category (nếu batch > 3)

---

## Reference Data & Samples

Data mẫu có sẵn trong `data/`:

| Level | File | Samples |
|-------|------|---------|
| N1 | `doan_van_dai_n1_clean.json` | 27 |
| N3 | `doan_van_dai_n3_clean.json` | 34 |

N2, N4, N5 **KHÔNG CÓ** file data cho đoạn văn dài — không applicable.

Load bằng:
```bash
# Stats N1/N3
python3 .claude/skills/jlpt-reading-long-passage/scripts/load_references.py --stats

# 2 random samples N1
python3 .claude/skills/jlpt-reading-long-passage/scripts/load_references.py --level N1 --count 2
```

**LƯU Ý khi đọc data gốc**:
- Data gốc DÙNG `<br>` nhiều (N1 29%, N3 52%) — thói quen xấu. Output HTML skill KHÔNG theo.
- Data gốc có `<span>` bọc paragraph — output KHÔNG cần.
- Data gốc N1 thường có 4 câu hỏi (74%) — skill follow spec `EXPECTED_Q_COUNT` (N1=3), không bắt chước data.
- Data gốc N3 thường có 4 câu (82%) — skill follow spec (N3=4) OK.
- Data gốc ruby = 0% — skill giữ ít ruby (0-3 cặp ưu tiên).

Chi tiết phân tích từng level xem `references/sample-analysis.md`.

---

## Common errors (dạng đoạn văn dài hay gặp)

1. **Gen level ngoài N1/N3** — SAI, skill này CHỈ N1/N3
2. **N1 gen 4 câu (bắt chước data)** — SAI, spec N1 = 3 câu
3. **N1 câu cuối là reference/content_match** — SAI, phải là `question_author_opinion`
4. **N3 câu cuối là reference** — SAI, phải là `question_content_match` hoặc `question_author_opinion` (tổng thể)
5. **Bài chỉ 2-3 paragraph** — SAI, N1 cần 5-8, N3 cần 4-6 paragraph
6. **Tất cả câu dùng cùng 1 label** — SAI, cần ≥ 2 labels khác nhau
7. **2-3 câu cùng hỏi 1 đoạn** — SAI, coverage rule yêu cầu đoạn khác nhau
8. **Marker `①` trong bài nhưng không có câu hỏi reference** — marker vô nghĩa
9. **Câu hỏi hỏi `①` nhưng HTML không có marker** — câu hỏi không thể trả lời
10. **Thiếu source line ở N1** — giảm authenticity (data 70%)
11. **N3 không có annotation** — giảm authenticity (data 64%)
12. **Dùng `<br>` giữa câu** — SAI, dùng `<p>` thuần
13. **Ruby quá nhiều (> 6 cho N1, > 15 cho N3)** — đoạn văn dài thực tế gần như không có ruby
14. **Distractor yếu** — đoạn văn dài đòi hỏi distractor sai ở 1 nuance (thời gian, nhân-quả đảo, phạm vi)
15. **Quên `question_` prefix trong label** — label phải là `question_content_match`, không phải `content_match`

---

## Cảnh báo bảo mật dữ liệu

> **🚫 KHÔNG ĐƯỢC GHI VÀO THƯ MỤC `rules/`** — `rules/question_sheet.csv`, `rules/topic.json`, `rules/kanji_jlpt_sensei.csv`, `rules/question_format.json`, `rules/mission.json`, `rules/rule_doc_hieu.md` là file tham chiếu, chỉ đọc. Mọi dữ liệu gen phải ghi vào `sheets/samples_v1.csv`.
