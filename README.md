# site-generator-speed-01

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/MichaelArrow17/site-generator-speed-01/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/MichaelArrow17/site-generator-speed-01/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

Entry for the make.fun bounty **"Build a static site faster, 22 November 2026"**
([site-generator-speed-01-run-20261122](https://make.fun/bounty/site-generator-speed-01-run-20261122), $100).
A faster drop-in for make.fun's own makefun-site 1.0.0: `solve.js` answers every
case exactly as makefun-site 1.0.0 does — the same output tree (each page's
`index.html`, the index and tag pages, `build-report.txt`, and each static file),
byte for byte — and runs measurably faster.

## Results

- **Exact answers:** 3/3 practice cases byte-identical to makefun-site 1.0.0
  (vs `expected.json` and vs the vendor build); 18/18 on six freshly drawn
  generator sets; 0 mismatches on 41,219 adversarial fuzz checks plus 200
  full-build mutations.
- **Speed (local, Node v24.22.1, 10 paired runs over the practice set):**
  this entry mean ~141 ms vs makefun-site 1.0.0 mean ~500 ms — **~3.55x
  faster**; faster in every one of the 10 paired runs. Fresh unseen draws:
  3.69x mean, 4.06x median, 10/10.
- Official `grade-local` (`--driver process`): 3 of 3 points, L1 regrade
  reproduced byte for byte.

## How it works

Full pipeline re-implementation with identical semantics to makefun-site
1.0.0: run-buffered inline parsing (native regex finds the next syntax char,
plain runs escaped with a fast path that skips the replace when nothing needs
escaping), fused markdown lexer+renderer (no token objects), indexOf-based
front matter, memoized slugify with an ASCII fast path, manual word count,
array-join template output, plus a build-constant template cache — templates
proved to read only the build-constant values `site`, `tags`, `recent` are
rendered once per build instead of ~1,500 times per set. See `BUILD_NOTES.md`
for the full design and benchmark record.

## Files

| File | What it does |
| --- | --- |
| `solve.js` | The entry: dependency-free ES module, `run(input)`. |
| `main.ts` | House driver (unchanged): reads `/input/cases.json`, writes `/output/answers.json`. |
| `BUILD_NOTES.md` | Design, results, and benchmark record. |
| `LICENSE` | MIT. |
