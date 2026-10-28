# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   The lab's files are invented for the purpose.
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 13 names what the tool
   cannot do: formats it cannot read, the errors it stores in
   your data instead of stopping for, comparisons that pass
   text through a numeric gate, strict CSV shapes, whole-file
   jobs, flat records against nested data, and machine numbers
   wearing money's clothes.
8. **The audit rule is in the text, early.** Chapters 5 and 6
   state the rule plainly — *output you have not counted and
   swept is a draft, not a result* — before any recipe depends
   on it, and chapters 11 and 14 turn it into procedures:
   counts before and after, an `is_numeric` sweep, an
   `is_error` hunt.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is a small Ubuntu 24.04 sandbox. Miller
**6.22.0** was taken from the project's official release
archive for linux-amd64 and run as a bare binary — no install
into the system, no services. All fixture files live in
`miller-lab/data/` and all transcripts in
`miller-lab/transcripts/`; every command quoted in the
chapters was run against those fixtures with that binary.

Run for real, with transcripts as evidence:

- Install and version from the release tarball (chapter 3;
  the binary's own `--version` answer).
- The flagless run that reads a CSV as DKVP and numbers its
  fields, then `--csv cat` and `--icsv --opprint cat` reading
  it properly (t00, t01).
- `cut -f`, `filter` with a numeric comparison, and
  `sort -f` with `-nr`, on the nine-row orders file (t02).
- `put` arithmetic (`$quantity * $unit_price`) including the
  genuine floating-point tails `9.600000000000001` and
  `4.800000000000001`, text joining with `.`, and the
  `fmtnum` remedy (t03, t06b); the discovery that `round`
  refuses a second argument (t06).
- The `(error)` value produced by arithmetic on the text
  `n/a` and by `+` on words, with the run still exiting 0
  (t04); the failed first attempt to hunt errors by comparing
  against the text `"(error)"`, and the working
  `is_error` / `!is_numeric` tests (t15).
- `filter '$count > 10'` passing the `n/a` row by
  type-ordered comparison, and the corrected
  `is_numeric(...) && ...` form returning only numeric rows
  (t04, t18).
- `stats1` sums, means and counts, grouped and ungrouped,
  including the sum that silently steps over `n/a`
  (57 = 40 + 17), `count`, `uniq -c -g`, and `rename`
  (t05, t13).
- `join` on the customer key: nine orders in, eight rows
  out, eli's order dropped; `--ul` adding nothing on this
  data; `--ur` under CSV output dying with the schema-change
  error after partial output; `--ur` under JSON and DKVP
  output surfacing the unpaired record (t06, t06b, t06c).
- Conversions both ways: CSV to JSON array and to JSON
  Lines, JSON to CSV, CSV to TSV (t07b); the first,
  failed attempt without a verb, kept as evidence for
  chapter 14 (t07).
- Ragged input: the default refusal mid-stream with exit 1,
  and `--allow-ragged-csv-input` passing rows through —
  short row padded, long row keeping its extra field (t08).
- Headerless input: `--implicit-csv-header` naming fields
  1, 2, 3, a `put` computing over `$2` with boolean output,
  `label` giving the columns real names, and `--nidx`
  treating the comma file as one wide whitespace-split
  field (t09, t16).
- Chains with `then`: filter–cut–sort (including the lab's
  own first attempt with `-r` alone, which sorted the
  quantities as text — 20 between 3 and 2 — kept in t10),
  put–stats1–sort with sorting on the computed summary
  field, and `sort` then `head` (t10, t13); both recipes
  whose chains had not been run before writing were run
  before the chapter stood (t17).
- The expression language: ternary choices, `toupper`,
  `sub`, `NR` (t11); the running total that fails with a
  plain local variable and succeeds with an out-of-stream
  `@` variable, the `begin`/`end` blocks, `emit`, and
  `put -q` collapsing the program to a grand total that
  agrees with `stats1`'s independent sum (t12b, t12c);
  the parse failures that taught the semicolon rule and
  the refusal of `+=` inside parentheses (t12, and the
  script-file session behind chapter 10).

Documented only — described from the project's documentation,
not run, and the text does not claim otherwise:

- Install routes other than the release tarball: package
  managers (Homebrew, distribution packages, Chocolatey,
  winget) and building from source with Go (chapter 3).
- JSON Lines as an *input* format (its output side was run,
  t07b), and Miller's deeper flattening options for nested
  JSON beyond the flat case (chapters 8 and 13).
- The expression language beyond its everyday core:
  user-defined functions, maps and loops are pointed to in
  the project's documentation, not taught (chapter 10).
- The comparison tools in chapter 12 are characterised, not
  benchmarked.

Fixtures are this guide's own and deliberately generic: a
nine-row orders file for an unnamed shop, a four-row
customers file with invented first names and three cities, a
stock file with one honestly missing count, a three-line
readings log, and a ragged file broken on purpose. One order
(1009, customer `eli`) is deliberately absent from the
customers file so the join chapter can show a real dropped
record. No example is modelled on any real person, network,
service or organisation.
