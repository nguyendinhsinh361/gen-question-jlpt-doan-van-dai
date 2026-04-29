# Rules: Câu hỏi & Đáp án (R5, R6)

> **Scope**: Đoạn văn dài có **3-4 câu hỏi** per bài (N1 = 3 câu, N3 = 4 câu). Mỗi câu hỏi cover MỘT đoạn/ý KHÁC NHAU của bài. Focus spec: N1 = outline/author's ideas; N3 = summary/logical development.

## R5. Câu hỏi (`question_label` + `question`)

### Nguyên tắc cốt lõi

- **100% căn cứ vào văn bản**: đáp án xác nhận rõ ràng bởi thông tin/ý trong bài. KHÔNG dựa kiến thức ngoài.
- **Không dùng ảnh trong câu hỏi** — `question_image_X` luôn empty.
- **3-4 câu hỏi KHÔNG trùng nhau**: mỗi câu test 1 đoạn/ý khác — không được 2 câu cùng hỏi 1 đoạn.
- **Đoạn văn dài = depth cao**: distractor đòi hỏi nuance (thời gian, nhân-quả đảo, phạm vi).

### 7 kiểu câu hỏi (`question_label`)

Labels từ `rules/mission.json` — **BẮT BUỘC dùng `question_` prefix**:

| `question_label`                   | Khi nào dùng | Keywords câu hỏi |
|------------------------------------|--------------|------------------|
| `question_content_match`           | Chọn câu phù hợp với nội dung | 最も合っているもの, 正しいもの, 本文の内容と合うもの, 内容に合っているものはどれか |
| `question_reason_explanation`      | Hỏi lý do / nguyên nhân (nhân quả) | なぜ, どうして, ～のはなぜか, ～の理由は何か |
| `question_reference`               | Hỏi đại từ chỉ định hoặc cụm được gạch chân | 「①...」とあるが、何を指すか, これ/それ/その/この + どんなもの |
| `question_meaning_interpretation`  | Hỏi nghĩa của câu/cụm từ | どういうことか, どういう意味か, ～とはどういうことか |
| `question_content_mismatch`        | Chọn câu KHÔNG phù hợp | 合わないもの, 正しくないもの, 間違っているもの |
| `question_author_opinion`          | Hỏi quan điểm/ý chính tác giả | 筆者の考え, 筆者は...について何と述べているか, 筆者が最も言いたいことはどれか, この文章全体のテーマはどれか |
| `question_fill_in_the_blank`       | Điền từ vào ô trống | [ ① ] に入るもの, ( 1 ) に入る最も適当なもの |

> **Lưu ý đoạn văn dài**: `question_fill_in_the_blank` hiếm dùng (data N1=7%, N3=0%). Skill **KHÔNG khuyến khích** dạng này.

### Distribution combo per level

**N1 (3 câu/bài)** — spec focus "outline / author's ideas":

| Combo | Q1 | Q2 | Q3 (bắt buộc author_opinion) |
|-------|----|----|------------------------------|
| **Combo 1** (phổ biến nhất) | `question_reference` (marker ①) | `question_reason_explanation` | `question_author_opinion` |
| Combo 2 (essay analytic) | `question_meaning_interpretation` | `question_reason_explanation` | `question_author_opinion` |
| Combo 3 (reference heavy) | `question_reference` (marker ①) | `question_reference` (marker ②) | `question_author_opinion` |

> **⛔ Rule N1**: **Câu cuối cùng của bài NÊN là `question_author_opinion`** — phù hợp spec "outline / main idea" của N1. Câu 3 test việc hiểu luận điểm chính / thái độ tác giả / ý tưởng tổng quát.

**N3 (4 câu/bài)** — spec focus "summary / logical development":

| Combo | Q1 | Q2 | Q3 | Q4 (bắt buộc tổng thể) |
|-------|----|----|----|-----|
| **Combo 1** (balanced, phổ biến) | `question_reference` (①) | `question_reference` (②) | `question_reason_explanation` | `question_content_match` |
| Combo 2 (reason heavy) | `question_reference` (①) | `question_reason_explanation` | `question_meaning_interpretation` | `question_content_match` |
| Combo 3 (essay analytic) | `question_reference` (①) | `question_reference` (②) | `question_reason_explanation` | `question_author_opinion` |
| Combo 4 (mix meaning) | `question_reference` (①) | `question_meaning_interpretation` | `question_reason_explanation` | `question_content_match` |

> **⛔ Rule N3**: **3 câu đầu test từng đoạn/cụm cụ thể, câu cuối là tổng thể** (`question_content_match` hoặc `question_author_opinion`) — phù hợp spec "summary / logical development".

