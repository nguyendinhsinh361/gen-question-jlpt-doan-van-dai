# HANDOVER — Đoạn Văn Dài Skill

Tài liệu giao/nhận cho skill **jlpt-reading-long-passage** (長文読解 / đoạn văn dài). Đọc file này trước khi chạy batch gen hoặc khi bàn giao cho team mới.

## 1. Mục đích

Gen dữ liệu huấn luyện AI cho **dạng "đoạn văn dài"** (長文読解) của JLPT. Mỗi bài gồm:

- 1 đoạn văn Nhật **dài** (550–1150 ký tự tuỳ level)
- **3 hoặc 4 câu hỏi** multiple-choice 4 đáp án (N1 = 3 câu, N3 = 4 câu)
- Giải thích tiếng Việt + tiếng Anh cho từng câu

**Scope hẹp — CHỈ N1 và N3**. Theo `rules/question_format.json`, N2/N4/N5 KHÔNG có kind "đoạn văn dài", nên skill hard-block các level đó (status `UNSUPPORTED_LEVEL`, exit 1).

So với các phase trước:

| Phase | Kind | Levels | Chars | Q/bài | Container |
|-------|------|--------|-------|-------|-----------|
| 0 | tìm thông tin | N1-N5 | varies | 1 | — (có PNG) |
| 1 | đoạn văn ngắn | N1-N5 | 80–290 | 1 | 640px |
| 2 | đoạn văn vừa | N1-N5 | 250–620 | 2–3 | 720px |
| **3** | **đoạn văn dài** | **N1 + N3** | **550–1150** | **3–4** | **780px** |

**Đoạn văn dài = workload cao nhất** (bài dài + nhiều câu + distractor tinh vi). Khuyến nghị batch nhỏ (3-5 bài/lần).

## 2. Cấu trúc project

```
gen-question-doan-van-dai/
├── data/                                      # Sample JSON từ đề JLPT cũ
│   ├── doan_van_dai_n1_clean.json             # 27 samples
│   └── doan_van_dai_n3_clean.json             # 34 samples
├── .claude/skills/jlpt-reading-long-passage/
│   ├── SKILL.md                               # Main skill definition
│   ├── scripts/
│   │   ├── process_html.py                    # Count + clean HTML + CSV upsert (3-4 Q)
│   │   └── load_references.py                 # Pretty-print JSON cho gen agent
│   └── references/
│       ├── sample-analysis.md                 # Phân tích pattern N1/N3
│       └── html-patterns.md                   # HTML template + marker strategy
├── .gemini/skills/jlpt-reading-long-passage/  # Mirror identical của .claude/
├── assets/html/                               # Output HTML files (runtime)
├── sheets/                                    # Output CSV files (runtime)
├── rules/                                     # Schema & spec
│   ├── question_sheet.csv                     # 45-col CSV header
│   ├── question_format.json                   # Xác nhận N1=3, N3=4
│   ├── kind_mission_mapping.json
│   ├── mission.json                           # Question label catalog
│   └── topic.json
├── HANDOVER.md                                # (file này)
└── PROMPTS.md                                 # Prompt templates cho gen agent
```

## 3. Pipeline chuẩn

### Bước 1 — Load references (calibrate style)

```bash
cd /path/to/gen-question-doan-van-dai
python3 .claude/skills/jlpt-reading-long-passage/scripts/load_references.py --stats
python3 .claude/skills/jlpt-reading-long-passage/scripts/load_references.py --level N1 --count 1 --seed 42
python3 .claude/skills/jlpt-reading-long-passage/scripts/load_references.py --level N3 --count 1 --seed 42
```

Gen agent đọc 1 sample cùng level (bài dài — đọc 1 là đủ) để học:
- Độ dài (P25–P75 của data: N1 ≈ 1045–1141, N3 ≈ 598–863)
- Chủ đề (N1 = triết học ngôn ngữ/phê bình xã hội; N3 = thư dài/essay đời sống)
- Cấu trúc multi-question (marker ①②③ nhiều, annotation 注 nhiều cho N3)

**KHÔNG bắt chước styling data gốc** — data có `<br>` và số câu 4 cho N1 (noise). Chỉ học **nội dung + question pattern**.

### Bước 2 — Gen HTML + câu hỏi từ LLM

LLM dùng prompt trong `PROMPTS.md` (template N1 / N3) để gen ra:

