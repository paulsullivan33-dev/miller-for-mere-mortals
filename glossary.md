# Glossary

**Accumulator** — a statistic `stats1`
computes, such as `sum`, `mean` or
`count`, named with `-a`.

**Chain** — several verbs in one `mlr`
command, joined by `then`, each
working on the records the previous
verb produced.

**DKVP** — delimited key-value pairs,
Miller's own loose format
(`quantity=3,product=pencil`) and its
default when no format is declared.

**DSL** — Miller's built-in expression
language, used by `put` and `filter`:
"domain-specific language".

**Field** — one named value in a
record: `quantity` is 3.

**Format flag** — the declaration of
what a file is and what the output
should be: `--csv`, `--ijson`,
`--otsv`, and their kin.

**Group-by** — the `-g` option that
makes a summary produce one record
per distinct value of a field.

**Header** — the first row of a CSV
or TSV file, holding the field names.

**Held file** — in a `join`, the file
named with `-f`, loaded whole and
kept in memory while the other file
streams past it.

**JSON Lines** — the `--jsonl`
format: one self-contained JSON
object per line, no surrounding
array.

**Local variable** — a bare name in
the expression language, alive only
for the record currently being
processed.

**Out-of-stream variable** — a name
spelled with `@`, living outside the
record stream and persisting from
record to record, for running totals
and counters.

**PPrint** — `--pprint`/`--opprint`,
the aligned text table Miller draws
for human readers.

**Ragged** — input whose rows
disagree with the header, or with
each other, about how many fields a
record has.

**Record** — one row of the table as
Miller holds it: a set of fields,
each a name and a value.

**Type inference** — Miller's
reading of values: text that spells a
number is held as a number, text that
does not stays text.

**Type-ordered comparison** — the
rule that values of different kinds
compare by kind first, so every text
value stands after every number in a
`>` test.

**Verb** — the operation a Miller
command performs: `cut`, `filter`,
`sort`, `put`, `join`, `stats1`, and
dozens more.
