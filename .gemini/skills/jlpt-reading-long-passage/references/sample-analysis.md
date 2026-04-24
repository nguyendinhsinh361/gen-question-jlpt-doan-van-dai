# Sample Analysis — Đoạn Văn Dài Reference Data

Phân tích định lượng dữ liệu mẫu thực tế ở `data/doan_van_dai_n{1,3}_clean.json` để gen agent biết **chính xác** mức độ dài, số câu hỏi, và pattern HTML cho N1/N3.

Số liệu chạy bằng `load_references.py --stats` + phân tích pattern.

## 0. Scope — CHỈ N1 & N3

| Level | Có "đoạn văn dài"? |
|-------|--------------------|
| N1    | ✅ `question_child=3`, ~1000 chars (spec) |
| N2    | ❌ KHÔNG có trong `question_format.json` |
| N3    | ✅ `question_child=4`, ~550 chars (spec) |
| N4    | ❌ KHÔNG có |
| N5    | ❌ KHÔNG có |

Skill này chỉ handle N1 và N3.

## 1. Phân Bố Độ Dài (`jp_char_count`)

| Level | Samples | Min  | P25  | P50  | P75  | Avg  | Max  |
|-------|---------|------|------|------|------|------|------|
| N1    | 27      | 1009 | 1045 | 1104 | 1141 | 1109 | 1347 |
| N3    | 34      | 561  | 598  | 660  | 863  | 743  | 1344 |

**Kết luận**:
- **N1 target** = P25–P75 = **1000–1150** (rounded)
- **N3 target** = dùng P25 + margin hẹp = **550–700** (spread rộng nhưng spec nói 550)
- **Hard reject**: N1<900, N3<500
- N3 max=1344 là outlier (bài essay bất thường) → spec ~550, skill dùng 550-700

## 2. Số Câu Hỏi Per Sample

| Level | 2q | 3q | 4q | 5q | Dominant |
|-------|----|----|----|----|----------|
| N1    | 0  | 7 (26%)  | **20 (74%)** | 0 | 4q (noise) |
| N3    | 0  | 6 (18%)  | **28 (82%)** | 0 | 4q |

**Mismatch giữa data & spec**:
- `rules/question_format.json` spec: N1=**3** câu, N3=**4** câu
- Data N1 hay có 4 câu (74%) — legacy dataset noise
- **Skill FOLLOW SPEC** — N1 gen đúng 3 câu, N3 đúng 4 câu

## 3. Pattern HTML Phổ Biến

| Pattern | N1 | N3 |
|---------|----|----|
| Có `<p>` | ~100% | ~100% |
| Có `<br>` | 29% | **52%** |
| Có `<ruby>` | **0%** | **0%** |
| Có `<u>` underline | **85%** | 50% |
| Có `<span>` | 11% | 35% |
| Có 注 annotation | 25% | **64%** |
| Có marker ①②③④ | 29% | **79%** |
| Có source line | **70%** | 38% |
| Có blank `[ ]`/(1) | 7% | 0% |

### Nhận xét Quan Trọng

1. **`<ruby>` = 0% ở cả 2 level** — đề đoạn văn dài thực tế KHÔNG dùng furigana. Skill cho phép nhưng **khuyến nghị 0-3 cặp**, ưu tiên không có.

2. **Marker ①②③ N3 lên đến 79%** — với 4 câu hỏi, 2-3 câu thường là reference-style có marker. Đây là key insight — N3 gần như luôn có ít nhất 1 marker.

3. **Annotation 注 N3 cao nhất (64%)** — bài dài có nhiều thuật ngữ cần giải thích. **Khuyến nghị** thêm 2-3 注 cho N3.

4. **Source line N1 = 70%** — đoạn văn dài N1 thường là tiểu luận/phê bình có citation. Rất khuyến nghị.

5. **`<u>` N1 = 85%** — rất cao. Do nhiều câu reference/meaning. Skill dùng `<u>` cho MỌI cụm từ được hỏi.

6. **`<br>` N3 = 52%** — data có nhiều (vì format thư/tuỳ bút) — **skill KHÔNG bắt chước**, dùng `<p>` thuần.

7. **Blank `[ ]` gần như 0%** — đoạn văn dài không dùng fill_in_the_blank. Không áp dụng label `fill_in_the_blank` cho dạng này.

## 4. Distribution Đề Xuất Per Level

### N1 (3 câu/bài) — spec focus "outline hoặc author's ideas"

Combo ưu tiên:

