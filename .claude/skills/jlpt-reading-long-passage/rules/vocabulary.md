# Rules: Từ vựng, Ngữ pháp & Furigana (R3, R4)

> **Scope**: Đoạn văn dài (長文 / long-passage) — **CHỈ N1 & N3**.

## R3. Trình độ kiến thức (Kanji, Từ vựng, Ngữ pháp)

### Nguyên tắc tổng quát

| Level | Kanji/Từ vựng cốt lõi | Ngữ pháp cốt lõi |
|-------|----------------------|-------------------|
| **N3** | ~650 kanji + ~3700 từ | ～について, ～によって, ～ため, ～にとって, ～ばかり, ～わけではない |
| **N1** | ~2000 kanji + ~10000 từ | ～いかんによらず, ～をもって, ～に先立ち, ～にもかかわらず, keigo cao |

### Golden principle — "THAY TỪ, KHÔNG RẮC FURIGANA"

Nếu bài N3 dùng quá nhiều kanji N2 → **viết lại bằng từ N3**, không phải rắc furigana bừa bãi cho từ N2.

**Furigana dùng khi không có cách nào thay thế khác** — ví dụ tên riêng, từ chuyên môn không có từ N3 tương đương (`哲学`, `遺伝子`, `振動` ở N3).

### Phân loại từ trong bài

1. **Từ khóa của bài** (key terms): từ trung tâm mà câu hỏi xoay quanh — **BẮT BUỘC** ở đúng level bài, KHÔNG cần furigana.
   VD: Trong bài N3 về 実験 (thí nghiệm) → chính từ 実験 phải là N3, không rắc furigana.

2. **Từ ngữ cảnh** (context words): từ phụ hỗ trợ, hay xuất hiện (家族, 学校, 食べる, 思う) → giữ đúng level.

3. **Từ chuyên môn / thuật ngữ** (jargon): từ không thể thay thế, vượt level → **có thể thêm furigana** hoặc **thêm 注 annotation** giải thích.

### Ruby count density per level (đoạn văn dài)

| Level | Above-level words | Ruby `<ruby>` expected |
|-------|-------------------|------------------------|
| **N3** | 0–8               | 0–15                    |
| **N1** | 0–3               | 0–6                     |

> **Lưu ý đặc thù đoạn văn dài**: Data gốc **N1/N3 ruby = 0%** — đề thi thực tế gần như KHÔNG dùng furigana. Skill vẫn cho phép furigana cho từ vượt level, NHƯNG **khuyến nghị giữ ít (0-3 cặp)**, ưu tiên 0.

> **Nguyên tắc**: ≥ 80% từ vựng/ngữ pháp phải ở đúng level. Ruby chỉ cho phần vượt level không thể tránh.

---

## R4. Furigana — Quy tắc & Kanji lookup

### R4.1 Compound Word Rule — CẤM dạng "Ab"

**LUÔN viết nguyên bộ kanji** rồi đặt furigana bao toàn bộ. **TUYỆT ĐỐI KHÔNG** tách nửa kanji nửa hiragana.

Chỉ chọn 1 trong 2:

1. **Full kanji + furigana**: `<ruby>構築<rt>こうちく</rt></ruby>`
2. **Full hiragana** (khi ở level thấp): `こうちく`

**❌ CẤM**: `構ちく`, `友だち`, `拠てん`, `経けん`

**✅ Ngoại lệ Okurigana**: `<ruby>届<rt>とど</rt></ruby>く` — furigana chỉ phủ kanji, okurigana đứng riêng ngoài ruby.

### R4.2 Furigana Lookup Procedure

Bước 1: **Xác định level kanji** bằng `rules/jlpt_kanji.csv` (2150 kanji, mapped từ N5→N1).

Bước 2: **Nếu kanji > level bài** → thêm furigana:

```html
<ruby>普遍的<rt>ふへんてき</rt></ruby>
```

Bước 3: **Nếu kanji ≤ level bài** → KHÔNG thêm furigana.

### R4.3 Ví dụ áp dụng

| Từ | Level kanji | Bài N3 | Bài N1 |
|----|-------------|--------|--------|
| 哲学 (てつがく) | N1 | `<ruby>哲学<rt>てつがく</rt></ruby>` | 哲学 (không furigana) |
| 普遍的 (ふへんてき) | N2 | `<ruby>普遍的<rt>ふへんてき</rt></ruby>` | 普遍的 (không furigana) |
| 振動 (しんどう) | N2 | `<ruby>振動<rt>しんどう</rt></ruby>` | 振動 (không furigana) |
| 経験 (けいけん) | N3 | 経験 (không furigana) | 経験 (không furigana) |
| 社会 (しゃかい) | N4 | 社会 (không furigana) | 社会 (không furigana) |

### R4.4 Furigana cho name riêng

Tên người/địa danh trong bài → **thêm furigana** nếu kanji có thể đọc nhiều cách:

- `<ruby>山口和彦<rt>やまぐちかずひこ</rt></ruby>`
- `<ruby>田中由美<rt>たなかゆみ</rt></ruby>`

Ở N3, tên đọc phổ biến có thể không cần furigana (`山田`, `佐藤`).

### R4.5 Số lượng ruby cho đoạn văn dài

Bài dài 550-1150 chars, nhưng data thực tế **N1/N3 ruby = 0%** — skill giữ ruby ít nhất có thể.

**Dấu hiệu rắc furigana sai**:
- Bài N1 có > 6 ruby tag → chắc chắn sai level bài (bài N1 nên đã dùng kanji cao cấp)
- Bài N3 có > 15 ruby tag → chắc chắn sai level bài (viết lại bằng từ N3 thay vì rắc ruby)
- Có ruby cho từ cơ bản như `食<rt>た</rt>べる`, `見<rt>み</rt>る`, `行<rt>い</rt>く` ở bài N3 → thừa
- Ruby tag kéo dài hơn 6 ký tự hiragana → từ quá vượt level, nên thay từ khác

### R4.6 Annotation `注` vs Furigana

Chọn một:

- **Furigana** cho từ có reading khó nhưng nghĩa dễ đoán từ ngữ cảnh
- **注 annotation** cho từ chuyên môn cần giải thích nghĩa

Ví dụ:
- `<ruby>憧れ<rt>あこがれ</rt></ruby>` → furigana OK (nghĩa dễ đoán)
- `普遍的` N2 term trong bài N3 → thêm `注1 普遍的：どこにでも共通してあてはまる様子` → không cần furigana cho 的

Cả hai cùng lúc OK nếu cần:
```html
<ruby>普遍的<rt>ふへんてき</rt></ruby>（注1）
...
<div class="annotations">
    <p>注1　普遍的：どこにでも共通してあてはまる様子</p>
</div>
```

> **Lưu ý đoạn văn dài**: N3 **khuyến nghị dùng `注` hơn furigana** — data N3 annotation = 64% (rất cao) vs ruby = 0%. Viết thuật ngữ ở kanji full + thêm 注 giải thích nghĩa thay vì rắc furigana cho reading.
