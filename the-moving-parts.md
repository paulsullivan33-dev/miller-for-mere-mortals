# The Moving Parts

A Miller command has five parts, and
every surprise in this guide belongs to
one of them. Learn the parts and the
surprises become named problems instead
of mysteries.

```bash
mlr --csv filter '$quantity > 2' then sort -nr quantity orders.csv
```

Reading left to right: the program, the
**format flags**, a **verb** with its
options and expression, the word `then`
and a second verb, and the **file**.
That is the whole grammar. This chapter
takes the parts one at a time.

## Records and fields

Miller does not think in lines. It
thinks in **records**: one row of the
table, held as a set of **fields**, each
field a name and a value. The orders
row

```text
1001,ana,notebook,3,4.50
```

becomes, inside Miller, five fields:
`order_id` is 1001, `customer` is `ana`,
`product` is `notebook`, `quantity` is
3, `unit_price` is 4.50. A verb receives
one record at a time, does its work —
most verbs simply pass records through
changed or unchanged — and hands the
record to the next verb or to the
output. This is why the format of the
*file* matters less than you expect:
CSV lines, JSON objects and TSV rows
all arrive as the same thing, a record,
and leave in whatever format the output
flags ask for.

Two consequences are worth knowing
early. First, field *order* in a record
is preserved and occasionally matters —
`cut` emits fields in the order you
list them, which is how you reorder
columns. Second, a record is allowed to
*change shape* as it travels: `put` adds
a field, `cut` removes fields, `join`
bolts two records together. Some output
formats tolerate a stream whose records
change shape, and some — CSV is the
strict one — refuse mid-file, loudly.
Chapter 7 meets that refusal in the
wild.

## Format flags

Miller does not sniff, guess or infer
the format of your file from its name.
You declare it:

- `--csv` — comma-separated, first row
  is the header of field names. `--icsv`
  and `--ocsv` set input and output
  separately; combined shorthands exist
  too, such as `--c2j` for CSV in, JSON
  out.
- `--tsv` — the same idea with tabs,
  which spares you the quoting rules
  commas drag in.
- `--json` — a JSON array of objects, or
  with `--jsonl`, one object per line.
- `--dkvp` — Miller's own loose change:
  `name=value` pairs on a line,
  `quantity=3,product=pencil`. It is the
  *default* when you name no format,
  which is why chapter 4's flagless run
  produced such strange fruit.
- `--pprint` (as an output, `--opprint`)
  — an aligned text table for human
  eyes, the format most of this guide's
  "look at it" moments use.

Input and output are independent. That
independence is not a detail; it is
chapter 8's whole subject.

## Verbs

The **verb** is the operation: `cat`
(pass records through), `cut`, `filter`,
`sort`, `head`, `rename`, `uniq`,
`count`, and on into the specialists —
`stats1` for statistics, `join` for
merging files, `put` for computing.
`mlr -l` lists them all, dozens of them;
this guide teaches the everyday core
and names the rest honestly rather than
touring the catalogue. Two things hold
for every verb:

- Each verb has its own help —
  `mlr cut --help` — and its options
  belong to it alone. `-f` after `cut`
  and `-f` after `join` do different
  jobs, because options are read by the
  verb they follow.
- Verbs chain with `then` (chapter 9):
  `filter ... then sort ...` is one
  `mlr` invocation whose records flow
  from verb to verb, the way a shell
  pipeline flows between programs —
  except inside one process, with no
  re-parsing of the table between steps.

## The expression language

`put` and `filter` take an expression in
Miller's small built-in language, the
**DSL**. Field names wear a dollar sign:
`$quantity`. Arithmetic is what you
expect (`$quantity * $unit_price`),
comparisons likewise (`$quantity > 2`),
and text is joined with a dot
(`$customer . " bought " . $product`).
Strings wear double quotes. Statements
end with semicolons when there is more
than one — even across lines, a rule
chapter 10 demonstrates by breaking it.

The language has three kinds of names,
and confusing two of them is the most
instructive mistake in this guide:

- `$name` — a field of the current
  record.
- `@name` — an **out-of-stream
  variable**: a value that persists from
  record to record, for running totals
  and counters (chapter 10).
- a bare `name` — a **local variable**,
  which lives only for the current
  record and is then forgotten. An awk
  user expects a bare name to remember;
  Miller's does not, and chapter 10
  shows the running total that comes
  out wrong because of it.

## Where things stand

Miller is a single static binary: no
interpreter to install, no server to
run, no state between invocations. It
reads the files you name — or standard
input, when you name none — and writes
to standard output. It streams: most
verbs handle one record at a time and
hold nothing back, so a file far larger
than memory flows through `filter` or
`cut` happily. The exceptions name
themselves: `sort` and the grouped
summaries must see everything before
they can answer, and they say so by
their nature, not in fine print.
