# PROMPTS.md — Đoạn Văn Dài

Prompt templates để gọi LLM gen content cho skill `jlpt-reading-long-passage` (長文読解).

**Scope chỉ 2 level**: N1 (3 câu/bài, ~1000-1150 ký tự) và N3 (4 câu/bài, ~550-700 ký tự). N2/N4/N5 KHÔNG có kind này — đừng gen.

Workload cao nhất của series (bài dài + nhiều câu + distractor tinh vi). Batch size khuyến nghị: **3-5 bài/lần**.

Cách dùng: copy prompt theo level, thay các `{PLACEHOLDER}` bằng giá trị thực tế, feed vào Claude/Gemini.

## 0. Common System Prompt (cả N1 và N3)

```
Bạn là trợ lý chuyên gen dữ liệu JLPT 長文読解 (đoạn văn dài).

Rule BẤT BIẾN:
1. Gen đoạn văn tiếng Nhật đúng Target Range:
   - N1: 1000-1150 ký tự
   - N3:  550- 700 ký tự
   (Đếm không whitespace, không tính <rt>, <style>, <script>.)
2. CHỈ gen cho N1 hoặc N3. N2/N4/N5 không có kind này.
3. HTML template: <!DOCTYPE html>, Noto Sans JP qua Google Fonts,
   <div class="passage"> max-width 780px, chứa <p> với text-indent 1em.
   KHÔNG dùng <br> giữa câu. Dùng <p> thuần.
4. Furigana RẤT HIẾM:
   - N1: tối đa 3 cặp <ruby>/<rt> (data 0%)
   - N3: tối đa 6 cặp (data 0%)
   Chỉ cho từ vượt level. Cấm dạng "Ab" (nửa kanji nửa hiragana).
5. Mỗi bài có số câu hỏi CHÍNH XÁC:
   - N1: 3 câu
   - N3: 4 câu
6. Trong 1 bài, ≥ 2 question_label khác nhau (N3 khuyến nghị ≥ 3 labels).
7. Các câu hỏi trong 1 bài test các ĐOẠN/CỤM TỪ KHÁC NHAU.
   Bài dài có nhiều đoạn → PHẢI tách câu hỏi ra khắp bài.
8. CÂU CUỐI của bài:
   - N1 câu 3: NÊN là question_author_opinion (outline/main idea)
   - N3 câu 4: NÊN là question_content_match hoặc question_author_opinion
9. Nếu dùng marker ①②③④ cho câu reference, MARKER phải xuất hiện trong HTML
   ngay trước cụm từ được hỏi, và câu hỏi dẫn bằng「①cụm từ」とあるが.
10. Distractor plausible, không có 2 đáp án cùng đúng. Đáp án answer được
    từ trong bài (không suy luận ngoài bài).
11. Giải thích VN + EN cho TỪNG CÂU (tại sao đáp án đúng + tại sao 3 đáp án
    kia sai).
12. Tên tác giả trong source line PHẢI tự chế, KHÔNG dùng tác giả có thật.
13. Paragraph count:
    - N1: 5-8 paragraph
    - N3: 4-6 paragraph

Output format: JSON với các field:
{
  "id": "{LEVEL}_{uuid32hex}",
  "level": "N1" hoặc "N3",
  "tag": "triết học ngôn ngữ",
  "html": "<!DOCTYPE html>...",
  "questions": [
    {
      "label": "question_reference",
      "question": "①「...」とあるが、何を指すか。",
      "answers": ["A option", "B option", "C option", "D option"],
      "correct": 2,
      "explain_vn": "...",
      "explain_en": "..."
    },
    ...3 hoặc 4 phần tử tuỳ level...
  ]
}
```

## 1. Prompt N1 — Tiểu luận/phê bình, 3 câu, câu cuối = author_opinion

