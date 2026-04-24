# Rules: HTML Template, Clean HTML, CSV Schema (R9, R10, R11)

> **Scope**: Đoạn văn dài (long-passage). **KHÔNG có screenshot PNG** — `general_image` luôn empty. **CHỈ N1 và N3**. Mỗi bài có **3-4 câu hỏi** (N1=3, N3=4) populate vào cùng 1 row CSV.

## R9. HTML Template

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{JP_TITLE_DÀI}</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700&display=swap');
        body {
            font-family: 'Noto Sans JP', sans-serif;
            background: #f9fafb;
            color: #111827;
            line-height: 1.95;
            word-break: keep-all;
            line-break: strict;
            overflow-wrap: break-word;
            margin: 0;
            padding: 40px 20px;
        }
        .passage {
            max-width: 780px;
            margin: 0 auto;
            background: white;
            padding: 56px 64px;
            border: 1px solid #e5e7eb;
            border-radius: 6px;
            font-size: 16px;
        }
        .passage p {
            margin: 0 0 1em 0;
            text-indent: 1em;
        }
        .passage .no-indent { text-indent: 0; }
        .marker { font-weight: bold; color: #1e40af; }
        .annotations {
            margin-top: 2em;
            padding-top: 1em;
            border-top: 1px dashed #d1d5db;
            font-size: 0.9em;
            color: #374151;
            line-height: 1.7;
        }
        .annotations p { margin: 0.3em 0; text-indent: 0; }
        .source {
            margin-top: 1.2em;
            text-align: right;
            font-size: 0.88em;
            color: #4b5563;
            text-indent: 0;
        }
        ruby { ruby-align: center; ruby-position: over; vertical-align: baseline; }
        ruby rt { font-size: 0.55em; color: #374151; letter-spacing: 0.02em; line-height: 1; vertical-align: top; }
        u { text-decoration: underline; text-decoration-thickness: 1.5px; }
    </style>
</head>
<body>
<div class="passage">
    {BODY_CONTENT}
</div>
</body>
</html>
```

### Key specs (MUST match exactly)

| Element | Value |
|---------|-------|
| Container width | `max-width: 780px` |
| Container margin | `margin: 0 auto` (căn giữa) |
| Container padding | `56px 64px` |
| Body line-height | `1.95` |
| Background body | `#f9fafb` (light gray) |
| Container background | `#fff` với `border: 1px solid #e5e7eb` |
| `word-break` | `keep-all` (xuống dòng ở ranh giới từ) |
| `text-align` | default (left) — KHÔNG justify |
| Tailwind CDN | ❌ KHÔNG dùng — CSS inline hết |
| Screenshot PNG | ❌ KHÔNG có |

### Template N1 — Essay dài 5-8 paragraph, 3 câu hỏi, có source line

```html
<div class="passage">
    <p>私たちは日常的に言葉を使っているが、言葉そのものについて立ち止まって考えることは意外に少ない。しかし、言葉は単なる伝達の道具ではなく、私たちの思考そのものを形作る枠組みでもある。このことを深く認識することは、自分自身の思考を豊かにする第一歩である。</p>
    <p>ある言語学者が指摘しているように、私たちが世界を認識する仕方は、使用している言語の構造によって大きく左右される。例えば、色の名前の数が少ない言語を使う人々は、その文化圏で区別される色の数も少ないという研究がある。<span class="marker">①</span><u>このような現象</u>は、言葉と思考が密接に結びついていることを示している。</p>
    <p>では、複数の言語を話せる人は、複数の思考様式を持つことができるのだろうか。この問いに対する答えは、単純ではない。確かに、言語によって表現できるニュアンスは異なるが、人間の認識能力そのものは普遍的であるとも言える。</p>
    <p>一方で、現代社会においては、単純化された言葉が蔓延しているという問題がある。短いメッセージ、分かりやすい表現、即座に理解できる言葉ばかりが求められ、複雑な概念を表現する豊かな語彙は徐々に失われつつある。<span class="marker">②</span><u>この傾向</u>は、長期的には社会全体の思考の質を低下させる恐れがある。</p>
    <p>なぜなら、複雑な思考は複雑な言葉を必要とするからだ。概念を正確に捉えるには、適切な語彙が不可欠である。語彙が貧しくなれば、思考もまた貧しくなる。これは単に知的な損失にとどまらず、民主主義や文化全体にも影響を及ぼす深刻な問題である。</p>
    <p>したがって、私たちは意識的に言葉を大切にし、豊かな語彙を保持する努力を続けなければならない。言葉を守ることは、思考を守ることであり、ひいては自分自身を守ることに他ならない。</p>
    <div class="annotations">
        <p>注1　普遍的（ふへんてき）：どこにでも共通してあてはまる様子</p>
    </div>
    <p class="source">（山口和彦「言葉と思考の未来」による）</p>
</div>
```

**Question 1** (`question_reference`): 「①このような現象」とあるが、何を指しているか。
**Question 2** (`question_reference`): 「②この傾向」とあるが、どのような傾向か。
**Question 3** (`question_author_opinion`): この文章で筆者が最も伝えたいことはどれか。

### Template N3 — Essay/story 4-6 paragraph, 4 câu hỏi, có annotation

```html
<div class="passage">
    <p>先週、テレビで面白い実験の結果を紹介していました。ある大学の研究チームが、植物にも感情のようなものがあるかどうかを調べる実験をしたのです。</p>
    <p>実験では、同じ部屋に置いた二つの同じ植物のうち、一つには毎日「ありがとう」「きれいだね」と優しい言葉をかけ、もう一つには「つまらない」「消えろ」と否定的な言葉をかけたそうです。一か月後、<span class="marker">①</span><u>驚くべき結果</u>が出ました。優しい言葉をかけられた植物は元気に育ち、花まで咲かせたのに対し、否定的な言葉をかけられた植物はほとんど育たず、葉が黄色くなってしまったのです。</p>
    <p>この結果について、研究者は次のように説明しています。植物が人間の言葉を理解しているわけではないが、話しかけるときの<ruby>振動<rt>しんどう</rt></ruby>（注1）や空気の流れ、また話しかけた人の表情や扱い方が、植物の成長に影響を与えているのではないか、と。<span class="marker">②</span><u>つまり、言葉そのものではなく、言葉を発する人の態度が大切だということです</u>。</p>
    <p>私はこの話を聞いて、人間同士の関係も同じだと思いました。私たちは日々、家族や友人、同僚に対してさまざまな言葉をかけています。その言葉が相手の心にどんな影響を与えているか、改めて考えさせられました。</p>
    <p>言葉は道具であると同時に、心そのものを映す鏡でもあります。優しい言葉をかけること、思いやりを持って話すこと——それは、相手だけでなく、自分自身の心も育てることにつながるのかもしれません。</p>
    <div class="annotations">
        <p>注1　振動：ゆれること</p>
        <p>注2　思いやり：人の気持ちを考える気持ち</p>
    </div>
</div>
```

**Question 1** (`question_reference`): ①「驚くべき結果」とあるが、どのような結果か。
**Question 2** (`question_meaning_interpretation`): ②「つまり、言葉そのものではなく、言葉を発する人の態度が大切だ」とあるが、どういう意味か。
**Question 3** (`question_reason_explanation`): 筆者は、なぜ人間同士の関係も同じだと思ったのか。
**Question 4** (`question_content_match`): この文章の内容と合っているものはどれか。

Chi tiết template per level + marker strategy xem `references/html-patterns.md`.

---

## R10. Clean HTML

### Clean HTML (`text_read`)

Strip all attributes, classes, and excess whitespace cho CSV column `text_read`. Clean HTML **GIỮ** `<ruby>` tag nhưng **BỎ** nội dung `<rt>` — nghĩa là chỉ có kanji gốc + okurigana, không có furigana trong CSV.

```python
class CleanHTMLExtractor(HTMLParser):
    SKIP_TAGS = ('style', 'script', 'rt')   # bỏ furigana trong CSV text_read
    def __init__(self):
        super().__init__()
        self.result, self.skip_depth = [], 0
        self.in_body, self.body_done = False, False
    def handle_starttag(self, tag, attrs):
        if tag == 'body': self.in_body = True; return
        if not self.in_body or self.body_done: return
        if tag in self.SKIP_TAGS: self.skip_depth += 1; return
        if self.skip_depth > 0: return
        self.result.append(f'<{tag}>')
    def handle_endtag(self, tag):
        if tag == 'body': self.body_done = True; return
        if not self.in_body or self.body_done: return
        if tag in self.SKIP_TAGS: self.skip_depth -= 1; return
        if self.skip_depth > 0: return
        self.result.append(f'</{tag}>')
    def handle_data(self, data):
        if not self.in_body or self.body_done or self.skip_depth > 0: return
        self.result.append(data)

def clean_html(full_html):
    ext = CleanHTMLExtractor()
    ext.feed(full_html)
    raw = ''.join(ext.result)
    raw = re.sub(r'\s+', ' ', raw)
    raw = re.sub(r'\s*<', '<', raw)
    raw = re.sub(r'>\s*', '>', raw)
    raw = re.sub(r'<(\w+)></\1>', '', raw)
    return raw.strip()
```

### Character count (`count_body_chars()`)

```python
from html.parser import HTMLParser
import re

class BodyTextExtractor(HTMLParser):
    def __init__(self):
        super().__init__()
        self.texts, self.skip_depth, self.in_body = [], 0, False
    def handle_starttag(self, tag, attrs):
        if tag == 'body': self.in_body = True
        if tag in ('rt', 'style', 'script'): self.skip_depth += 1
    def handle_endtag(self, tag):
        if tag in ('rt', 'style', 'script'): self.skip_depth -= 1
    def handle_data(self, d):
        if self.in_body and self.skip_depth == 0: self.texts.append(d)

def count_body_chars(html_string):
    ext = BodyTextExtractor()
    ext.feed(html_string)
    return len(re.sub(r'[ \t\n\r\u3000]', '', ''.join(ext.texts)))
```

Rules:
- Count từ **full HTML file**, KHÔNG phải clean HTML.
- Skip `<rt>` (furigana), `<style>`, `<script>` content.
- Remove all whitespace: space, tab, newline, full-width space (`　`).
- Numbers, punctuation, Latin chars ALL count.

### ⛔ KHÔNG có screenshot

Đoạn văn dài KHÔNG chụp PNG. CSV column `general_image` LUÔN empty string (`""`). Không cần Playwright / viewport / `screenshot.py` — skill này KHÔNG bundle screenshot script.

---

## R11. CSV Schema & File Naming

45 columns matching `rules/question_sheet.csv`:

| Column | Value cho đoạn văn dài |
|--------|------------------------|
| `_id` | `{LEVEL}_{uuid.uuid4().hex}` — 32-char hex |
| `level` | **`N1` hoặc `N3` only** (N2/N4/N5 không apply) |
| `tag` | Topic tiếng Việt từ `rules/topic.json` (VD: `triết học`, `phê bình văn hóa`, `đời sống`) |
| `jp_char_count` | Result of `count_body_chars()` |
| `kind` | **Always `đoạn văn dài`** |
| `general_audio` | `""` (empty) |
| `general_image` | **`""` (empty — KHÔNG có PNG)** |
| `text_read` | Clean HTML (no attributes, no `<rt>` content, collapsed whitespace) |
| `text_read_vn` | `""` (empty) |
| `text_read_en` | `""` (empty) |

### Câu hỏi — số lượng tùy level

| Level | Cần populate | Cần empty |
|-------|--------------|-----------|
| **N1** | `question_{1,2,3}` | `question_{4,5}` |
| **N3** | `question_{1,2,3,4}` | `question_{5}` |

Với mỗi slot `i` được populate:

| Field | Value |
|-------|-------|
| `question_label_i` | Một trong 7 labels (xem `rules/questions.md` R5) |
| `question_i` | Câu hỏi tiếng Nhật |
| `question_image_i` | `""` (empty) |
| `answer_i` | 4 options ngăn cách `\n`, KHÔNG số thứ tự |
| `correct_answer_i` | Integer `1`-`4` |
| `explain_vn_i` | Giải thích VN 3 phần (xem `rules/questions.md` R6) |
| `explain_en_i` | Giải thích EN 3 phần (cùng nội dung với VN) |

Với mỗi slot `j` KHÔNG dùng (j > n): tất cả 7 cột `question_label_j`, `question_j`, `question_image_j`, `answer_j`, `correct_answer_j`, `explain_vn_j`, `explain_en_j` = `""`.

### File Naming

All files và CSV `_id` column dùng cùng ID: `{LEVEL}_{uuid}`

- **Pattern**: `{LEVEL}_{uuid}.html` — e.g. `N1_a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5.html`
- **Level prefix UPPERCASE**: `N1`, `N3` only
- **UUID**: 32-char hex từ `uuid.uuid4().hex` (full, không cắt)
- **_id in CSV** = cùng value = filename without extension

```python
import uuid
def gen_id(level: str) -> str:
    assert level in ("N1", "N3"), "Đoạn văn dài chỉ support N1 và N3"
    return f"{level}_{uuid.uuid4().hex}"
```

### ⛔ KHÔNG BAO GIỜ sửa CSV bằng tay

> **Nội dung câu hỏi thường chứa commas (ví dụ `100,000円`, `それは、そうだ`) + newlines trong `answer_i` → sẽ vỡ cột CSV khi mở bằng text editor thô.**
> **LUÔN dùng `scripts/fill_qa.py`** để điền Q&A — script tự quote đúng mọi trường hợp, tự validate label có `question_` prefix, tự validate đúng số câu theo level.
> Hoặc dùng `scripts/process_html.py` với `--questions-json` (khuyến nghị cho 3-4 câu).

---

## QC Automation Scripts

```python
import re
from pathlib import Path

def check_html(html_path: str, level: str) -> dict:
    """Kiểm tra tự động các tiêu chí đo được cho đoạn văn dài."""
    html = Path(html_path).read_text(encoding='utf-8')
    results = {}

    # TC1: Character count (min AND max)
    char_count = count_body_chars(html)
    char_range = {"N1": (1000, 1150), "N3": (550, 700)}
    hard_reject = {"N1": 900, "N3": 500}
    if level not in char_range:
        results["TC1_chars"] = {"pass": False, "error": f"Level {level} không thuộc scope đoạn văn dài"}
        return results
    lo, hi = char_range[level]
    results["TC1_chars"] = {
        "count": char_count, "range": f"{lo}-{hi}",
        "pass": lo <= char_count <= hi + 50,
        "hard_reject": char_count < hard_reject[level]
    }

    # TC2: Flow text (no 。<br>)
    br_in_prose = len(re.findall(r'。\s*<br\s*/?>', html))
    results["TC2_flow_text"] = {"br_in_prose": br_in_prose, "pass": br_in_prose == 0}

    # TC3: Container CSS (780px cho đoạn văn dài)
    has_max_width = bool(re.search(r'max-width:\s*780px', html))
    has_margin_auto = bool(re.search(r'margin:\s*0\s+auto', html))
    results["TC3_container"] = {"pass": has_max_width and has_margin_auto}

    # TC4a: Ruby count per level
    ruby_count = len(re.findall(r'<ruby>', html))
    ruby_max = {"N1": 6, "N3": 15}
    results["TC4_ruby_count"] = {"count": ruby_count, "max": ruby_max[level],
                                 "pass": ruby_count <= ruby_max[level] + 2}

    # TC4b: Wrong furigana format (check parentheses)
    paren_furigana = re.findall(r'[\u4e00-\u9fff]+[(][ぁ-ん]+[)]', html)
    bracket_furigana = re.findall(r'[\u4e00-\u9fff]+【[ぁ-ん]+】', html)
    results["TC4_furigana_format"] = {
        "paren_found": paren_furigana,
        "bracket_found": bracket_furigana,
        "pass": len(paren_furigana) == 0 and len(bracket_furigana) == 0
    }

    # TC4c: Ruby without rt
    ruby_blocks = re.findall(r'<ruby>(.*?)</ruby>', html, re.DOTALL)
    ruby_without_rt = [b for b in ruby_blocks if '<rt>' not in b]
    results["TC4_ruby_has_rt"] = {
        "missing_rt": ruby_without_rt,
        "pass": len(ruby_without_rt) == 0
    }

    # TC5: Paragraph count
    p_count = len(re.findall(r'<p(?:\s[^>]*)?>', html))
    p_range = {"N1": (5, 10), "N3": (4, 8)}
    lo, hi = p_range[level]
    results["TC5_paragraphs"] = {"count": p_count, "expected": f"{lo}-{hi}",
                                 "pass": lo <= p_count <= hi + 2}

    # TC6: Marker consistency — nếu có marker ①②③ thì phải có câu hỏi reference tương ứng
    markers = re.findall(r'<span class="marker">([①②③④])</span>', html)
    results["TC6_markers"] = {"count": len(markers), "markers": markers}

    return results


def check_csv_row(csv_row: dict, level: str) -> dict:
    """Kiểm tra row CSV cho đoạn văn dài — số câu hỏi đúng level."""
    expected_q = {"N1": 3, "N3": 4}[level]
    filled = sum(1 for i in range(1, 6) if csv_row.get(f"question_{i}", "").strip())
    empty_beyond = all(
        csv_row.get(f"question_{i}", "") == ""
        for i in range(expected_q + 1, 6)
    )
    # Rule đặc biệt: câu cuối phải là author_opinion (N1) hoặc content_match/author_opinion (N3)
    last_label = csv_row.get(f"question_label_{expected_q}", "")
    if level == "N1":
        last_label_ok = last_label == "question_author_opinion"
    else:  # N3
        last_label_ok = last_label in ("question_content_match", "question_author_opinion")
    return {
        "question_count": filled,
        "expected": expected_q,
        "count_pass": filled == expected_q,
        "empty_beyond_pass": empty_beyond,
        "kind_pass": csv_row.get("kind") == "đoạn văn dài",
        "general_image_pass": csv_row.get("general_image") == "",
        "last_label": last_label,
        "last_label_pass": last_label_ok,
    }
```

---

## Bundled Scripts

```bash
# Đếm ký tự:
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py --count-only --file <html-file>

# Validate batch (Target Range + broken ruby):
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_dai

# Full pipeline qua JSON (RECOMMENDED cho 3-4 câu):
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
    --file <html-file> \
    --csv sheets/samples_v1.csv \
    --tag "triết học" \
    --questions-json /tmp/qs.json

# Safe Q&A filling (khuyến khích cho agent — tránh vỡ CSV):
python3 .claude/skills/jlpt-reading-long-passage/scripts/fill_qa.py \
    --csv sheets/samples_v1.csv \
    --row-id N1_abcdef... \
    --level N1 \
    --q1-label question_reference \
    --q1 "..." \
    --a1 "Option 1
Option 2
Option 3
Option 4" \
    --ca1 2 \
    --evn1 "..." \
    --een1 "..." \
    --q2-label question_reason_explanation \
    --q2 "..." \
    --a2 "..." \
    --ca2 3 \
    --evn2 "..." \
    --een2 "..." \
    --q3-label question_author_opinion \
    --q3 "..." \
    --a3 "..." \
    --ca3 1 \
    --evn3 "..." \
    --een3 "..."

# Refresh CSV sau khi sửa HTML (giữ Q&A):
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py \
    --refresh \
    --file assets/html/doan_van_dai/{LEVEL}_{uuid}.html \
    --csv sheets/samples_v1.csv

# Load reference samples để calibrate style:
python3 .claude/skills/jlpt-reading-long-passage/scripts/load_references.py --level N1 --count 2
```
