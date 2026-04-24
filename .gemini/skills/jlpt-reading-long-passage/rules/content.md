# Rules: Nội dung, Layout, Format & Visual (R1, R2, R7, R8)

> **Scope**: Đoạn văn dài (長文 / long-passage) — văn xuôi Nhật 550–1150 ký tự với **3-4 câu hỏi** trắc nghiệm. **CHỈ áp dụng cho N1 (3 câu) và N3 (4 câu)** — N2/N4/N5 không có dạng này per `rules/question_format.json`.

## R1. Chủ đề & Văn phong theo level

> **NGUYÊN TẮC**: Đoạn văn dài là **văn nghị luận / essay / thư dài / tùy bút** có luận điểm triển khai qua nhiều đoạn — test khả năng **hiểu outline, logic development, ý tác giả, câu tham chiếu** qua 3-4 câu hỏi phủ các đoạn khác nhau (intro / body / counter / conclusion).

| Level | Chủ đề | Văn phong | Cấu trúc câu |
|-------|--------|-----------|--------------|
| **N3** | Essay dài về đời sống, bài báo có phân tích, thư dài có nội dung xã hội, giai thoại có ý nghĩa, câu chuyện có bài học | Nửa formal nửa conversational | ～について, ～によって, ～に対して, ～場合は, ～ために, ～にとって |
| **N1** | Tiểu luận triết học / văn hóa / xã hội, phê bình văn học sâu, luận điểm có triển khai logic phức tạp, tùy bút cao cấp, phân tích hệ thống giáo dục/xã hội | Rất formal, văn viết cao cấp | ～いかんによらず, ～をもって, ～に先立ち, ～にもかかわらず, keigo cao |

### Topic tag — BẮT BUỘC

Tag chọn từ `rules/topic.json` — đoạn văn dài dùng **tiếng Việt** (khớp data mẫu & `smoke_test.csv`).

| Category | Ví dụ tag (vi) | Phù hợp level |
|----------|----------------|---------------|
| Tiểu luận / phê bình cao cấp | `triết học`, `phê bình văn học`, `phê bình văn hóa`, `tiểu luận` | **N1** |
| Xã hội / văn hóa | `xã hội`, `văn hóa`, `truyền thống`, `ngôn ngữ`, `môi trường` | N1, N3 |
| Khoa học / công nghệ | `khoa học`, `công nghệ`, `sinh học`, `y học`, `thí nghiệm` | N1, N3 |
| Công việc / kinh tế | `công việc`, `kinh tế`, `thương mại`, `sự nghiệp` | **N1** |
| Đời sống / cá nhân | `đời sống`, `thư từ`, `nhật ký dài`, `kỷ niệm`, `trải nghiệm cá nhân` | **N3** |
| Giáo dục | `giáo dục`, `học tập`, `nghiên cứu`, `trường học` | N1, N3 |
| Tâm lý / cảm xúc | `tâm lý học`, `cảm xúc`, `tư duy`, `phát triển bản thân` | N1, N3 |
| Nghệ thuật / thể thao / du lịch | `nghệ thuật`, `âm nhạc`, `văn học`, `thể thao`, `du lịch` | N1 |

> **⚠️ KHÔNG dùng tag tiếng Anh hoặc tiếng Nhật.**
> Phải dùng tiếng Việt đúng cột `vi` của `rules/topic.json` (ví dụ: ✅ `văn hóa`, `triết học`, `đời sống`).

Trong batch ≥ 3 bài, chọn topic từ ≥ 2 category khác nhau để đa dạng.

---

## R2. Format văn bản & Độ dài

### Target character count (BẮT BUỘC)

Đo bằng `count_body_chars()` — đếm **ký tự visible trong body**, bỏ whitespace, bỏ `<rt>` furigana.

Dữ liệu tham khảo từ sample JSON `data/doan_van_dai_n{1,3}_clean.json`:

| Level | Samples | Min  | P25  | P50  | P75  | Avg  | Max  |
|-------|---------|------|------|------|------|------|------|
| N1    | 27      | 1009 | 1045 | 1104 | 1141 | 1109 | 1347 |
| N3    | 34      | 561  | 598  | 660  | 863  | 743  | 1344 |

Target range khuyến nghị:

| Level | Target Range | Hard Reject (< Min) |
|-------|--------------|---------------------|
| **N1**    | **1000–1150** | < 900 → gen lại    |
| **N3**    | **550–700**   | < 500 → gen lại    |

> **🚫 HARD REJECT**: Nếu `count_body_chars()` **thấp hơn Hard Reject threshold**, bài **PHẢI gen lại từ đầu**. Không chấp nhận, không chỉnh sửa nhỏ — gen lại hoàn toàn.
> **⚠️ UNDER TARGET**: Dưới Target Range nhưng ≥ Hard Reject → bổ sung 1-2 câu văn hoặc thêm 1 đoạn elaboration / 1 `注` annotation.
> **⚠️ OVER TARGET**: Cho phép dài hơn tới +50 chars (bài dài cho phép chênh lệch rộng hơn); quá nhiều thì cảnh báo.

