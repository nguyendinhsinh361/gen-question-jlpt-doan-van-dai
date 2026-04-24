# HTML Patterns — Đoạn Văn Dài

Cụ thể hoá template HTML cho N1 (essay dài) và N3 (essay/thư dài). Spec đoạn văn dài: container 780px, padding 56px 64px, line-height 1.95; paragraph 5-8 (N1) hoặc 4-6 (N3); source line rất phổ biến.

## 1. Base HTML Template (áp dụng cả N1 & N3)

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[Japanese title ngắn]</title>
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
        .passage p { margin: 0 0 1em 0; text-indent: 1em; }
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
    <!-- content here -->
</div>
</body>
</html>
```

## 2. N1 Template — Tiểu luận / phê bình cao cấp (5-8 paragraph, 1000-1150 chars)

Cấu trúc essay chuẩn:

```
<p>① Introduction — nêu vấn đề/luận điểm chính</p>
<p>② Supporting evidence 1 — có thể chứa marker ① cho Q1 reference</p>
<p>③ Supporting evidence 2 — logic chain</p>
<p>④ Counter-argument hoặc nuance — có thể chứa marker ② cho Q2 reference</p>
<p>⑤ Synthesis / implication</p>
<p>⑥ Conclusion — tác giả chốt luận điểm (Q3 author_opinion test ở đây)</p>
<div class="annotations"><p>注1 ...</p></div>  <!-- 25% data có -->
<p class="source">（[fake author]「[fake title]」による）</p>  <!-- 70% data có -->
```

Ví dụ cụ thể N1 — chủ đề ngôn ngữ:

```html
<div class="passage">
    <p>私たちは日常的に言葉を使っているが、言葉そのものについて立ち止まって考えることは意外に少ない。しかし、言葉は単なる伝達の道具ではなく、私たちの思考そのものを形作る枠組みでもある。このことを深く認識することは、自分自身の思考を豊かにする第一歩である。</p>
    <p>ある言語学者が指摘しているように、私たちが世界を認識する仕方は、使用している言語の構造によって大きく左右される。例えば、色の名前の数が少ない言語を使う人々は、その文化圏で区別される色の数も少ないという研究がある。<span class="marker">①</span><u>このような現象</u>は、言葉と思考が密接に結びついていることを示している。</p>
    <p>では、複数の言語を話せる人は、複数の思考様式を持つことができるのだろうか。この問いに対する答えは、単純ではない。確かに、言語によって表現できるニュアンスは異なるが、人間の認識能力そのものは普遍的であるとも言える。つまり、言語は思考の一部を規定するが、思考のすべてを支配するわけではない。</p>
    <p>一方で、現代社会においては、単純化された言葉が蔓延しているという問題がある。短いメッセージ、分かりやすい表現、即座に理解できる言葉ばかりが求められ、複雑な概念を表現する豊かな語彙は徐々に失われつつある。<span class="marker">②</span><u>この傾向</u>は、長期的には社会全体の思考の質を低下させる恐れがある。</p>
    <p>なぜなら、複雑な思考は複雑な言葉を必要とするからだ。概念を正確に捉えるには、適切な語彙が不可欠である。語彙が貧しくなれば、思考もまた貧しくなる。これは単に知的な損失にとどまらず、民主主義や文化全体にも影響を及ぼす深刻な問題である。</p>
    <p>したがって、私たちは意識的に言葉を大切にし、豊かな語彙を保持する努力を続けなければならない。それは単なる学問的な課題ではなく、人間として生きるための根本的な責務である。言葉を守ることは、思考を守ることであり、ひいては自分自身を守ることに他ならない。</p>
    <div class="annotations">
        <p>注1　普遍的（ふへんてき）：どこにでも共通すること</p>
    </div>
    <p class="source">（山口和彦「言葉と思考の未来」による）</p>