1. HTML file đầy đủ (có `<!DOCTYPE>`, Noto Sans JP CSS, `max-width: 780px`)
2. **3 câu (N1) hoặc 4 câu (N3)**, mỗi câu 4 đáp án + đáp án đúng
3. Giải thích VN + EN cho từng câu
4. Source line khuyến nghị (N1 70% nên có, N3 38% optional)
5. Annotation 注 khuyến nghị (N3 64% → 2-3 cái; N1 25% → 1-2 cái)

Output khuyến nghị ở dạng JSON file `questions.json`:

```json
{
  "questions": [
    {
      "label": "question_reference",
      "question": "①「自分たちのシステム」とあるが、何を指すか。",
      "answers": ["A option", "B option", "C option", "D option"],
      "correct": 2,
      "explain_vn": "...",
      "explain_en": "..."
    },
    {
      "label": "question_reason_explanation",
      "question": "筆者はなぜ...と考えているのか。",
      "answers": ["A", "B", "C", "D"],
      "correct": 3,
      "explain_vn": "...",
      "explain_en": "..."
    },
    {
      "label": "question_author_opinion",
      "question": "筆者が最も言いたいことはどれか。",
      "answers": ["A", "B", "C", "D"],
      "correct": 1,
      "explain_vn": "...",
      "explain_en": "..."
    }
  ]
}
```

### Bước 3 — Save HTML

Tên file: `{LEVEL}_{uuid4().hex}.html`. Ví dụ: `N1_a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6.html`.

```python
import uuid
filename = f"N1_{uuid.uuid4().hex}.html"
```

Save vào `assets/html/`.

### Bước 4 — Process + commit CSV (2 cách)

**Cách A (KHUYẾN NGHỊ) — JSON file chứa tất cả câu hỏi**:

```bash
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
    --file assets/html/N1_abc123...html \
    --csv sheets/samples_v1.csv \
    --tag "triết học ngôn ngữ" \
    --questions-json /tmp/qs.json
```

**Cách B — CLI flags (3-4 câu, prone to error)**:

```bash
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
    --file assets/html/N3_abc123...html \
    --csv sheets/samples_v1.csv \
    --tag "thí nghiệm thực vật" \
    --q1-label question_reference --q1 "①..." --a1 "A|B|C|D" --c1 2 --ev1 "..." --ee1 "..." \
    --q2-label question_reference --q2 "②..." --a2 "A|B|C|D" --c2 3 --ev2 "..." --ee2 "..." \
    --q3-label question_reason_explanation --q3 "なぜ..." --a3 "A|B|C|D" --c3 1 --ev3 "..." --ee3 "..." \
    --q4-label question_content_match --q4 "内容と合うのはどれか" --a4 "A|B|C|D" --c4 4 --ev4 "..." --ee4 "..."
```

Script sẽ:

- Count `jp_char_count` từ full HTML (skip `<rt>`, whitespace)
- Extract clean HTML (bỏ attribute, collapse whitespace, bỏ `<rt>`)
- **Validate level** (N1/N3 only) — block commit nếu N2/N4/N5
- **Validate số câu hỏi vs spec** (N1=3, N3=4) — warning
- **Validate ≥ 2 labels khác nhau** — warning (recommend ≥ 3 khác nhau cho N3)
- **Hard-reject** nếu dưới threshold (N1<900, N3<500) — exit 1, không commit CSV
- Cảnh báo nếu dưới Target Range hoặc vượt xa Target (> hi + 100)
- Upsert row vào CSV (45 columns) theo `_id` = filename, populate `question_1..question_{n}`

### Bước 5 — Validate batch

```bash
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
    --validate --html-dir assets/html
```

Exit 0 = tất cả pass; 1 = có file fail (UNDER_TARGET / HARD_REJECT / UNSUPPORTED_LEVEL).

### Bước 6 — Refresh sau khi edit HTML

Giữ câu hỏi cũ, chỉ refresh `jp_char_count` + `text_read`:

```bash
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
    --refresh --html-dir assets/html --csv sheets/samples_v1.csv
```

## 4. Target ranges & số câu (BẮT BUỘC)

| Level | Target Range | Hard Reject | Số câu/bài | Combo đề xuất |
|-------|--------------|-------------|-----------|---------------|
| N1    | 1000–1150    | < 900       | **3**     | reference + reason + **author_opinion** (câu cuối) |
| N3    | 550–700      | < 500       | **4**     | reference + reference + reason + **content_match** (câu cuối) |

