# Troubleshooting

Symptoms, in the order a newcomer
meets them. Every one of these
happened in this guide's lab; the
transcripts in `miller-lab/transcripts/`
hold the evidence.

## The output has numbered fields: 1=..., 2=...

**Cause:** the format flag is missing
or wrong, and Miller read the file in
its default DKVP format, numbering
what it could not name (chapter 4,
transcript `t00`).

**Fix:** say what the file is —
`--csv`, `--tsv`, `--ijson` — before
the verb. Then re-check with
`--opprint cat` that the header row
became field names.

## "verb ... not found"

**Cause:** a filename landed where a
verb belongs — classically a format
conversion attempted with flags alone
and the file read as the command
(transcript `t07`, first attempt).

**Fix:** conversions are verbs too.
Put a verb between the flags and the
file: `mlr --csv --ojson cat file.csv`.
The same error with a real verb name
in it means a spelling: `mlr -l` lists
the verbs that exist.

## The sort looks drunk: 20 between 3 and 2

**Cause:** numeric fields sorted as
text — `-r` without `-n`
(chapter 4, transcript `t10`'s first
chain). As characters, `20` sorts
before `3` because `2` < `3`.

**Fix:** `-nr` for descending numbers,
`-n` for ascending. Text fields neither
need nor want it.

## (error) appears in a computed column

**Cause:** the expression met a value
of the wrong kind — text where a
number was required, usually an `n/a`,
a dash, or an empty cell in arithmetic
(chapter 5, transcript `t04`).

**Fix:** find them with
`filter 'is_error($field)'`
(transcript `t15`), then decide: fix
the source, exclude the rows with
`is_numeric` before computing, or
accept them — as a decision, written
down, not as a surprise in the report.

## The filter kept rows it should have dropped

**Cause:** type-ordered comparison —
text outranks every number, so
`$count > 10` passes `n/a`
(chapter 6, transcript `t04`).

**Fix:** `filter 'is_numeric($count)
&& $count > 10'`. Any numeric gate on
a column you have not audited needs
the `is_numeric` half.

## The summary is smaller than the file deserves

**Cause:** `stats1` builds statistics
from numeric values only and steps
over the rest silently (chapter 6).

**Fix:** compare the accumulator's
`count` (as in `quantity_count`)
against `mlr --csv count` on the file.
A gap is unaudited data; chapter 11's
audit recipe finds the rows.

## The join came out short

**Cause:** streamed records with no
partner in the held file are dropped
by a plain (inner) join (chapter 7,
transcript `t06` — nine in, eight
out).

**Fix:** count both sides first. To
see the dropped records, `--ur` (or
`--ul` for the held file's) — and
write them in JSON or DKVP, because
under CSV the next symptom is yours.

## "CSV schema change" and the run dies mid-file

**Cause:** records of a second shape
reached a CSV writer that had already
written a header for the first shape —
unpaired join records, an `end`-block
`emit`, or ragged input forgiven at
the door (chapters 7, 8, 10).

**Fix:** emit that stream as `--ojson`
or `--odkvp`, or make the shapes agree
before the CSV step. Remember the
error arrives *after* partial output:
anything downstream of the failure has
seen a truncated file.

## "CSV header/data length mismatch", exit 1

**Cause:** a row with fewer or more
fields than the header — genuinely
ragged input (chapter 8, transcript
`t08`).

**Fix:** repair the file if the
raggedness is corruption. If it is
the file's nature, `--allow-ragged-csv-input`
passes rows through — short rows
padded with empties, long rows keeping
their extra fields — and the shape
problem becomes yours, deliberately.

## The running total is just the column repeated

**Cause:** a plain variable in
`begin`/the main block is a *local*,
forgotten after each record — the awk
habit (chapter 10, transcript `t12b`).

**Fix:** memory across records wears
an `@`: `begin { @t = 0 } @t +=
$quantity; $running = @t`
(transcript `t12c`).

## "cannot parse DSL expression"

**Cause, three usual suspects:**
statements not separated by semicolons
(even across lines, chapter 10); an
`if` body without its curly braces; or
a compound assignment used inside
parentheses as a sub-expression, which
the grammar refuses where a statement
would accept it.

**Fix:** lay the expression out in a
`-f` file, semicolons ending every
statement, braces on every body, and
let the error's own words — Miller
names the unexpected token — point at
the place.

## round() refuses a second argument

**Cause:** Miller's `round` takes one
argument and rounds to a whole number;
`round($x, 2)` is an error, not a
rounding to cents (chapter 5,
transcript `t06`).

**Fix:** for decimal places, format
instead — `fmtnum($x, "%0.2f")` — and
remember the result is text, with
everything chapter 6 says about text
in summaries.