```
Gen 1 bài JLPT N1 đoạn văn dài về chủ đề {TOPIC} (ví dụ: triết học ngôn ngữ,
phê bình văn hóa sâu, phân tích hệ thống giáo dục/xã hội, tùy bút cao cấp,
phê bình văn học, khoa học nhận thức, xã hội học).

Yêu cầu cụ thể:
- Độ dài: 1000-1150 ký tự (count không whitespace, không <rt>)
- Văn phong: rất formal, văn bản luận thuyết
  (～いかんによらず, ～をもって, ～に先立ち, ～をものともせず, ～にほかならない,
   ～というものだ, ～ざるを得ない, ～に足る, ～にたえない)
- Cấu trúc: 5-8 paragraph (KHÔNG dùng <br>)
- Furigana: tối đa 3 cặp <ruby>/<rt>, chỉ cho từ vượt N1 (ví dụ: 曖昧<rt>あいまい</rt>)
- Annotation 注 khuyến nghị (data 25%): 1-2 chú thích dưới <div class="annotations">
- Source line RẤT NÊN có (data 70%): `（{tên tác giả tự chế}「{tên sách}」による）`
- Underline <u> nhiều (data 85%) — dùng cho MỌI cụm từ được hỏi
- Marker ①②③ (data 29%) — dùng cho 1-2 câu reference

BẮT BUỘC: 3 câu hỏi với ≥ 2 labels khác nhau.

Combo câu hỏi đề xuất (chọn 1):
  A. (phổ biến nhất)
     Q1: reference (marker ①) — test 1 đoạn cụ thể
     Q2: reason_explanation — test logic chain
     Q3: author_opinion — tổng kết luận điểm chính
  B. (essay analytic)
     Q1: meaning_interpretation — test hiểu nuance
     Q2: reason_explanation
     Q3: author_opinion
  C. (reference heavy)
     Q1: reference (①)
     Q2: reference (②)
     Q3: author_opinion

**Rule cứng**: câu 3 cuối cùng LUÔN là question_author_opinion.
Bài dài N1 focus vào "outline, main idea" — phải có 1 câu tổng kết.

Reference with marker pattern:
  HTML: <p>...<span class="marker">①</span><u>特定の言語的枠組み</u>...</p>
  Question: 「①特定の言語的枠組み」とあるが、どういうことか。
  (label = question_reference hoặc question_meaning_interpretation)

Reason pattern:
  Question: 筆者が...と考えるのはなぜか。
  Question: 筆者が...を問題視する理由は何か。

Author opinion pattern:
  Question: 筆者が最も言いたいことはどれか。
  Question: この文章全体のテーマとして最も適切なものはどれか。
  Question: 筆者の主張として最も適切なものはどれか。

4 đáp án bắt buộc tinh vi — N1 distractor:
  - Paraphrase sai ở 1 keyword
  - Đúng nội dung nhưng chệch mức độ (絶対/しばしば)
  - Trộn 2 ý của bài thành 1 ý sai
  - Ngược polarity (肯定↔否定)

Output JSON theo Common System Prompt, mảng "questions" có 3 phần tử,
questions[2].label = "question_author_opinion" (khuyến nghị mạnh).
```

## 2. Prompt N3 — Thư dài/essay đời sống/giai thoại, 4 câu, câu cuối = content_match

```
Gen 1 bài JLPT N3 đoạn văn dài về chủ đề {TOPIC} (ví dụ: thư dài có nội dung,
essay về đời sống/học tập, giai thoại có ý nghĩa, bài báo có phân tích,
câu chuyện có lesson, thí nghiệm khoa học đơn giản, môi trường, văn hóa ẩm thực).

Yêu cầu cụ thể:
- Độ dài: 550-700 ký tự
- Văn phong: nửa formal nửa conversational, văn viết trung cấp
  (～について, ～によって, ～場合は, ～ために, ～ということだ, ～ものだ, ～のに,
   ～ように, ～そうだ, ～とおり)
- Cấu trúc: 4-6 paragraph (KHÔNG <br>; dùng <p> thuần)
- Furigana: tối đa 6 cặp, cho từ N2+
- Annotation 注 RẤT NÊN có (data 64%): 2-3 chú thích dưới <div class="annotations">
  (ví dụ: 注1 都合：... / 注2 ～というわけで：...)
- Source line optional (data 38%) — có thể thêm nếu topic là essay/bài báo
- Marker ①②③④ RẤT PHỔ BIẾN (data 79%) — với 4 câu, nên có ≥ 2 marker
- Underline <u> cho mọi cụm từ được hỏi (data 50%)

BẮT BUỘC: 4 câu hỏi với ≥ 2 labels khác nhau (khuyến nghị ≥ 3 labels).

Combo câu hỏi đề xuất (chọn 1):
  A. (balanced, phổ biến nhất)
     Q1: reference (marker ①)
     Q2: reference (marker ②)
     Q3: reason_explanation
     Q4: content_match (tổng thể bài)
  B. (reason heavy)
     Q1: reference (marker ①)
     Q2: reason_explanation
     Q3: meaning_interpretation
     Q4: content_match
  C. (essay analytic)
     Q1: reference (marker ①)
     Q2: reference (marker ②)
     Q3: reason_explanation
     Q4: author_opinion

**Rule cứng**: câu 4 cuối cùng NÊN là question_content_match hoặc question_author_opinion
(tổng thể bài). 3 câu đầu test từng đoạn/cụm cụ thể.

Multi-marker pattern:
  HTML:
    <p>...<span class="marker">①</span><u>その考え</u>...</p>
    <p>...<span class="marker">②</span><u>このような結果</u>...</p>
  Q1: ①「その考え」とあるが、どのような考えか。
  Q2: ②「このような結果」とあるが、どのような結果か。

Reason pattern:
  Question: 筆者はなぜ...と言っているか。
  Question: ...のはどうしてか。

Content_match (câu cuối tổng thể):
  Question: この文章の内容と合っているものはどれか。
  Question: 本文の内容に合うものはどれか。

Ví dụ câu hỏi N3 phổ biến (theo data):
  - ①「...」とあるが、何を指すか。
  - ②「...」とあるが、どういうことか。
  - 筆者はなぜそう考えているか。
  - 本文の内容と合うものはどれか。

4 đáp án bắt buộc plausible — N3 distractor sai ở 1 nuance:
  - Đảo ngược nhân-quả (vì A nên B ↔ vì B nên A)
  - Nhầm đối tượng (bố ↔ mẹ; học sinh A ↔ học sinh B)
  - Sai thời gian (quá khứ ↔ hiện tại ↔ tương lai)
  - Tuyệt đối hoá (いつも ↔ 時々)
  - Thiếu/thừa điều kiện

Output JSON theo Common System Prompt, mảng "questions" có 4 phần tử,
questions[3].label ∈ {"question_content_match", "question_author_opinion"}.
```