> **Scope hẹp**: N2, N4, N5 KHÔNG có kind này. Nếu tệp `.html` mang tên `N2_*` đi qua pipeline, script sẽ báo `UNSUPPORTED_LEVEL` và KHÔNG commit vào CSV.

> **Data vs Spec**: Data N1 gốc đa số có 4 câu (74%), N3 đa số 4 câu (82%). Skill **LUÔN follow SPEC** — N1 = 3, N3 = 4. Bỏ qua noise trong data.

> **N3 target chọn hẹp**: Data N3 có spread 561–1344 (outlier), P25 = 598. Spec ~550. Skill chốt **550–700** (bài N3 quá dài dễ vượt tầm từ vựng học viên).

> **Câu cuối của bài** (best practice):
> - **N1 câu 3** nên là `question_author_opinion` (phù hợp spec "outline, main idea")
> - **N3 câu 4** nên là `question_content_match` hoặc `question_author_opinion` (tổng thể)

## 5. CSV Schema (45 columns)

Populate tương tự đoạn văn vừa nhưng nhiều câu hơn:

| Column | Value |
|--------|-------|
| `_id`  | `{LEVEL}_{uuid32hex}` (`LEVEL` ∈ {N1, N3}) |
| `level` | N1 hoặc N3 |
| `tag`  | Topic label |
| `jp_char_count` | Result `count_body_chars()` |
| `kind` | Always `đoạn văn dài` |
| `general_audio` | "" |
| `general_image` | "" (empty — no PNG) |
| `text_read` | Clean HTML |
| `text_read_vn` / `text_read_en` | "" |
| `question_label_1` | Label câu 1 |
| `question_1` | Câu hỏi 1 tiếng Nhật |
| `question_image_1` | "" |
| `answer_1` | `1. A\n2. B\n3. C\n4. D` |
| `correct_answer_1` | 1-4 |
| `explain_vn_1` / `explain_en_1` | Giải thích câu 1 |
| `question_label_2`..`explain_en_2` | Câu 2 (BẮT BUỘC) |
| `question_label_3`..`explain_en_3` | Câu 3 (BẮT BUỘC) |
| `question_label_4`..`explain_en_4` | Câu 4 (**CHỈ N3**) |
| `question_5` | **""** (dài tối đa 4 câu) |

## 6. Quality Gates

Trước khi coi 1 batch là xong:

- [ ] 100% file qua Hard Reject threshold
- [ ] ≥ 80% file nằm trong Target Range
- [ ] 100% file có `level` ∈ {N1, N3}
- [ ] **100% row có đúng số câu hỏi per level** (N1=3, N3=4)
- [ ] **Trong 1 bài, ≥ 2 `question_label` khác nhau** (ideally ≥ 3 với N3)
- [ ] **N1 câu cuối = `question_author_opinion`** (khuyến nghị mạnh)
- [ ] **N3 câu cuối = `question_content_match` hoặc `question_author_opinion`**
- [ ] Các câu hỏi trong 1 bài KHÔNG test cùng 1 đoạn (tách ra)
- [ ] Marker `①②③④` trong HTML khớp với câu hỏi reference
- [ ] Bài N1 có 5–8 paragraph, N3 có 4–6 paragraph
- [ ] Batch ≥ 3 bài có ≥ 2 `tag` khác nhau
- [ ] 0 file có furigana dạng "Ab" (cấm 週かん, 友だち)
- [ ] 100% file có `general_image = ""`
- [ ] 100% row có `kind = "đoạn văn dài"`
- [ ] 0 row có `question_5` non-empty
- [ ] Mỗi câu hỏi có 4 đáp án trong `answer_X`, 1 `correct_answer_X` là 1–4
- [ ] `explain_vn_X` + `explain_en_X` đều non-empty cho mọi câu

## 7. Edge cases & pitfalls

