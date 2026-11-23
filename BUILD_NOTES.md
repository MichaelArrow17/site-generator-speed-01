# site-generator-speed-01 — BUILD NOTES

## Key items

- **Competition:** site-generator-speed-01-run-20261122 — drop-in FASTER static site generator (solve.js) for makefun-site 1.0.0. Output must be byte-identical to the baseline on every case (200–600 Markdown files, 14–30 templates); judged on CPU time vs the house baseline at 95% confidence.
- **Prize:** $100 total, 10 places x $10. Free entry. **Closes Wed 25 Nov 04:00 WITA** (opened 22 Nov 20:00 UTC).
- **Anchor:** `makefun anchor 17a75502341fd8aa8ea5ff4cf5f5c1fa0498dd0b round 34204355` (kit init run in the empty repo before any write, in this fresh session started 23 Nov ~14:47 WITA, after the competition opened).
- **Kit:** `~/workspace/site-generator-kit-dl/kit.tar.gz`, sha256 `8f67c11c8e85f27fdb0efdc4ac194c9922314d1806c49295894b305c99b7e812` (42 files). Note: kit.json's `bundle_sha256` field says `2850d1b8…` — the platform rebuilt the kit after its morning 503 outage; the downloaded kit is the trusted one. Do NOT re-download (endpoint was 503 all morning).
- **Benchmark targets (live API 23 Nov ~08:33 WITA):** leader @aldii24 380,000 µs; #10 cutoff 590,000 µs; baseline ≈ 1,800,000 µs (from entry #8 claiming "3.2x CPU").
- **Result:** **~3.55x faster than baseline** (paired, 10/10 runs faster). Practice set: mine mean ~141 ms / median ~130 ms vs baseline ~500 ms (two paired runs: 137.6/124.3 and 144.2/136.3; baseline 529.4/471.3). Fresh unseen draws (E/F): 3.69x mean, 4.06x median, 10/10. (Pass 1 was 3.03x; pass 2 improved it ~17% — see "Pass 2" below.)
- **Server estimate (rough):** baseline server ≈ 1,800,000 µs vs local ~500 ms gives a ~3.6x server/local ratio, so mine lands ~500,000–510,000 µs server-equivalent — inside the top-10 cutoff (590,000 µs), near the ideal ≤ 500,000 µs, but well behind the leader (380,000 µs). Expect a top-10 place, probably not top 3.
- **Exactness:** practice 3/3 byte-identical vs expected.json AND vs vendor build; fresh generator draws 18/18 (A/B + C/D + E/F, all on the final build); adversarial fuzz 41,219 unit checks + 200 full-build mutations, **0 mismatches**.
- **grade-local:** 3/3 cases passed, L1 regrade reproduced byte for byte, result_sha256 `8f899b6fec293eddd8a721d415a6f4a33501189408c8cff30248fdaa18b6ddbb` (commit 897dd7f; local Node v24.22.1; server harness runs Node v26.13.1, so the sha cannot equal the server's).
- **Status:** build finished and verified (pass 1 + pass 2). **NOT submitted** — submit needs Deng Aji's review + a GitHub repo he creates manually (connector cannot create repos). No entry fee exists. **Recording note (pass 2):** live `makefun record` is REFUSED for this competition (HTTP 400 — it takes uploaded transcripts, not sealed recordings). Submit must bundle the build-1 session transcript + the pass-2 session transcript (`/home/hatch/agents/agent-a385d2e7-5fd8-42c5-af32-25a58ea4b06d/sessions/a385d2e7-5fd8-42c5-af32-25a58ea4b06d.jsonl`). No recording ID exists for this pass.

## Pass 2 (23 Nov, user-approved: squeeze more speed, or prove maxed out)

- **Before (pass 1, commit 086d5c7):** mine mean 170.4 ms / median 146.0 ms vs baseline 515.6 / 482.8 ms (3.03x). Fresh draws: 3.24x mean, 3.28x median.
- **After (pass 2, commit 897dd7f):** mine mean ~141 ms / median ~130 ms vs baseline ~500 ms (3.55x combined). Fresh draws E/F: mine mean 343.6 / median 312.2 vs baseline 1269.2 / 1268.4 (3.69x mean, 4.06x median, 10/10). **~17% faster than pass 1.**
- **What changed — commit e34fda7 (real gain):** `inline` rewritten: the per-call `flush` closure replaced by a module-level `pushEsc` helper, runs escaped with `escFast` (skips the `replace` when nothing needs escaping), and a plain-text fast path (`RE_SYNTAX` pre-test). Interleaved micro-bench over the 1,019 practice bodies: inline 42.5 → 35.9 ms (~16%). Paired practice timing: 155.8 → 143.8 ms mean.
- **What changed — commit 897dd7f (within noise, kept):** `renderNodes` `for` reuses one vars/child-scope pair per loop instead of two fresh objects per iteration (rendering is synchronous, so no iteration observes the next one's binding; `{% for loop in ... %}` keeps per-iteration objects, where the shorthand `loop` wins exactly as the reference's `{[name]: item, loop}` does). `pageOf` reuses the post's already-built card from `b.cards` instead of rebuilding it. Paired effect within noise (141.6 vs 143.8 ms mean; medians 132.2 vs 130.7); kept as an allocation/GC reduction, fully verified, not a claimed speedup.
- **What did NOT work:**
  - escapeHtml test-first fast path alone: 9.24 → 8.95 ms/iter over whole bodies (noise); a manual char-loop escape is much slower (21.7 vs 9.4 ms/iter, pass-1 data). Dropped as a standalone change.
  - wordsOf variants: table lookup 20.96, regex match-count 27.17, space/newline-only scan 19.74 vs current 20.16 ms/iter — all noise. Dropped.
  - inline plain-text fast path alone: 42.91 vs 42.51 ms interleaved (no gain). An early non-interleaved read showed 33% — a measurement artifact (GC/heap state), not real. The fast path stays inside the V2 rewrite (harmless, helps the no-syntax case); the V2 gain comes from the closure removal + escFast.
  - Profile at pass-2 start (self-time): escapeHtml 10.8% + its regex 5.2%, inline 10.1%, renderMarkdown 9.6%, GC 8.6%, wordsOf 7.8%, renderNodes 4.8%, render 3.6%, link 2.8%.
- **Verification (final commit 897dd7f):** practice exactness 3/3 vs expected + vendor; fresh draws 18/18 (A/B + C/D + E/F); rebuilt fuzzers 41,219 checks + 200 build mutations, 0 failures; grade-local 3/3, L1 regrade byte-for-byte, result_sha256 `8f899b6fec293eddd8a721d415a6f4a33501189408c8cff30248fdaa18b6ddbb`.
- **Verdict: IMPROVED, not maxed out — but the remaining gap to the leader (380,000 µs) is large.** Server estimate now ~500,000–510,000 µs vs cutoff 590,000 µs. Honest expectation unchanged: one $10 top-10 place, probably not top 3. Further squeezing would chase single-digit percents against a ~25% gap; stopped here per the stop rule (two no-gain attempts: for-scope reuse, pageOf card reuse).
- **Infra note:** the VM restarted mid-pass and wiped /tmp, including /tmp/sgs-bench (all pass-1 bench scripts, fresh draws A–D and their seeds). The repo was untouched. Bench scripts were rebuilt verbatim under `~/workspace/sgs-bench/` (persistent from now on): exact/paired/paired-fresh/fresh-diff + rebuilt fuzzers `fuzz2.mjs` + `fuzz-build2.mjs` (same coverage areas and counts as pass 1). Fresh draws E/F drawn with new seeds (recorded in `~/workspace/sgs-bench/fresh-seeds-pass2.txt`). All pass-2 verification above was re-run with the rebuilt scripts against the final commit.

## Design (solve.js, 38,836 bytes; main.ts is the unchanged house driver)

Full pipeline re-implementation with identical semantics to the vendored makefun-site 1.0.0:

- **Inline:** run-buffered — a native regex finds the next syntax char (`*`, backtick, `[`, `\`) instead of a per-character loop; plain runs are escaped with the regex `replace` (measured faster than a manual char loop: 11.1 ms vs 22.9 ms per iteration over all 1,019 practice bodies). Fresh regex per call (recursion-safe). `**` with no close skips both stars (first version re-processed the second `*` as an emphasis opener — caught by the inline fuzz, 109 mismatches, fixed). Pass 2: the per-call `flush` closure became a module-level `pushEsc`, runs use `escFast` (skips the replace when nothing needs escaping), and strings with no syntax char at all return `escFast(s)` directly.
- **Front matter:** indexOf-based, no whole-text split/join; CRLF fast path (`includes('\r')` check).
- **Markdown:** fused lexer+renderer (no token objects), cheap first-char dispatch; `blocks.join('\n') + '\n'` final join; list items accumulated in an array then joined.
- **Slugify:** memoized, with an ASCII fast path (NFKD of ASCII is identity).
- **Word count:** manual loop, no `\S+` regex match array; whitespace test reordered so letters (the common case) take two comparisons.
- **Templates:** array-join output; `render` builds one scope object per page.
- **Build-constant template cache:** templates proved (by a dependency analysis over parse trees, with for-binding tracking and include-cycle detection) to read only the build-constant top-level values `site`, `tags`, `recent` are rendered once per include depth and reused — but only when no enclosing for-loop binds a name the template reads (shadowing guard: for-scopes always hold `loop` in their vars; the top scope never does). In the practice cases this caches header, nav, footer and the filler partials (aside/badge/note/promo/strip) — 14/27 templates in g-0001, 10/22 in g-0002, 8/21 in g-0003. Page/post/index/tag/tag-list/templates are correctly excluded (they read `page`/`posts`/`tag`/`pager`).

## Verification evidence

All run locally with Node v24.22.1 and `--no-incremental-marking` (the flag main.ts sets). Scratch bench scripts live in `~/workspace/sgs-bench/` (not in the repo; rebuilt there in pass 2 after a VM restart wiped the old `/tmp/sgs-bench/`). Pass-2 verification is in the "Pass 2" section above.

- **Practice exactness** (`exact.mjs`): g-0001, g-0002, g-0003 — vs expected: true, vs vendor: true. 3/3.
- **Paired timing** (`paired.mjs`, 10 interleaved runs): mine ms 367 191 144 140 146 154 146 152 132 133; baseline ms 888 465 490 459 486 475 471 488 451 483. First run is a JIT warm-up outlier (367 ms); the harness runs 1 untimed warm-up round before 9 timed rounds, so warm-up is excluded there. Speedup 2.10x at v1 (mean 255.3 ms) → 3.03x at final (mean 170.4 ms).
- **grade-local** (`--kit . --bot <repo> --driver process`, docker not available on this VM): `status finished`, `scores {"cases_passed":3,"cases_total":3,"passed":1}`, checks 3/3, `L1 regrade reproduced byte for byte`, result_sha256 `8415ed809ede89447f45b7c54af225d588620ffac432d48f765edf3db7ffc71b`. Note: grade-local grades the COMMIT, not the worktree — solve.js + main.ts were committed first (commit 086d5c7).
- **Fresh generator draws:** ran the kit's own `program/generator/draw.ts` (Node 24 type-strips .ts) with two fresh random 64-hex seeds (recorded in `/tmp/sgs-bench/fresh-seeds.txt`), producing `/tmp/sgs-bench/fresh-a/` and `/tmp/sgs-bench/fresh-b/` (3 cases each). Differential vs expected.json and vs vendor build: **6/6 pass**.
- **Fresh-set paired timing** (`paired-fresh.mjs`, 10 runs over both fresh sets, 6 cases): mine mean 355.2 ms / median 336.6 ms; baseline mean 1,149.2 ms / median 1,105.7 ms. Speedup 3.24x mean, 3.28x median, faster 10/10.
- **Adversarial fuzz** (`fuzz.mjs`, seeded deterministic PRNG, all vs the vendor lib functions directly): inline 20,000 random strings (emphasis/code/link/escape/entity/unicode soup) — 0 failures; parseFrontMatter 5,000 — 0; renderMarkdown 5,000 random block docs — 0; slugify/longDate 6,000 — 0; parseTemplate + renderNodes 5,219 random templates (ifs, fors, includes incl. cycles, malformed tags, shadowing names like `{% for site in tags %}` and `{% for loop in posts %}`) — 0. **Total 41,219 checks, 0 failures.** (One intermediate run showed 94 template failures — that was a fuzzer bug: it passed `escape: undefined` because `escapeHtml` is not exported from solve.js; fixed by passing the vendor escape, which is verified identical via the inline/markdown fuzz.)
- **Build mutation fuzz** (`fuzz-build.mjs`): 200 mutated practice cases (adversarial bodies, templates, front matter, site fields) compared mine vs vendor build, canonical JSON — **0 failures**.

## Profile notes

- Baseline (3-case practice set): ~532 ms; hot spots escapeHtml 33.5%, inline 10.7%, renderMarkdown 10.4%, GC 7.0%, wordsOf + `\S+` regex 7.8%.
- Final mine: inline 9.9%, escapeHtml 12.4% + regex 3.7%, renderMarkdown 9.5%, renderNodes 13.4%, render 5.7%, wordsOf 8.7% (before the whitespace-test reorder; the reorder is included in the final timing above).
- The template cache moved template rendering from ~21% of total to a much smaller share; header/nav/footer/fillers render once per build instead of ~1,500 times per set.
- Pass 2 profile (self-time, before the inline change): escapeHtml 10.8% + its regex 5.2%, inline 10.1%, renderMarkdown 9.6%, GC 8.6%, wordsOf 7.8%, renderNodes 4.8%, render 3.6%, link 2.8%.

## Files

- `~/workspace/site-generator-speed-01/solve.js` — the entry (38,836 bytes).
- `~/workspace/site-generator-speed-01/main.ts` — house driver, unchanged from the baseline.
- `~/workspace/site-generator-speed-01/BUILD_NOTES.md` — this file.
- `~/workspace/site-generator-speed-01/LICENSE` — MIT (from kit init).
- Git log: `17a7550 makefun anchor: drand round 34204355` → `66eb317 LICENSE: MIT` → `086d5c7 Faster drop-in site generator…` → `7e7683c Add BUILD_NOTES.md…` → `e34fda7 Pass 2: faster inline…` → `897dd7f Pass 2: reuse for-scope objects…` (+ this file, to be committed).

## What remains (for Deng Aji / parent)

1. Deng Aji reviews this build (BUILD_NOTES.md + solve.js).
2. He creates a GitHub repo under MichaelArrow17 manually; PHANTOM pushes the repo and submits with the recordings.
3. Submit before Wed 25 Nov 04:00 WITA. No fee, no payment involved.
