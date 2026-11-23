[![make.fun bounty](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/jajanb923-netizen/site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/jajanb923-netizen/site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

# site-generator-speed-01: a faster drop-in for makefun-site 1.0.0

`run(input)` returns exactly what make.fun's own `makefun-site` 1.0.0 returns for
every input — the output tree of a generated site (each page's `index.html`, the
index and tag pages, `build-report.txt`, each static file), byte for byte — but
faster: about 4× the house baseline's speed on the practice set (steady state),
about 2× in a fresh process.

`main.ts` is the harness driver: it reads every case from `/input/cases.json`,
answers each with `run`, and writes `/output/answers.json`. `solve.js` is a
dependency-free ES module in the repository root.

## How it is faster

The semantics are makefun-site's own (its front matter, template dialect,
Markdown subset, slug rule and report — the reference code is the arbiter
wherever it differs from its README). The speed comes from doing the same work
with less of it:

- **Inline rendering batches plain text** and escapes it in bulk, instead of
  escaping one character at a time (the reference's dominant cost).
- **Markdown renders in one pass** over the lines, with no token objects and
  no repeated regex tests on the same line (cheap first-character guards).
- **Front matter is scanned with `indexOf`**, never splitting the whole body.
- **Slug and long-date results are memoised within one case** (each case is
  answered alone; nothing is cached from one case to the next).
- **Link resolutions are cached** per (source directory, target).
- **Template loops keep their scopes on a flat push/pop stack** instead of
  allocating a scope object per iteration.
- `\r\n` normalisation, word counting and escaping all fast-path the common
  case.

Nothing about the output changes: every rule is the reference's own, verified
below.

## Verification

- Practice set (`practice/input/cases.json` vs `practice/reference/expected.json`):
  **3/3 cases identical** to the oracle, byte for byte.
- Six further instances drawn from the kit's own generator with fresh seeds
  (18 cases): **18/18 identical** to the baseline and to the oracle.
- Ten hand-built edge cases (front-matter refusals of every flavour, Unicode
  slugs and collisions, Markdown torture, unparseable templates, include
  depth, loop shadowing, static shadowing, empty sites): **10/10 identical**
  to the baseline.

## Layout

- `solve.js` — the fast drop-in (`run(input)`).
- `main.ts` — the harness driver (the house baseline's driver, unchanged).
- `.makefun/owner.json` — the claim file naming the entry's wallet.