1. **Chỉ gen 2 câu hỏi** (như đoạn văn vừa N1/N2) — SAI, đoạn văn dài **BẮT BUỘC 3 câu (N1) / 4 câu (N3)**.
2. **Gen 4 câu cho N1** — SAI (dù data gốc 74% có 4 câu). Follow spec 3 câu.
3. **Gen bài level N2/N4/N5** — HARD-BLOCKED bởi script. Nếu muốn N2 dài, dùng skill `jlpt-reading-theme` (phase 4).
4. **Nội dung 2 câu hỏi trùng đoạn** — SAI, phải tách ra các đoạn khác nhau của bài.
5. **Marker ①②③ không match câu hỏi** — câu `①「cụm từ」とあるが` phải có `<span class="marker">①</span><u>cụm từ</u>` trong HTML.
6. **Char count dưới target** vì chỉ 3-4 paragraph — phải **5-8 paragraph** cho N1 và **4-6 paragraph** cho N3.
7. **Distractor quá dễ** — đoạn văn dài đòi hỏi distractor tinh vi (paraphrase sai, đối tượng nhầm, polarity đảo).
8. **Source line cho N3** — optional (data gốc chỉ 38%), KHÔNG bắt buộc. N1 thì nên có (70%).
9. **Annotation 注 cho N3** — **RẤT NÊN có** 2-3 cái (data 64%). N1 1-2 cái cũng phổ biến (25%).
10. **Data gốc có `<br>`** — KHÔNG bắt chước; dùng `<p>` thuần.
11. **Furigana quá nhiều** — data 0% có ruby, skill giới hạn tối đa 3 cặp (N1) / 6 cặp (N3), ưu tiên 0.
12. **Correct index conversion** — JSON `correctAnswer` là 0-based, CSV `correct_answer_X` là 1-based.
13. **Dạng "Ab" (週かん, 友だち)** — tuyệt đối không. Full kanji + ruby HOẶC full hiragana.
14. **Container width** = **780px** (rộng hơn 720 của vừa, 640 của ngắn). Đừng nhầm.
15. **Văn phong phức tạp cho N3** — N3 là trung cấp; câu quá phức tạp/trừu tượng sẽ fail. Dùng văn phong thư từ / essay đời sống.

## 8. Sample patterns theo data (tóm tắt, chi tiết xem `references/sample-analysis.md`)

| Pattern | N1 | N3 |
|---------|----|----|
| Char P25–P75 | 1045–1141 | 598–863 |
| Có `<u>` underline | **85%** | 50% |
| Có marker ①②③ | 29% | **79%** |
| Có 注 annotation | 25% | **64%** |
| Có source line | **70%** | 38% |
| Có `<ruby>` | **0%** | **0%** |
| Có fill_in_blank `[ ]` | 7% | 0% |
| Có `<br>` (KHÔNG mimic) | 29% | 52% |

**Key insights**:

- **N1 = essay/phê bình**: nhiều underline, source line cao, marker trung bình. Câu cuối luôn là `author_opinion` / outline tổng thể.
- **N3 = thư/tùy bút đời sống**: marker rất cao (79%), annotation rất cao (64%), thường có ①②③④ marker phân đoạn. Câu cuối = content_match tổng thể.
- **Cả 2 level không dùng ruby** — chỉ dùng khi từ trên mức level.

## 9. Status output của process_html.py

Khi chạy `--validate` hoặc `--refresh`, mỗi file được classify:

| Status | Nghĩa | Block commit? |
|--------|-------|---------------|
| `OK` | Trong target range | ❌ |
| `UNDER_TARGET` | Dưới target nhưng trên hard reject | ❌ (warning only) |
| `OVER_TARGET` | Vượt xa target (> hi + 100) | ❌ (warning only) |
| `HARD_REJECT` | Dưới threshold (N1<900, N3<500) | ✅ (exit 1) |
| `UNSUPPORTED_LEVEL` | Level không phải N1/N3 | ✅ (exit 1) |
| `UNKNOWN_LEVEL` | Tên file không match regex `^N[1-5]_[0-9a-f]{8,}\.html$` | ✅ (exit 1) |

## 10. Tương lai (Phase 4+)

- Phase 4: `đọc hiểu chủ đề` (N1/N2 ~1000, 3 câu so sánh chủ đề)
- Phase 5: `đọc hiểu tổng hợp` (N1/N2 ~600, 2 đoạn A+B, 2 câu so sánh view)
- Post-pipeline: bổ sung `text_read_vn` / `text_read_en` nếu cần bản dịch
- CSV consolidation: merge CSVs của 5 skill thành 1 `sheets/master.csv` để train

## 11. Liên hệ

- Skill owner: Nguyễn Đình Sinh <sinhnd@eupgroup.net>
- Phase trước: `../gen-question-doan-van-vua/` (đoạn văn vừa, 2-3 câu)
- Phase 1 reference: `../gen-question-doan-van-ngan/` (đoạn văn ngắn, 1 câu)
- Phase 0 reference: `../gen-question-jlpt/` (tìm thông tin, có screenshot)
- Master plan: `../PLAN_5_DANG_DOC_HIEU.md`