> **BẮT BUỘC**: Trong 1 bài, **KHÔNG dùng cùng 1 `question_label` cho tất cả câu**. Phải có **≥ 2 labels khác nhau, lý tưởng ≥ 3** (vì 3-4 câu = đa dạng).

### Marker — Câu hỏi khớp HTML

Nếu câu hỏi dùng `question_reference` với "「①...」とあるが":
- Trong HTML PHẢI có `<span class="marker">①</span><u>...</u>` đúng cụm được hỏi.
- Với bài N1 có 2 câu reference → dùng `①` cho câu 1, `②` cho câu 2.
- Với bài N3 có 3 câu reference → dùng `①`, `②`, `③` tương ứng.

Nếu `question_meaning_interpretation`:
- Cụm từ hỏi nên có `<u>...</u>` (không bắt buộc marker ①② nếu chỉ 1 câu meaning).

Nếu `question_author_opinion` hoặc `question_content_match`:
- **KHÔNG cần marker** — câu này hỏi về tổng thể bài, không về 1 cụm từ cụ thể.

### Paraphrasing test (BẮT BUỘC cho đoạn văn dài)

Đáp án đúng KHÔNG được copy-paste 1 cụm ≥ 4 từ liên tiếp từ bài.

**SAI**: Bài viết `「この考え方は昔から多くの哲学者や作家が語ってきた普遍的な価値観である」`, đáp án chỉ chép `「多くの哲学者や作家が語ってきた」`.

**ĐÚNG**: Paraphrase — `「長い間、多くの思想家や文学者が述べてきたもの」`.

Đoạn văn dài N1/N3 đòi hỏi **paraphrase chặt chẽ** — không được copy > 4 từ (N1) hoặc > 5 từ (N3) liên tiếp.

### Câu hỏi phải độc lập

Mỗi câu hỏi đứng độc lập — không tham chiếu câu hỏi khác (không có `前の問題で言及した...`).

### Coverage — 3-4 câu test các đoạn KHÁC NHAU

Đoạn văn dài có 4-8 paragraph, thừa đủ nội dung để tách câu hỏi ra các đoạn riêng biệt.

**Map câu hỏi ↔ paragraph**:
- **N1 (3 câu, 5-8 paragraph)**: Q1 test paragraph 2-3, Q2 test paragraph 3-4 (hoặc paragraph khác Q1), Q3 test toàn bài (author_opinion)
- **N3 (4 câu, 4-6 paragraph)**: Q1 test paragraph 2, Q2 test paragraph 3-4, Q3 test paragraph 4-5 (reason), Q4 test toàn bài

### Văn phong câu hỏi (thể động từ) theo level — BẮT BUỘC

Câu hỏi (`question_X`) và 4 lựa chọn (`answer_X`) phải dùng đúng **thể động từ** theo level:

| Level | Thể bắt buộc | Đặc trưng kết câu |
|-------|--------------|-------------------|
| **N1, N2, N3** | **Thể thường** (普通体 / だ・である調) | `〜か。` / `〜のはどれか。` / `〜と考えられるか。` (KHÔNG dùng です/ます) |
| **N4, N5** | **Thể ます** (です・ます調) | `〜ですか。` / `〜のはどれですか。` / `〜と思いますか。` |

**Quy tắc cứng:**
- Đoạn văn dài CHỈ có **N1 và N3** → cả 2 level đều dùng **thể thường**, KHÔNG được dùng です/ます
- Câu hỏi và **cả 4 đáp án** phải nhất quán cùng thể (không trộn lẫn)

---

## R6. Đáp án (`answer_X`, `correct_answer_X`, `explain_vn_X`, `explain_en_X`)

### R6.1 Format 4 đáp án

```
answer_X = "Option A\nOption B\nOption C\nOption D"
```

- **Ngăn cách**: `\n` (newline) giữa các option
- **KHÔNG prefix** `1.`, `①`, `1)` — chỉ nội dung thuần
- **Đúng 4 option** per câu (không 3 không 5)
- Độ dài: tương đương nhau (không có 1 option ngắn bất thường)
- `correct_answer_X` = integer 1-4 (1-based)

**Trong CSV**, `answer_X` sẽ được lưu dưới dạng 1 cell đa dòng:

```csv
"選択肢1
選択肢2
選択肢3
選択肢4"
```

Dùng `fill_qa.py` để auto-quote.

### R6.2 Nguyên tắc đáp án

- **Đáp án đúng**: xác nhận rõ bởi thông tin/ý trong bài, PHẢI paraphrase (cả N1 và N3).
- **Đáp án sai (distractor)**: PHẢI dùng **thông tin/ý THẬT từ bài** nhưng sai ngữ cảnh/hiểu lầm. **NGHIÊM CẤM bịa thông tin không có trong bài**.
- **Đoạn văn dài = distractor tinh vi nhất trong series** — distractor sai ở 1 nuance (thời gian, nhân-quả đảo, phạm vi, subject).
- **Đa dạng vị trí đúng**: trong batch ≥ 10 bài, `correct_answer` phân bố đều 1/2/3/4 (không lệch quá 40% về 1 con số).