**Lưu ý**: N3 P75 = 863, Max = 1344 là outlier (bài essay bất thường trong dataset). Spec chính thức là ~550, nên skill dùng 550-700 để sát spec.

### Số câu hỏi per bài (BẮT BUỘC)

Dựa trên `rules/question_format.json` (spec JLPT chính thức):

| Level | Q Count | Focus spec |
|-------|---------|------------|
| **N1** | **3**   | outline / author's ideas |
| **N3** | **4**   | summary / logical development |

- N2, N4, N5 **KHÔNG CÓ** dạng đoạn văn dài — skill này chỉ apply N1/N3.
- CSV columns `question_1..question_3` (N1) hoặc `question_1..question_4` (N3) populate đủ.
- Các column còn lại = "" (empty string).
- **Data gốc có noise** — N1 data phần lớn có 4 câu (74%), N3 có 4 câu (82%). Skill luôn follow spec (N1=3, N3=4), không bắt chước data.

### Paragraph count (BẮT BUỘC)

| Level | Paragraph count | Lý do |
|-------|-----------------|-------|
| **N1** | **5–8** | Bài 1000-1150 chars, essay có intro/body/counter/conclusion |
| **N3** | **4–6** | Bài 550-700 chars, story/essay có flow đơn giản hơn |

> **⛔** Bài chỉ 2-3 paragraph ở 1000+ chars → quá đặc, khó đọc → REJECT. Chia nhỏ theo đoạn ý tưởng.

### Flow text (KHÔNG `<br>` giữa câu)

> **⛔ LỖI PHỔ BIẾN**: Data gốc N3 dùng `<br>` nhiều (52%) để mô phỏng thư/tuỳ bút.
> **Output KHÔNG dùng `<br>` giữa câu** — dùng `<p>` thuần, text flow liên tục.

**Quy tắc:**
- **ĐÚNG**: `<p>Câu 1。Câu 2。Câu 3。</p>` — 1 paragraph 1 `<p>`
- **SAI → REJECT**: `<p>Câu 1。<br>Câu 2。</p>`
- **Ngắt paragraph** chỉ khi chuyển ý hoàn toàn khác
- **Xuống hàng trong source code** chỉ để dễ đọc; HTML parser sẽ collapse whitespace

Đoạn văn dài **KHÔNG có exception `<br>`** nào — luôn dùng `<p>` thuần.

### CSS layout bắt buộc

- **Container**: `max-width: 780px`, `margin: 0 auto`, `padding: 56px 64px`, white background trên light gray body `#f9fafb`
- **Body**: `word-break: keep-all`, `line-break: strict`, `overflow-wrap: break-word`, `line-height: 1.95`
  (`keep-all` đảm bảo xuống dòng ở ranh giới từ, tránh cắt kanji compound)
- **Paragraph**: `<p>` với `text-indent: 1em` (chuẩn văn Nhật)
- **Font**: Noto Sans JP qua Google Fonts (KHÔNG dùng Tailwind CDN)

Template chi tiết xem `rules/technical.md` R9.

### Test mơ hồ (BẮT BUỘC)

> **Mỗi đoạn văn phải có DUY NHẤT 1 cách hiểu hợp lý cho mỗi câu hỏi.**
> Sau khi viết xong, đọc lại: "Câu hỏi này có thể hiểu theo cách thứ 2 không?" Nếu có → sửa lại.
> **Đặc biệt quan trọng với 3-4 câu multi-question**: các câu hỏi KHÔNG được test cùng 1 đoạn/ý.

---

## R7. Các dạng bài (document formats)

Đoạn văn dài có 5 dạng chính:

| # | Format | Level | Đặc điểm | Ví dụ chủ đề |
|---|--------|-------|----------|--------------|
| 1 | **Essay triết học / phê bình** | **N1** | 5-8 paragraph văn nghị luận sâu, có luận điểm + counter-argument + kết luận, source line ≥70% | 言葉と思考, 文化論, 社会批評, 教育論 |
| 2 | **Tiểu luận văn hóa / xã hội** | N1 | Phân tích hệ thống xã hội, 5-7 paragraph | 現代社会, 伝統と革新, ジェンダー |
| 3 | **Essay / bài báo có phân tích** | **N3** | 4-6 paragraph essay nhẹ, có intro + body + reflection + conclusion | 実験, 動物, 自然, 健康 |
| 4 | **Thư dài có nội dung** | **N3** | Thư 4-6 paragraph có câu chuyện + rút ra bài học | 友情, 家族, 学校生活 |
| 5 | **Giai thoại / anecdote dài** | N3 | Câu chuyện 4-6 đoạn có plot + ý nghĩa | 子どもの時, 旅行, 出会い |

Mỗi bài phải có đủ nội dung để trả lời 3-4 câu hỏi phủ các đoạn khác nhau — vì vậy **minimum 4-5 paragraph**.

### Source line convention

Format: `（[fake author]「[fake title]」による）` hoặc `（[author]「[title]」より）`

