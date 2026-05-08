# Prompt — Gen bài Đoạn Văn Dài (JLPT 長文読解)

## Cách dùng

Copy prompt bên dưới, thay `{số}` rồi paste vào Claude hoặc Gemini.

**⛔ Scope chỉ 2 level**: **N1** (3 câu, ~1000–1150 chars) và **N3** (4 câu, ~550–700 chars). N2/N4/N5 KHÔNG có.

> **🚨 ZERO-TOLERANCE WORKFLOW**: SKILL.md có **5 GATE bắt buộc** (0→1, 1→2, 2→3, 3→4, 4→5). Mỗi gate phải log `GATE X→Y PASSED`. **1 mục FAIL = sửa/gen lại → QC TỪ ĐẦU**, đến khi 32/32 PASS mới hoàn thành.

---

## Prompt

```
Đọc .claude/skills/jlpt-reading-long-passage/SKILL.md rồi gen bài đọc hiểu đoạn văn dài:
- N3: {số} bài (4 câu, ~550–700 chars)
- N1: {số} bài (3 câu, ~1000–1150 chars)

⛔ CHỈ N1 và N3. Lưu CSV: sheets/samples_v1.csv. HTML: assets/html/doan_van_dai/{LEVEL}_{uuid}.html.

🔒 5 GATE bắt buộc — KHÔNG QUA = KHÔNG SANG BƯỚC TIẾP. Log explicit GATE X→Y PASSED.

═══ BƯỚC 0 — CHUẨN BỊ (1 lần) → GATE 0→1 ═══
Đọc đầy đủ:
- rules/rule_doc_hieu.md (Phần 2.4 thể chia toàn 普通形 N1/N3, Phần 5 — 7 loại bẫy + **Peripheral Source cho N1 với 注 dài**, Phần 8.3 N3 長文 + Phần 10.3 N1 長文)
- rules/{content,vocabulary,technical,questions}.md + rules/kanji_jlpt_sensei.csv
- Load 2 sample/level: scripts/load_references.py --level {N1|N3} --count 2
- Scan sheets/samples_v1.csv
GATE 0→1: tick 6/6 → log "GATE 0→1 PASSED".

═══ BƯỚC 1 — GEN HTML + 3-4 Q+A → GATE 1→2 ═══
1. _id = {LEVEL}_{uuid32}
2. Tag = **tiếng Anh** từ rules/topic.json (philosophy, science...)
3. Gen HTML: marker ①②③ khớp Q reference, **toàn 普通形**, furigana chỉ vượt level (cấm "Ab")
   - N1: source line tự chế (CẤM 朝日/読売/村上春樹...). N1 nên có 1-2 注 — tiếng Nhật đơn giản
   - N3 thường KHÔNG có source
4. Gen Q + 4 đáp án; ≥ 2 unique label per bài; câu cuối nên là author_opinion/content_match (test thesis, KHÔNG marker); distractor ≥ 3 loại bẫy + Peripheral Source kiểm tra cho N1
5. Tạo CSV bằng process_html.py + fill_qa.py (KHÔNG sửa CSV tay)
GATE 1→2: tick 5/5 → log "GATE 1→2 PASSED".

═══ BƯỚC 2-3 — QC 32 MỤC → GATE 2→3 + GATE 3→4 ═══
GATE 2→3: cam kết check ĐẦY ĐỦ 32 mục → log "GATE 2→3 PASSED".
Đánh giá 32 mục: A. HTML 11 + B. Content 6 + C. Q&A 12 (label, paraphrase ≥4 từ N1 / ≥5 N3, explain VN+EN, self-solve khớp) + D. Multi-Q Coverage 3.
**Peripheral Source self-check cho N1**: distractor có lấy info từ 注 không? Nếu có và Q không phải về 注 → REJECT.
GATE 3→4: liệt kê FAIL + diagnosis → log "GATE 3→4 PASSED".

═══ BƯỚC 4-5 — SỬA + LẶP → GATE 4→5 ═══
- Fix HTML → `--refresh`. Fix Q&A → fill_qa.py
- ≥ 50% FAIL / self-solve FAIL / char Hard Reject → **GEN LẠI** (giữ _id)
- Quay lại GATE 2→3 → QC 32/32 TỪ ĐẦU
- Tối đa 5 vòng → báo user
GATE 4→5: 32/32 PASS + --validate clean → log "🎉 ALL PASSED (32/32) + GATE 4→5 PASSED" → bài tiếp.

═══ HARD REJECT (gen lại ngay) ═══
- Q count sai: N1=3 (slot 4-5 empty), N3=4 (slot 5 empty)
- Char range ngoài: N1 1000–1150 | N3 550–700
- 2 câu cùng test 1 đoạn; câu cuối KHÔNG test thesis; marker không khớp Q hoặc dư
- <ruby> thiếu <rt>; furigana "Ab"; thể chia có です・ます (cả N1 và N3 phải toàn 普通形)
- N1 dùng tên thật cho source; 注 dùng tiếng Anh/Việt
- Tag tiếng Việt/Nhật; trùng topic; <2 unique label per bài

═══ CUỐI BATCH ═══
python3 .claude/skills/jlpt-reading-long-passage/scripts/process_html.py --validate --html-dir assets/html/doan_van_dai --csv sheets/samples_v1.csv
```