## 3. Batch Prompt — Gen nhiều bài 1 lần

```
Gen {N} bài JLPT {LEVEL} đoạn văn dài. Yêu cầu đa dạng:

1. Topic: chọn từ {N_TOPIC} nhóm khác nhau:
   N1:
   - Triết học ngôn ngữ/tư duy
   - Phê bình văn hóa/xã hội
   - Phân tích hệ thống giáo dục
   - Tùy bút/phê bình văn học
   - Khoa học nhận thức/tâm lý học
   - Xã hội học/kinh tế học
   N3:
   - Thư dài có nội dung (từ/để bạn bè, đồng nghiệp)
   - Essay về học tập/đời sống
   - Giai thoại có lesson
   - Bài báo phân tích trung cấp
   - Thí nghiệm đơn giản
   - Văn hóa ẩm thực/truyền thống
   - Môi trường/sức khỏe

2. Số câu hỏi per bài: BẮT BUỘC đúng spec
   - N1: 3 câu  | N3: 4 câu

3. question_label:
   - Mỗi bài ≥ 2 labels khác nhau (N3 khuyến nghị ≥ 3)
   - N1: câu cuối = question_author_opinion (khuyến nghị mạnh)
   - N3: câu cuối = question_content_match hoặc question_author_opinion
   - Batch-level: ≥ 4 labels khác nhau trong cả batch

4. Mỗi bài có _id riêng: {LEVEL}_{uuid32hex}.

5. Độ dài nằm trong Target Range:
   - N1: 1000-1150
   - N3:  550- 700

6. Các câu hỏi trong cùng bài phải test các ĐOẠN/CỤM TỪ khác nhau.

7. Visual elements:
   - N1: source line 70%+, underline 85%+, annotation 1-2
   - N3: marker ①②③④ bắt buộc ≥ 2, annotation 2-3 (bắt buộc), source line optional

Batch size khuyến nghị: 3-5 bài/lần (workload cao nhất của series).

Output: array của {N} JSON objects theo Common System Prompt.
```

## 4. Fix Prompt — Khi bài fail validate

### Case: UNDER_TARGET (trên HARD_REJECT nhưng dưới TARGET)

```
Bài {ID} có {CHARS} ký tự, dưới Target Range của {LEVEL} ({LO}-{HI}).

Bổ sung thêm {NEEDED} ký tự bằng một trong các cách (theo thứ tự ưu tiên):
1. Thêm 1-2 câu văn vào paragraph giữa (mở rộng ý, không đưa ý mới)
2. Thêm 1 paragraph mới ở giữa bài (nối ý đang phát triển)
3. Thêm 1 chú thích 注1/注2 ở <div class="annotations">
4. Mở rộng chi tiết 1 dẫn chứng/ví dụ đã có

KHÔNG thêm paragraph có ý trái ngược — phải flow nối tiếp.
Giữ nguyên: _id, tất cả questions, đáp án đúng.
Marker ①②③ đã dùng phải giữ nguyên vị trí (nếu không sẽ break câu hỏi reference).

Output: JSON object mới (cùng schema, cùng _id, cùng questions), đã chỉnh độ dài.
```

