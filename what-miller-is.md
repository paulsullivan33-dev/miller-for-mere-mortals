# What Miller Is

A CSV file is a table written as text.
The first line names the columns; each
line after it is one row:

```text
order_id,customer,product,quantity,unit_price
1001,ana,notebook,3,4.50
1002,ben,pencil,12,0.80
```

Sooner or later you want something from
such a file that a text editor cannot
give you: only the rows where the
quantity is over 2. The rows sorted by
customer. A new column that multiplies
quantity by price. The total quantity
per customer. A second file — which
customer lives in which city — merged
in on the customer name.

The traditional Unix tools answer these
questions by *position*. `cut -d, -f4`
takes the fourth field. `awk '$4 > 2'`
tests the fourth field. `sort -t, -k2`
sorts on the second. Positions are
compact and fast, and they have one
structural weakness: they say nothing
about what the field *is*. When the
export that feeds your script gains a
column, or a colleague reorders two of
them, every position in every command
now points at the wrong data — and
nothing complains. The numbers still
look like numbers. They are just
somebody else's numbers.

**Miller is the same toolbox rebuilt
around names.** Its command is `mlr`.
Where `cut` wants `-f4`, Miller wants
`-f quantity`. Where awk tests `$4 > 2`,
Miller tests `$quantity > 2`. The column
can move anywhere in the row; the name
still finds it. Miller's author calls it
"awk, sed, cut, join, and sort for
name-indexed data," and that is exactly
the shelf it sits on — with two additions
the old tools never had: it understands
several table formats natively (CSV,
TSV, JSON and more, chapter 8), and it
converts between them as a side effect
of reading and writing.

## A first taste

After installing (chapter 3), this is
the whole of a Miller command:

```bash
mlr --csv filter '$quantity > 2' orders.csv
```

Three ideas are packed into that line,
and the whole guide is an unpacking of
them:

- `--csv` says what the file *is* —
  Miller does not guess formats, and
  chapter 4 shows what happens when you
  leave the flag off.
- `filter` is a **verb**: the operation
  to perform. Miller has dozens of them
  (`mlr -l` lists them all) — `cut`,
  `sort`, `join`, `stats1` for summaries,
  `put` for computing new fields, and
  more.
- `'$quantity > 2'` is an expression in
  Miller's small language: keep the rows
  where the field *named* `quantity`
  holds a number over 2.

The output is the same CSV, minus the
rows that failed the test:

```text
order_id,customer,product,quantity,unit_price
1001,ana,notebook,3,4.50
1002,ben,pencil,12,0.80
1006,dario,pencil,20,0.80
1008,ana,pencil,6,0.80
1009,eli,notebook,4,4.50
```

That is transcript `t02` in
`miller-lab/transcripts/`, run against
the lab's own nine-row orders file,
which ships in `miller-lab/data/` so
you can repeat every command in this
guide yourself.

## Why anyone would do this

- **The export changed again.** The
  report you run every Friday reads a
  CSV whose columns were rearranged by
  an update. Name-based commands survive
  the rearrangement; position-based ones
  silently read the wrong columns.
- **The question is about the table,
  not the text.** "Total per customer"
  is a grouping question. Miller's
  `stats1 -g customer` answers it in one
  command (chapter 6); in raw text tools
  it is a small program.
- **Two files share a key.** Orders know
  the customer; a second file knows the
  customer's city. Miller's `join`
  merges them on the name they share
  (chapter 7), the way a database would,
  without a database.
- **The format is wrong for the next
  tool.** The API answered in JSON; the
  spreadsheet wants CSV. One Miller
  command converts, because reading one
  format and writing another is the same
  command with different flags
  (chapter 8).

## Who it is for

The person who receives tables — from a
system, a service, a colleague, an
instrument — and needs answers from them
at a terminal: the filtered extract, the
sorted list, the summary, the merged
file. The person maintaining scripts
that read exports whose shape they do
not control. The person who knows the
awk one-liner would work and would
rather write the command their successor
can still read after the columns move.

## What it is not

Miller is not a database: it reads files
and writes files, and when you need the
data to *stay* somewhere, be queried by
five programs at once, or refuse
contradictory updates, that is a
database's job (chapter 12 says more).
It is not a spreadsheet: there is no
grid to look at, no cell to click — its
whole interface is commands and text,
which is precisely why it can be
scripted and repeated. It is not a data
cleaning service: it will faithfully
total a column in which one entry is
the text `n/a` by stepping over that
row without a word (chapter 6 shows
this, and it is a behaviour to know,
not a bug to fix). And it is not
forgiving about being told the wrong
format: Miller believes the flags you
gave it, and chapter 4 opens with the
quiet nonsense that comes out when the
flags and the file disagree.