### R6.3 Các loại bẫy (distractor traps) — BẮT BUỘC đa dạng

Trong 4 đáp án (1 đúng + 3 sai), 3 sai PHẢI dùng ≥ 3 loại bẫy khác nhau:

| Loại bẫy | Mô tả | Ví dụ |
|----------|-------|-------|
| **① Reversal** | Đảo ngược logic/điều kiện/quan hệ nhân-quả từ bài | Bài nói "A dẫn đến B" → distractor nói "B dẫn đến A" |
| **② Detail swap** | Đổi chi tiết (subject/object/số lượng/thời gian) | Bài: "研究チームは植物で実験した" → distractor: "研究チームは動物で実験した" |
| **③ Scope** | Mở rộng/thu hẹp phạm vi | Bài: "ある場合" → distractor: "どんな場合も" |
| **④ Misinterpretation** | Hiểu sai nghĩa từ/cụm từ chỉ định | `この考え方` ám chỉ X, distractor diễn giải là Y |
| **⑤ Part of truth** | Chỉ đúng 1 phần của bài, bỏ sót ý quan trọng | Bài có 2 luận điểm A+B → distractor chỉ nói A (thiếu B) |
| **⑥ Over-generalization** | Áp dụng cho mọi người/mọi trường hợp trong khi bài chỉ nói 1 đối tượng | Bài: "筆者は" → distractor: "日本人は全員" |

### R6.4 Self-test cho distractor (trước khi finalize)

Với mỗi distractor, tự hỏi:

1. **Textual evidence test**: Distractor này có dùng info/ý xuất hiện trong bài không?
   - KHÔNG → đang bịa → sửa
2. **Trap type test**: Distractor này thuộc loại bẫy nào? Đã match 1 trong 6 loại chưa?
   - KHÔNG → đang random → sửa
3. **Plausibility test**: Người đọc nhanh có thể chọn nhầm đáp án này không?
   - KHÔNG (quá xa) → đáp án quá dễ loại → sửa cho gần hơn
   - CÓ NHƯNG đọc kỹ lại thấy sai → OK
4. **Refutation test**: Trích được câu cụ thể từ bài để bác bỏ distractor không?
   - KHÔNG → đang bịa → sửa
5. **Only-one-correct test**: Có thể có 2 đáp án cùng đúng không?
   - CÓ → sửa cho rõ chỉ 1 đáp án đúng

### R6.5 Format explanation — 3 phần BẮT BUỘC (VN + EN)

> **Explanation không chỉ "có nội dung" — nó phải CHỨNG MINH câu hỏi + đáp án đúng có logic.**
> Đoạn văn dài 5-8 paragraph nên explanation PHẢI **chỉ rõ paragraph số mấy**, trích cụ thể từ bài để chứng minh.

**Phần 1 — Đáp án đúng (BẮT BUỘC trích bài + paragraph):**
- Nêu rõ "Đáp án đúng: (X)" + nội dung paraphrase
- **Trích dẫn câu/đoạn cụ thể** trong bài xác nhận đáp án (vd: "Paragraph 4 viết: `「...」`", "Câu cuối paragraph 6: `「...」`")
- **Bắt buộc chỉ rõ paragraph số mấy** vì bài dài — không đủ để nói "trong bài"
- Nếu là author_opinion (câu cuối): trích **thesis statement** ở paragraph mở đầu / kết / chuyển ý
- Nêu paraphrase: đáp án dùng từ đồng nghĩa nào với bài

**Phần 2 — Đáp án sai (TỪNG đáp án + loại bẫy):**
- Đi qua **TẤT CẢ 3 đáp án sai** (1 đáp án 1 dòng), không bỏ sót
- Mỗi đáp án sai phải nêu:
  1. **Loại bẫy** (Reversal / Detail swap / Scope / Misinterpretation / Part of truth / Over-generalization / Mixing)
  2. **Trích cụ thể** từ bài chứng minh sai + **chỉ rõ paragraph** (vd: "Paragraph 2 nói X nhưng đáp án 2 nói Y → trái ý")
- KHÔNG dùng câu chung chung — phải chỉ rõ paragraph nào, ý nào trong bài đủ để bác bỏ

**Phần 3 — Tóm tắt chiến lược:**
- 1 câu ngắn: thí sinh cần làm gì để giải đúng (vd: "Câu cuối → tìm thesis ở paragraph kết, loại trừ chi tiết phụ ở paragraph giữa").

### R6.6 Ví dụ đáp án + explanation (BÀI N1 mẫu — 6 paragraph)