### Case: OVER_TARGET (> HI + 100)

```
Bài {ID} có {CHARS} ký tự, vượt xa Target Range của {LEVEL} ({LO}-{HI}).

Rút ngắn còn {LO}-{HI} ký tự bằng cách:
1. Loại bỏ 1 paragraph có nội dung tangential (không liên quan trực tiếp đến câu hỏi)
2. Rút gọn câu dài thành 1-2 câu ngắn
3. Bỏ annotation/source line thừa

KHÔNG được xoá đoạn chứa marker ①②③ nếu câu hỏi đang reference marker đó.

Giữ nguyên: _id, tất cả questions, đáp án đúng.

Output: JSON object mới (cùng schema, cùng _id, cùng questions).
```

### Case: HARD_REJECT

```
Bài {ID} có {CHARS} ký tự, DƯỚI Hard Reject của {LEVEL} ({THRESHOLD}).
KHÔNG chỉnh sửa — GEN LẠI TỪ ĐẦU.

Yêu cầu:
  {LEVEL} Target Range {LO}-{HI} ký tự.
  Số câu: {Q_COUNT} (spec).
  Paragraph count: {PARAGRAPH_RANGE}.
  Topic: {TOPIC}.
  Labels đề xuất: {LABEL_COMBO}.
  Câu cuối: {FINAL_LABEL}.

Output: JSON object mới hoàn toàn.
```

### Case: UNSUPPORTED_LEVEL

```
Bài {ID} có level {LEVEL} nhưng kind "đoạn văn dài" CHỈ dành cho N1 và N3.

Xử lý:
1. Nếu nội dung phù hợp N1/N3 → sửa _id thành {N1|N3}_{uuid_mới}, 
   điều chỉnh độ dài + số câu theo level mới.
2. Nếu nội dung không phù hợp (ví dụ quá đơn giản cho N3) → 
   chuyển sang skill khác (jlpt-reading-medium-passage cho N2/N4/N5).

Không commit vào CSV của đoạn văn dài cho đến khi level đã chuẩn hoá.
```

### Case: Sai số câu hỏi (warning)

```
Bài {ID} đang có {ACTUAL_Q} câu hỏi, nhưng {LEVEL} yêu cầu {EXPECTED_Q} câu.

Fix:
- Nếu dư câu: bỏ câu kém nhất (test cùng đoạn với câu khác, hoặc lặp label).
- Nếu thiếu câu: thêm câu test đoạn CHƯA được hỏi, với label khác các câu hiện có.
  Ưu tiên:
    - N1 nếu thiếu câu cuối → thêm question_author_opinion
    - N3 nếu thiếu câu cuối → thêm question_content_match

Giữ nguyên: _id, HTML, đáp án các câu còn lại.

Output: JSON object mới, "questions" có đúng {EXPECTED_Q} phần tử.
```

### Case: Câu cuối không phải author_opinion/content_match (warning)

```
Bài {ID} là {LEVEL} đoạn văn dài, nhưng câu cuối có label = {ACTUAL_LABEL}
thay vì label tổng thể khuyến nghị.

Fix (giữ 3 câu đầu nguyên vẹn):
- N1: chuyển câu 3 thành question_author_opinion, test main idea tổng thể.
  Template: 筆者が最も言いたいことはどれか。 / この文章全体のテーマとして〜
- N3: chuyển câu 4 thành question_content_match, test tổng thể bài.
  Template: この文章の内容と合っているものはどれか。

4 đáp án plausible phải bao phủ main idea bài.

Giữ nguyên: _id, HTML, câu 1-2 (N1) hoặc câu 1-3 (N3).

Output: JSON object mới.
```

### Case: Các câu hỏi cùng test 1 đoạn (warning)

```
Bài {ID} có câu {Q_A} và câu {Q_B} cùng test 1 đoạn/cụm trong bài.

Fix: chuyển 1 trong 2 câu sang test đoạn KHÁC (bài dài có nhiều đoạn).
Ưu tiên: câu label ít nổi bật hơn → chuyển sang test đoạn chưa hỏi.

Giữ nguyên các câu còn lại, _id, HTML.

Output: JSON object mới.
```

### Case: Dạng "Ab"

```
Bài {ID} có dạng "Ab" (nửa kanji nửa hiragana) sai quy tắc furigana.

Cụm vi phạm: "{VIOLATION}" (ví dụ: 週かん, 友だち)

Fix: chọn 1 trong 2 cách:
1. Full kanji + <ruby>: <ruby>週間<rt>しゅうかん</rt></ruby>
2. Full hiragana: しゅうかん

Giữ nguyên phần còn lại, chỉ sửa cụm vi phạm.

Output: JSON object mới (cùng _id, cùng questions, HTML đã fix).
```

