# COMP 370/570 Homework 4 – Explore My Little Pony Dialog

Dataset: `clean_dialog.csv` from [My Little Pony Transcript (Kaggle)](https://www.kaggle.com/liury123/my-little-pony-transcript).

Commands below were run from the `hw4/` directory. `csvtool` was used for column extraction; `grep`, `wc`, `head`, `sort`, `uniq`, and `file` were used for counting and inspection.

Because the original file is Windows-1252 (not UTF-8) and uses CRLF line endings, a UTF-8 copy was made for reliable `csvtool` use:

```bash
iconv -f CP1252 -t UTF-8 clean_dialog.csv > clean_dialog_utf8.csv
```

---

## How big is the dataset?

```bash
ls -lh clean_dialog.csv
wc -c clean_dialog.csv
wc -l clean_dialog.csv
csvtool height clean_dialog_utf8.csv
```

**Result:**

| Measure | Value |
|---|---|
| File size | ~4.7 MB (4,896,994 bytes) |
| Total lines (`wc -l`) | 36,860 |
| Rows (`csvtool height`) | 36,860 (1 header + 36,859 dialog lines) |

---

## What’s the structure of the data?

```bash
head -1 clean_dialog.csv
csvtool width clean_dialog_utf8.csv
csvtool col 1,2,4 clean_dialog_utf8.csv | head -5
```

**Fields (4 columns):**

| Field | Description | Example values |
|---|---|---|
| `title` | Episode title | `Friendship is Magic, part 1`, `Winter Wrap Up` |
| `writer` | Episode writer(s) | `Lauren Faust`, `Amy Keating Rogers` |
| `dialog` | Spoken line text | Narrative / dialogue strings (often with a leading space) |
| `pony` | Speaker label | `Twilight Sparkle`, `Narrator`, `Spike`, … |

Sample (title, writer, pony):

```text
title,writer,pony
"Friendship is Magic, part 1",Lauren Faust,Narrator
"Friendship is Magic, part 1",Lauren Faust,Narrator
"Friendship is Magic, part 1",Lauren Faust,Narrator and Twilight Sparkle
"Friendship is Magic, part 1",Lauren Faust,Twilight Sparkle
```

There are **843** distinct speaker labels and **66** distinct writer strings.

```bash
csvtool col 4 clean_dialog_utf8.csv | sed 's/^"//;s/"$//' | tail -n +2 | sort -u | wc -l
csvtool col 2 clean_dialog_utf8.csv | sed 's/^"//;s/"$//' | tail -n +2 | sort -u | wc -l
```

---

## How many episodes does it cover?

```bash
csvtool col 1 clean_dialog_utf8.csv | sed 's/^"//;s/"$//' | tail -n +2 | sort -u | wc -l
```

**Result: 197 unique episode titles.**

---

## Unexpected aspect of the dataset

Several properties looked surprising and could complicate later analysis. The most important:

### 1. Speaker labels are not always a single character (biggest analysis risk)

Many rows use multi-speaker or variant labels instead of a single main-pony name, for example:

- `Narrator and Twilight Sparkle`
- `Pinkie Pie and Rarity`
- `Fluttershy and Rainbow Dash`
- `Mean Twilight Sparkle`, `Young Rainbow Dash`, `Pinkie Pie 2`

```bash
csvtool col 4 clean_dialog_utf8.csv | sed 's/^"//;s/"$//' | grep ' and ' | sort | uniq -c | sort -rn | head
```

There are **294** speaker labels containing ` and `. Exact-name counts for the Mane characters therefore **undercount** lines where that pony speaks jointly with someone else, and **exclude** “Mean/Young/Future/…” variants. Any later analysis that assumes `pony == "Twilight Sparkle"` is a complete Twilight speech act will be biased.

### 2. Related issues found during exploration

- **Encoding:** `file clean_dialog.csv` reports non-ISO extended-ASCII (Windows-1252). Reading as UTF-8 fails (e.g. byte `0x97`).
- **Line endings:** CRLF (`\r\n`), so end-of-line `grep` patterns need care.
- **Dialog formatting:** most `dialog` values begin with a leading space.
- **Aggregated speaker:** `Others` appears **979** times as a single speaker bucket.

---

## Task 4: Main pony speaker frequency

Extract the speaker column, then count exact matches with `grep`:

```bash
csvtool col 4 clean_dialog_utf8.csv | sed 's/^"//;s/"$//' | tail -n +2 > pony_column.txt
TOTAL=$(wc -l < pony_column.txt)   # 36859

grep -c '^Twilight Sparkle$' pony_column.txt
grep -c '^Rarity$' pony_column.txt
grep -c '^Pinkie Pie$' pony_column.txt
grep -c '^Rainbow Dash$' pony_column.txt
grep -c '^Fluttershy$' pony_column.txt
```

Percent of all lines: `100 * count / 36859` (rounded to 2 decimal places).

| pony_name | total_line_count | percent_all_lines |
|---|---:|---:|
| Twilight Sparkle | 4745 | 12.87 |
| Rainbow Dash | 3072 | 8.33 |
| Pinkie Pie | 2833 | 7.69 |
| Rarity | 2660 | 7.22 |
| Fluttershy | 2109 | 5.72 |

See `Line_percentages.csv` for the submission table.