**Combo 1** (phổ biến nhất — 50%+ nên dùng):
- Q1: reference (marker ①) — test 1 đoạn cụ thể
- Q2: reason_explanation — test logic chain
- Q3: author_opinion — tổng kết luận điểm chính

**Combo 2** (essay analytic):
- Q1: meaning_interpretation — test hiểu nuance
- Q2: reason_explanation
- Q3: author_opinion

**Combo 3** (reference heavy):
- Q1: reference (①)
- Q2: reference (②)
- Q3: author_opinion

> **Rule N1**: câu cuối cùng NÊN là `author_opinion` (phù hợp spec "outline/main idea").

### N3 (4 câu/bài) — spec focus "summary, logical development"

Combo ưu tiên:

**Combo 1** (balanced, phổ biến):
- Q1: reference (marker ①)
- Q2: reference (marker ②)
- Q3: reason_explanation
- Q4: content_match (tổng thể)

**Combo 2** (reason heavy):
- Q1: reference (①)
- Q2: reason_explanation
- Q3: meaning_interpretation
- Q4: content_match

**Combo 3** (essay analytic):
- Q1: reference (①)
- Q2: reference (②)
- Q3: reason_explanation
- Q4: author_opinion

> **Rule N3**: 3 câu đầu test từng đoạn/cụm cụ thể, câu 4 là tổng thể (content_match hoặc author_opinion).

## 5. Chủ Đề Phổ Biến (từ đọc sample)

| Level | Chủ đề hay gặp |
|-------|----------------|
| N1    | Triết học ngôn ngữ/tư duy, phê bình văn hóa sâu, phân tích hệ thống giáo dục/xã hội, tùy bút cao cấp, phê bình văn học |
| N3    | Thư dài có nội dung, essay về đời sống/học tập, giai thoại có ý nghĩa, bài báo có phân tích, câu chuyện có lesson |

## 6. Câu Hỏi Phổ Biến Theo Level

### N1 — Formal, outline/author's ideas

- `「①...」とあるが、どういう意味か。` — meaning_interpretation (marker)
- `筆者は、なぜ...と考えているのか。` — reason_explanation
- `筆者が最も言いたいことはどれか。` — author_opinion
- `この文章全体のテーマとして最も適切なものはどれか。` — author_opinion (tổng quát)

### N3 — Summary, logical development

- `①「...」とあるが、何を指すか。` — reference
- `②「...」とあるが、どういうことか。` — meaning/reference
- `筆者はなぜ...と言っているか。` — reason
- `この文章の内容と合っているものはどれか。` — content_match (tổng thể)

## 7. Rule Tóm Lược Cho Gen Agent

1. **Độ dài**: N1 target 1000-1150, N3 target 550-700
2. **Số câu hỏi**: Theo SPEC — N1=3, N3=4 (KHÔNG theo data noise)
3. **Trong 1 bài**: ≥ 2 `question_label` khác nhau (ideally ≥ 3)
4. **Các câu hỏi KHÔNG test cùng 1 đoạn** — bài dài có nhiều đoạn, tách ra
5. **N1 câu cuối = `author_opinion`** (focus outline)
6. **N3 câu cuối = `content_match`/`author_opinion`** (focus summary)
7. **Marker ①②③** cho mỗi câu reference — đặt trong HTML
8. **Annotation 注** khuyến nghị N3 (2-3 cái), N1 (1-2 cái)
9. **Source line** khuyến nghị N1 (70%), N3 (38%)
10. **HTML**: 5-8 paragraph N1, 4-6 paragraph N3
11. **Furigana ít**: data 0%, skill max 3-6 cặp

## 8. Tóm Tắt Spec Đoạn Văn Dài

| Aspect | N1 | N3 |
|--------|-----|-----|
| Scope | Only N1 | Only N3 |
| Char count | 1000–1150 (hard reject < 900) | 550–700 (hard reject < 500) |
| Questions/bài | 3 | 4 |
| Paragraph count | 5–8 | 4–6 |
| Container CSS | 780px, padding 56px 64px, line-height 1.95 | 780px, padding 56px 64px, line-height 1.95 |
| Marker `①②③` rate | 29% | 79% |
| Underline `<u>` rate | 85% | 50% |
| Annotation 注 rate | 25% | 64% |
| Source line rate | 70% | 38% |
| Fill-in-blank | 0% (không dùng) | 0–7% (không khuyến nghị) |
| Câu cuối label | `question_author_opinion` | `question_content_match` / `question_author_opinion` |

**Workload đoạn văn dài cao** (bài dài + nhiều câu + distractor tinh vi) — đòi hỏi theo dõi sát 32-check QC.