- Author: tên Nhật tự chế (2-4 chữ, họ + tên), VD: `山口和彦`, `田中由美`, `佐藤健一`, `鈴木美咲`
- **⛔ TUYỆT ĐỐI KHÔNG** dùng tên tác giả thật (`村上春樹`, `夏目漱石`, `芥川龍之介`, `太宰治`, `吉本ばなな`...)
- Title Nhật tự chế ngắn gọn phù hợp nội dung, VD: `「言葉と思考の未来」`, `「日々の小さな発見」`, `「現代社会のゆくえ」`

### Tần suất source line theo data (khuyến nghị)

| Level | Tần suất thực tế | Có nên thêm? |
|-------|------------------|--------------|
| **N1**    | **70%**              | **RẤT NÊN** — đặc trưng essay/phê bình N1 |
| **N3**    | 38%              | Optional — khi bài là essay/phê bình |

---

## R8. Visual elements cho multi-question

### Marker ①②③④

**Tần suất data**: N1=29%, **N3=79%**.

Với đoạn văn dài 3-4 câu hỏi, **marker dùng rất nhiều** — mỗi câu `question_reference` cần 1 marker riêng, đặt rải đều qua các đoạn.

**Quy tắc**:
- Mỗi câu `question_reference` → 1 marker riêng trong bài (`①`, `②`, `③`)
- Marker đứng **ngay trước** cụm từ bị hỏi, kèm `<u>...</u>`:
  ```html
  <p>... <span class="marker">①</span><u>このような現象</u>は ...</p>
  ...
  <p>... <span class="marker">②</span><u>この傾向</u>は ...</p>
  ...
  <p>... <span class="marker">③</span><u>この結論</u>に達した ...</p>
  ```
- Question 1: `「①このような現象」とあるが、何を指しているか。`
- Question 2: `「②この傾向」とあるが、どのような傾向か。`
- Question 3: `「③この結論」とあるが、どういう結論か。`

**Marker Strategy Cho 3-4 Câu Hỏi** (xem `references/html-patterns.md` section 4 cho 4 pattern cụ thể):
- **N1 (3 câu)**: 1-2 marker + câu cuối tổng kết (author_opinion)
- **N3 (4 câu)**: 2-3 marker + câu cuối tổng thể (content_match/author_opinion)

### Underline `<u>`

**Tần suất data**: **N1=85%** (cao nhất series), N3=50%.

Lý do: mỗi câu hỏi reference/meaning đều cần gạch chân cụm từ bị hỏi. Skill dùng `<u>` cho MỌI câu hỏi `question_reference` và `question_meaning_interpretation`.

### Annotation `注`

**Tần suất data**: N1=25%, **N3=64%**.

Bài dài → nhiều thuật ngữ cần giải thích. **Khuyến nghị**:
- **N3**: **RẤT NÊN** thêm 2-3 `注` (data 64%)
- **N1**: Optional 1-2 `注` (data 25%, chỉ khi có thuật ngữ chuyên môn)

Format:
```html
<div class="annotations">
    <p>注1 普遍的（ふへんてき）：どこにでも共通すること</p>
    <p>注2 振動（しんどう）：ゆれること</p>
</div>
```

- Đặt **ngay trước** `<p class="source">` (nếu có)
- Mỗi 注 một `<p>` riêng, dùng full-width `：`
- Số thứ tự `注1`, `注2`, `注3`
- Giải thích bằng **tiếng Nhật đơn giản** — KHÔNG tiếng Anh/Việt

### Source line

**Tần suất**: N1=70%, N3=38% (xem R7 bên trên).

### Blank `[ ]` / `( 1 )` — gần như KHÔNG dùng

**Tần suất data**: N1=7%, N3=0%.

Đoạn văn dài rất hiếm khi dùng blank. Skill **KHÔNG khuyến khích** fill_in_the_blank cho đoạn văn dài.

### Cheatsheet — Visual elements per level

| Level | Marker ①②③ | `<u>` | 注 | Source | Blank `[ ]` |
|-------|------------|-------|----|--------|-------------|
| **N1** | Đôi khi (29%) | **RẤT CÓ (85%)** | Ít (25%) | **RẤT NÊN (70%)** | Gần như không (7%) |
| **N3** | **RẤT CÓ (79%)** | Có (50%) | **RẤT NÊN (64%)** | Đôi khi (38%) | Không (0%) |

### Common errors

1. ❌ Có marker `①` trong bài nhưng KHÔNG có câu hỏi reference tương ứng → marker vô nghĩa
2. ❌ Câu hỏi hỏi `①...とあるが` nhưng bài không có marker `①` → câu hỏi không thể trả lời
3. ❌ 3-4 câu hỏi trong 1 bài đều test cùng 1 đoạn → thiếu coverage
4. ❌ Source line dùng tên tác giả thật → vi phạm IP
5. ❌ N1 thiếu source line → giảm authenticity (data 70%)
6. ❌ N3 không có annotation → giảm authenticity (data 64%)
7. ❌ Dùng `<br>` giữa câu → sai, dùng `<p>` thuần
8. ❌ Bài chỉ 2-3 paragraph → quá đặc, chia 5-8 (N1) hoặc 4-6 (N3)