### Case: Marker không match câu hỏi

```
Bài {ID} có câu hỏi「{Q_NUM}」tham chiếu marker {MARKER} nhưng HTML không có marker đó.

Fix: thêm marker vào HTML:
  <span class="marker">{MARKER}</span><u>{PHRASE}</u>
ngay trước hoặc bao quanh cụm từ được hỏi.

Hoặc: đổi câu hỏi không dùng marker, chỉ quote cụm từ:
  「{PHRASE}」とあるが、〜

Giữ nguyên các phần còn lại.

Output: JSON object mới.
```

## 5. Quality Check Prompt (gọi sau batch)

```
Kiểm tra chất lượng batch {N} bài JLPT {LEVEL} đoạn văn dài. Cho mỗi bài:

1. Level = N1 hoặc N3:                          PASS / FAIL
2. Độ dài (target {LO}-{HI}):                    PASS / FAIL ({CHARS} chars)
3. Số câu hỏi = {EXPECTED_Q}:                    PASS / FAIL ({ACTUAL_Q})
4. Paragraph count ({PARAGRAPH_RANGE}):          PASS / FAIL
5. ≥ 2 labels khác nhau trong bài:               PASS / FAIL
6. Câu cuối = {FINAL_LABEL}:                     PASS / FAIL (actual: {LAST_LABEL})
7. Các câu hỏi test đoạn KHÁC NHAU:              PASS / FAIL
8. Marker ①②③ match câu hỏi (nếu có):            PASS / FAIL
9. Furigana đúng quy tắc (không "Ab"):           PASS / FAIL
10. Mỗi câu chỉ 1 đáp án đúng:                   PASS / FAIL
11. Distractor plausible level-phù hợp:          PASS / FAIL
12. Câu hỏi answer được từ trong bài:            PASS / FAIL
13. explain_vn + explain_en non-empty:           PASS / FAIL
14. Source line có/N1 khuyến nghị:               PASS / WARN
15. Annotation ≥ 2/N3 khuyến nghị:               PASS / WARN

Batch-level:
- ≥ 4 question_label khác nhau trong batch?      YES / NO
- ≥ 2 tag (topic) khác nhau?                     YES / NO
- Tất cả _id unique?                             YES / NO
- {N} bài, tổng {N}×{EXPECTED_Q} = {TOTAL_Q} câu hỏi — đúng?   YES / NO

Output: bảng markdown 1 row per bài + summary batch-level.
```

## 6. Variables reference

| Placeholder | Giá trị mẫu |
|-------------|-------------|
| `{LEVEL}`   | N1 hoặc N3 (CHỈ 2 level) |
| `{TOPIC}`   | triết học ngôn ngữ, essay học tập, thư dài... |
| `{N}`       | Số bài (khuyến nghị 3-5) |
| `{N_TOPIC}` | Số nhóm topic (thường 2-3) |
| `{LO}, {HI}` | Target Range (N1: 1000-1150, N3: 550-700) |
| `{THRESHOLD}` | Hard Reject (N1: 900, N3: 500) |
| `{CHARS}`   | Char count thực tế |
| `{NEEDED}`  | Số ký tự cần bổ sung |
| `{ID}`      | `{LEVEL}_{uuid32hex}` |
| `{VIOLATION}` | Cụm vi phạm rule furigana |
| `{Q_COUNT}` / `{EXPECTED_Q}` | Số câu hỏi spec (N1: 3, N3: 4) |
| `{ACTUAL_Q}` | Số câu thực tế (để fix warning) |
| `{ACTUAL_LABEL}` | Label thực tế của câu cuối |
| `{LAST_LABEL}` | Label câu cuối cùng của bài |
| `{FINAL_LABEL}` | Label khuyến nghị câu cuối (N1: author_opinion, N3: content_match) |
| `{LABEL_COMBO}` | Combo labels đề xuất (xem section level) |
| `{PARAGRAPH_RANGE}` | N1: 5-8, N3: 4-6 |
| `{Q_NUM}` / `{Q_A}` / `{Q_B}` | 1, 2, 3, 4 (số thứ tự câu hỏi) |
| `{MARKER}` | ①, ②, ③, ④ |
| `{PHRASE}` | Cụm từ được gạch chân |
| `{TOTAL_Q}` | Tổng câu hỏi kỳ vọng cả batch (N×3 hoặc N×4) |