**Bài N1** (giả tưởng — 6 paragraph): Tác giả viết về ngôn ngữ và tư duy. Paragraph 1-2 nêu vấn đề (đơn giản hoá ngôn ngữ ngày nay), paragraph 3-4 phân tích cơ chế "ngôn ngữ định hình tư duy", paragraph 5 đề cập đa ngôn ngữ, paragraph 6 kết luận thesis.

**Question 3** (`question_author_opinion` — câu cuối):
> この文章で筆者が最も伝えたいことはどれか。

**Answers** (4 options, no prefix):
```
言語は思考そのものを形作るため、豊かな語彙を保持することが重要である
複数の言語を話せる人は、複数の思考様式を持つことができる
現代社会では単純化された言葉が便利で望ましい
言語学者の研究によって思考の限界が明らかになった
```

**correct_answer**: 1

**explain_vn**:
```
ĐÁP ÁN ĐÚNG (1): 言語は思考そのものを形作るため、豊かな語彙を保持することが重要である (Ngôn ngữ định hình chính tư duy nên giữ gìn vốn từ phong phú là quan trọng).
Paragraph 6 (kết) viết: 「言葉を守ることは、思考を守ることであり、ひいては自分自身を守ることに他ならない」 (Bảo vệ ngôn ngữ là bảo vệ tư duy, và cuối cùng là bảo vệ chính mình). Paragraph 3-4 đã xây dựng cơ chế "ngôn ngữ định hình tư duy". Đáp án 1 paraphrase đúng thesis tổng thể.

ĐÁP ÁN SAI:
(2) 複数の言語を話せる人は... — Part of truth: Paragraph 5 có đề cập đa ngôn ngữ, nhưng tác giả viết 「多言語話者であっても思考様式が複数になるとは限らない」 — phủ định luận điểm này. KHÔNG phải ý chính.
(3) 現代社会では単純化された言葉が便利で望ましい — Reversal: Paragraph 1-2 phê phán "đơn giản hoá ngôn ngữ" là vấn đề. Đáp án 3 đảo ngược thesis — sai hoàn toàn.
(4) 言語学者の研究によって思考の限界が明らかになった — Scope: Bài KHÔNG đề cập "giới hạn tư duy" hay "nghiên cứu của nhà ngôn ngữ học". Đây là phạm vi bịa, không có trong bất kỳ paragraph nào.

Tóm tắt: Câu cuối (author_opinion) → trích thesis ở paragraph kết (P6), so với phương án; loại trừ đáp án "đảo ngược thesis" (3), "ý phụ trong P5" (2), "bịa phạm vi" (4).
```

**explain_en**:
```
CORRECT ANSWER (1): 言語は思考そのものを形作るため、豊かな語彙を保持することが重要である (Language shapes thought itself, so preserving rich vocabulary is important).
Paragraph 6 (conclusion) states: 「言葉を守ることは、思考を守ることであり、ひいては自分自身を守ることに他ならない」 (Protecting language is protecting thought, and ultimately protecting oneself). Paragraphs 3-4 build up the mechanism of "language shapes thought." Option 1 correctly paraphrases the overall thesis.

WRONG ANSWERS:
(2) 複数の言語を話せる人は... — Part of truth: Paragraph 5 mentions multilingualism, but the author writes 「多言語話者であっても思考様式が複数になるとは限らない」 — refuting this very claim. NOT the main idea.
(3) 現代社会では単純化された言葉が便利で望ましい — Reversal: Paragraphs 1-2 criticize "language simplification" as the problem. Option 3 reverses the thesis — completely wrong.
(4) 言語学者の研究によって思考の限界が明らかになった — Scope: Article does NOT mention "limits of thought" or "linguistic research." This scope is fabricated; no paragraph contains it.

Summary: For final author_opinion questions → quote the thesis in the closing paragraph (P6) and compare to options; eliminate "thesis reversal" (3), "secondary point in P5" (2), "fabricated scope" (4).
```

**Explanation phải bằng cả 2 ngôn ngữ (VN + EN)** với cùng nội dung logic — không phải dịch máy.

### R6.7 Đa câu trong 1 bài — Tránh trùng

**BẮT BUỘC**:
- Câu Q1 test đoạn/ý khác hẳn Q2, Q3, Q4
- Nếu Q1 hỏi paragraph 2-3 thì Q2 phải hỏi paragraph 4-5
- Không được 2 câu cùng test `この現象` (cùng reference)
- Không được 2 câu đều là `question_reason_explanation` test cùng 1 nguyên nhân
- **N1**: câu 3 phải test tổng thể (`author_opinion`), không trùng paragraph của câu 1/2
- **N3**: câu 4 phải test tổng thể (`content_match`/`author_opinion`), 3 câu đầu mỗi câu 1 đoạn riêng
