# Miller for Mere Mortals

Most data that passes through a working
day arrives as a table: a CSV export from
some system, a JSON answer from some
service, a spreadsheet saved as text. The
classic Unix tools for tables — `cut`,
`sort`, `awk` — address columns by
*position*: field 1, field 3, field 5.
That works until the export grows a new
column, or someone reorders two of them,
and then every script you wrote quietly
starts reading the wrong numbers.
**Miller** is the same family of tools
rebuilt around *names*. Its command,
`mlr`, cuts, filters, sorts, summarizes,
joins and converts files whose columns
have names — CSV, JSON, TSV and a few
more formats — and it does so by field
name, so `quantity` stays `quantity`
wherever it happens to sit in the row.

This guide explains how that works: the
**record** and **verb** model Miller is
built on, your first commands against a
small orders file, computing new columns
with the `put` verb and its expression
language, summarizing with `stats1`,
joining two files on a shared key the
way a database would, converting between
CSV and JSON in both directions, and
chaining verbs into one command that
does a whole job. The surprises are
stated plainly, because Miller has a
few: arithmetic that produces
`9.600000000000001`, a filter that lets
the text `n/a` pass a "greater than 10"
test, and a variable that refuses to
remember anything from one row to the
next unless you spell its name with an
`@`. Each of those happened in this
guide's lab, and each is shown, not
hidden.

The lab behind this guide ran the real
thing: **Miller 6.22.0**, the official
release binary, on a small Linux machine,
against fixture files it made for the
purpose — a nine-row orders file, a
four-row customers file, a stock file
with one honest `n/a` in it, a headerless
readings file, and one deliberately
ragged file whose rows disagree about
how many fields they have. Against those
files the lab cut, filtered and sorted by
name; computed line totals and watched
floating point show its seams; ran sums
that silently stepped over the `n/a`;
joined the orders to the customers and
then went looking for the order the join
had dropped; converted CSV to JSON and
JSON back to CSV; fed Miller a ragged
file and read its refusal, then its
forgiveness; and built a running total
twice — once the awk way, which does not
work, and once Miller's way, which does.
Every transcript is kept in
`miller-lab/transcripts/`.

What the lab could not do, it did not
fake. Miller is one binary with no
server, no accounts and no graphical
interface, so there is less to leave out
than usual; what is labelled
documented-only here is the packaging
around it — the install routes for
operating systems the lab does not run,
and the characterizations of the
neighbouring tools in chapter 12, which
are described rather than benchmarked.
`criteria.md` keeps the full ledger of
what was run and what was only read.

## What this guide covers

- [**What Miller is**](what-miller-is.md) —
  the mental model: tables whose columns
  have names, and a tool that works with
  the names
- [**The moving parts**](the-moving-parts.md) —
  records, fields, verbs, format flags,
  and the expression language, and which
  part holds what
- [**Installing Miller**](installing-miller.md) —
  one static binary, the package routes,
  and checking you got the real thing
- [**Your first commands**](your-first-commands.md) —
  reading a file with `cat`, and `cut`,
  `filter` and `sort` by field name
- [**Computing with put**](computing-with-put.md) —
  new columns from old ones, the `.` that
  joins text, and the `(error)` that
  appears where arithmetic met a word
- [**Summarizing data**](summarizing-data.md) —
  `stats1` sums and averages, group-by,
  counting, and the numbers a summary
  quietly leaves out
- [**Joining two files**](joining-two-files.md) —
  matching orders to customers on a
  shared key, and the rows a join can
  silently drop
- [**Formats in, formats out**](formats-in-formats-out.md) —
  CSV to JSON and back, headerless files,
  and the ragged file that stops the line
- [**Chaining verbs**](chaining-verbs.md) —
  `then`: several verbs in one command,
  each working on the last one's output
- [**The expression language**](the-expression-language.md) —
  choices, text functions, built-in
  numbers like `NR`, and the `@`
  variables that remember across rows
- [**Recipes**](recipes.md) — a total per
  customer, a spreadsheet-ready extract,
  a JSON answer flattened for a report,
  a quick audit of a file you just received
- [**The neighbours**](the-neighbours.md) —
  `awk` and friends, `jq`, spreadsheets
  and the database: what each does that
  Miller doesn't, and vice versa
- [**Limits**](limits.md) — the files
  Miller cannot read, the errors it puts
  in your data instead of stopping, and
  where its road honestly ends
- [**Troubleshooting**](troubleshooting.md) —
  the run that read a CSV as something
  else, the sort that put 20 between 3
  and 2, the missing order, and the traps
  this guide's lab fell into so you
  don't have to
- [**Glossary**](glossary.md) — the
  terms, collected in one place

## What it doesn't cover

Miller as a programming platform: the
expression language has user-defined
functions, maps and loops, and chapter 10
shows its everyday core, but a full
language reference is the project's own
documentation's job, and this guide
points there rather than competing with
it. Files too big for your machine's
patience, databases that should stay
databases, and data cleaning as a
profession — deciding which of two
spellings is the true one — are likewise
out. This guide stays with one person, a
terminal, and files that have already
arrived.

## How to use this guide

Read chapter 1 for the model and
chapter 2 before running anything —
knowing which part holds what turns
later confusion into a named problem.
Chapter 4 runs your first commands in
the same sitting as the install, against
the lab's own orders file, which ships
in `miller-lab/data/` so you can repeat
every command in this guide and check
your output against the transcripts in
`miller-lab/transcripts/`. Chapter 5 is
where the tool starts earning its keep,
and chapter 10 is the one to read slowly:
the difference between a variable that
forgets and a variable that remembers is
one character, and it is the character
most newcomers get wrong.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [jq for Mere Mortals](https://paulsullivan33-dev.github.io/jq-for-mere-mortals/)
- [yq for Mere Mortals](https://paulsullivan33-dev.github.io/yq-for-mere-mortals/)
- [Text Processing for Mere Mortals](https://paulsullivan33-dev.github.io/text-processing-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/miller-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/miller-for-mere-mortals/blob/main/LICENSE-CODE).