</div>
```

→ Q1 (reference, marker ①): `「①このような現象」とあるが、何を指しているか。`
→ Q2 (reason_explanation): `筆者はなぜ複雑な言葉が必要だと考えているのか。`
→ Q3 (author_opinion): `この文章で筆者が最も伝えたいことはどれか。`

## 3. N3 Template — Thư/essay có logic (4-6 paragraph, 550-700 chars)

Cấu trúc:

```
<p>① Introduction — set context (có thể thư mở đầu "〇〇さん、お元気ですか")</p>
<p>② Main story / event (marker ① cho Q1)</p>
<p>③ Consequence / feeling (marker ② cho Q2)</p>
<p>④ Reflection / lesson (Q3 reason)</p>
<p>⑤ Conclusion (Q4 content_match)</p>
<div class="annotations"><p>注1 ...</p><p>注2 ...</p></div>  <!-- 64% data có -->
<p class="source">（fake author「fake title」による）</p>  <!-- 38% data có -->
```

Ví dụ cụ thể N3 — chủ đề thí nghiệm/đời sống:

```html
<div class="passage">
    <p>先週、テレビで面白い実験の結果を紹介していました。ある大学の研究チームが、植物にも感情のようなものがあるかどうかを調べる実験をしたのです。</p>
    <p>実験では、同じ部屋に置いた二つの同じ植物のうち、一つには毎日「ありがとう」「きれいだね」と優しい言葉をかけ、もう一つには「つまらない」「消えろ」と否定的な言葉をかけたそうです。一か月後、<span class="marker">①</span><u>驚くべき結果</u>が出ました。優しい言葉をかけられた植物は元気に育ち、花まで咲かせたのに対し、否定的な言葉をかけられた植物はほとんど育たず、葉が黄色くなってしまったのです。</p>
    <p>この結果について、研究者は次のように説明しています。植物が人間の言葉を理解しているわけではないが、話しかけるときの振動や空気の流れ、また話しかけた人の表情や扱い方が、植物の成長に影響を与えているのではないか、と。②つまり、言葉そのものではなく、言葉を発する人の態度が大切だということです。</p>
    <p>私はこの話を聞いて、人間同士の関係も同じだと思いました。私たちは日々、家族や友人、同僚に対してさまざまな言葉をかけています。その言葉が相手の心にどんな影響を与えているか、改めて考えさせられました。</p>
    <p>言葉は道具であると同時に、心そのものを映す鏡でもあります。優しい言葉をかけること、思いやりを持って話すこと——それは、相手だけでなく、自分自身の心も育てることにつながるのかもしれません。</p>
    <div class="annotations">
        <p>注1　振動（しんどう）：ゆれること</p>
        <p>注2　思いやり：人の気持ちを考える気持ち</p>
    </div>
</div>
```

→ Q1 (reference, marker ①): `①「驚くべき結果」とあるが、どのような結果か。`
→ Q2 (reference, marker ②): `②「つまり、言葉そのものではなく、言葉を発する人の態度が大切だ」とあるが、どういう意味か。`
→ Q3 (reason_explanation): `筆者は、なぜ人間同士の関係も同じだと思ったのか。`
→ Q4 (content_match): `この文章の内容と合っているものはどれか。`

## 4. Marker Strategy Cho 3-4 Câu Hỏi

Với bài dài, các câu hỏi nên bám các đoạn/cụm khác nhau. Marker ①②③ đặt rải đều.

### Pattern 1 — N1 (3 câu): 2 markers + 1 tổng kết

```html
<p>... <span class="marker">①</span><u>cụm A</u> ...</p>
<p>... <span class="marker">②</span><u>cụm B</u> ...</p>
<p>... [đoạn cuối không marker — Q3 test toàn bài author_opinion]</p>
```

### Pattern 2 — N1 (3 câu): 1 marker + 1 reason + 1 tổng kết

```html
<p>... <span class="marker">①</span><u>cụm A</u> ...</p>
<p>... [đoạn lý do — Q2 reason] ...</p>
<p>... [kết luận — Q3 author_opinion] ...</p>
```

### Pattern 3 — N3 (4 câu): 2 markers + 1 reason + 1 tổng kết (phổ biến nhất)

```html
<p>... <span class="marker">①</span><u>cụm A</u> ...</p>
<p>... <span class="marker">②</span><u>cụm B</u> ...</p>
<p>... [đoạn lý do — Q3] ...</p>
<p>... [kết luận — Q4 content_match] ...</p>
```

### Pattern 4 — N3 (4 câu): 3 markers + 1 tổng kết

```html
<p>... <span class="marker">①</span><u>cụm A</u> ...</p>
<p>... <span class="marker">②</span><u>cụm B</u> ...</p>
<p>... <span class="marker">③</span><u>cụm C</u> ...</p>
<p>... [kết luận — Q4 author_opinion] ...</p>
```

## 5. Source Line Rules

| Level | Data rate | Khuyến nghị |
|-------|-----------|-------------|
| N1    | 70%       | **RẤT NÊN** thêm (đặc trưng N1 dài) |
| N3    | 38%       | Optional, thêm khi bài là essay/phê bình |

Format: `（[fake author]「[fake title]」による）`

- Author: tên Nhật 2-4 chữ, tự chế (ví dụ: 山口和彦, 田中由美, 佐藤健一)
- Title: cụm Nhật ngắn, liên quan chủ đề (ví dụ: 「言葉と思考の未来」, 「日々の小さな発見」)

Examples:
- `（山口和彦「言葉と思考の未来」による）`
- `（田中由美「現代社会のゆくえ」より）`
- `（佐藤健一「日々の小さな発見」による。一部改変）`

## 6. Annotation 注 Rules

| Level | Data rate | Khuyến nghị |
|-------|-----------|-------------|
| N1    | 25%       | 1-2 annotation (cho thuật ngữ rare) |
| N3    | 64%       | **RẤT NÊN** thêm 2-3 annotation |

Format:
```html
<div class="annotations">
    <p>注1　単語（よみ）：意味</p>
    <p>注2　単語：意味</p>
</div>
```

Hoặc định dạng có marker inline:
```html
<p>...言葉（注1）が持つ力...</p>
...
<div class="annotations">
    <p>注1　言葉の持つ力：言葉が人や物事に影響を与える力</p>
</div>
```

## 7. Visual Elements Reference

| Element | N1 rate | N3 rate | Khi nào dùng |
|---------|---------|---------|--------------|
| `<u>` | 85% | 50% | MỌI cụm từ được hỏi (reference/meaning) |
| `<span class="marker">①</span>` | 29% | 79% | Đặt ngay trước cụm từ được hỏi, mỗi câu reference 1 marker |
| `<div class="annotations">` | 25% | **64%** | Khi có thuật ngữ khó — rất khuyến nghị N3 |
| `<p class="source">` | **70%** | 38% | Cuối bài — đặc trưng essay/tiểu luận |
| `<br>` | 29% | 52% | **KHÔNG dùng** (data có nhưng skill bắt `<p>` thuần) |
| `<ruby>/<rt>` | 0% | 0% | Chỉ khi thực sự cần — max 3-6 cặp |
| `[　]` blank | 7% | 0% | Gần như không dùng ở đoạn văn dài |

## 8. Paragraph Count Guidelines

| Level | Paragraph count | Lý do |
|-------|----------------|-------|
| N1    | **5–8**         | Bài 1000-1150 chars, essay có multiple sections |
| N3    | **4–6**         | Bài 550-700 chars, story/essay có flow đơn giản |

Nếu paragraph quá ít (≤3) ở bài 1000+ chars → bài quá đặc, khó đọc. Chia nhỏ theo đoạn ý tưởng.

## 9. Cheatsheet Cho Gen Agent

```
N1 (3 câu, 1000-1150 chars):
  ✓ 5-8 paragraph
  ✓ 1-2 marker trong HTML
  ✓ source line (RẤT NÊN)
  ✓ 1-2 annotation (optional)
  ✓ Câu cuối = author_opinion
  ✗ KHÔNG <br>, KHÔNG blank [ ]
  ✗ KHÔNG ruby nhiều (0-3 cặp)

N3 (4 câu, 550-700 chars):
  ✓ 4-6 paragraph
  ✓ 2-3 marker trong HTML (79% data)
  ✓ 2-3 annotation (RẤT NÊN)
  ✓ source line (optional)
  ✓ Câu cuối = content_match hoặc author_opinion
  ✗ KHÔNG <br>, KHÔNG blank [ ]
  ✗ KHÔNG ruby nhiều (0-6 cặp)
```

## 10. Common Mistakes

1. **Bài chỉ 3 paragraph** → quá đặc → chia thành 5-8 đoạn (N1) hoặc 4-6 đoạn (N3)
2. **N1 câu cuối là reference** → SAI, phải là author_opinion
3. **4 câu N3 cùng test 1 đoạn** → SAI, mỗi câu 1 đoạn riêng
4. **Marker ①②③ không có trong HTML** → sai, phải match câu hỏi
5. **Thiếu source line ở N1** → giảm authenticity (data 70%)
6. **N3 không có annotation** → giảm authenticity (data 64%)
7. **Dùng `<br>`** → sai, dùng `<p>` thuần
8. **Ruby quá nhiều** → đoạn văn dài thực tế gần như không có ruby
