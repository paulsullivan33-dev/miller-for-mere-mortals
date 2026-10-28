# Formats In, Formats Out

Every Miller command so far read CSV
and wrote CSV, which disguises the
fact that the two choices are
independent. The record stream in the
middle (chapter 2) has no format at
all; formats are how records get in
and how they leave. This chapter spends
that freedom: CSV becomes JSON, JSON
becomes CSV, and two kinds of problem
file — the one with no header and the
one whose rows disagree — are handled
honestly, meaning one of them first
stops the line.

## CSV to JSON, and back

The verb for a pure conversion is
`cat` — records in, records out, no
operation — with different flags on
each side:

```bash
mlr --csv --ojson cat orders.csv
```

```text
[
{
  "order_id": 1001,
  "customer": "ana",
  "product": "notebook",
  "quantity": 3,
  "unit_price": 4.50
},
```

(transcript `t07b`). Two details in
that fragment deserve a look. The
numbers are numbers, not strings:
Miller *infers* types as it reads, so
`quantity` crossed over as a JSON
number without being asked. And
`4.50` kept its trailing zero — Miller
remembers how a number was written in
its source and writes it back the same
way when it can, a small fidelity that
matters when a downstream tool compares
text.

The return trip uses the third fixture,
a three-order JSON file:

```bash
mlr --ijson --ocsv cat orders.json
```

```text
order_id,customer,product,quantity,unit_price
2001,ana,notebook,2,4.50
2002,ben,mug,1,9.25
2003,cora,pencil,10,0.80
```

Field order follows the order the keys
first appear in the JSON. A relative of
the JSON array is worth knowing:
`--ojsonl` writes **JSON Lines** — one
self-contained object per line, no
surrounding array —

```text
{"order_id": 1001, "customer": "ana", "product": "notebook", "quantity": 3, "unit_price": 4.50}
```

— the shape log pipelines and other
stream tools prefer, because a reader
never holds more than one record. TSV,
met in chapter 2, is the same trick in
the other direction: `--icsv --otsv`
rewrites the orders with tabs, dodging
every comma-quoting question a CSV
answer would raise in a spreadsheet.

One caution about the trip back: it is
not always a round trip. JSON can nest
— an object inside an object, a list
inside a field — and a flat CSV cell
cannot hold a nest. Miller flattens
what it can and has options for the
rest; deeply nested JSON is a shape
Miller reads bravely and writes out
flat, and chapter 13 counts real
nesting among its honest limits.

## The file with no header

Some files arrive without the naming
line — an instrument's log, an old
system's extract:

```text
monday,21.5,ok
tuesday,19.0,ok
wednesday,23.25,check
```

`--implicit-csv-header` tells Miller to
treat every line as data and lend the
columns the names `1`, `2`, `3`:

```bash
mlr --csv --implicit-csv-header cat readings.csv
```

```text
1,2,3
monday,21.5,ok
tuesday,19.0,ok
wednesday,23.25,check
```

Positional names are a step backwards
toward `awk`'s world, and that is the
honest framing: a headerless file gives
you nothing better to hold on to. But
the machinery all works — here is a
`put` flagging the warm days, field
`$2` and all, producing the booleans
`true` and `false` as field values:

```bash
mlr --csv --implicit-csv-header put '$warm = ($2 > 20)' readings.csv
```

```text
1,2,3,warm
monday,21.5,ok,true
tuesday,19.0,ok,false
wednesday,23.25,check,true
```

Often the better move is to give the
file real names once, at the door, and
work by name ever after: Miller's
`label` verb renames the fields in
order, so a chained
`--implicit-csv-header label
day,temp,status` turns the numbered
columns into named ones for every verb
that follows (transcript `t16`). The
fully headerless *formats* have their
own flags, with their own separator
rules: `--nidx`, for a whitespace-split
file, numbers the fields the same way —
but pointed at this comma file it saw
one wide field per line, commas and
all, because in its format the
separator is the space. Headerless is
not one idea; it is one idea per
delimiter, and the flag has to match
the file's.

## The ragged file

The last fixture is broken on purpose:
its rows disagree about the shape of
the table.

```text
a,b,c
1,2,3
4,5
6,7,8,9
```

Miller's default answer is a refusal,
and it arrives *mid-stream*
(transcript `t08`):

```bash
mlr --csv cat ragged.csv
```

```text
a,b,c
1,2,3
mlr: CSV header/data length mismatch 3 != 2 at filename ragged.csv row 3
```

Exit status 1 — but notice that the
header and the first good row were
already printed before the failure.
Miller streams (chapter 2), so a
pipeline downstream may have consumed
partial output before the error was
known. Miller is not being careless
here; it is being consistent: a row
whose length contradicts the header is
a data error, and it names the file,
the row and both lengths.

When you judge the file's raggedness
to be its nature rather than its
corruption — exports with optional
trailing columns are the usual case —
the forgiveness has a flag:

```bash
mlr --csv --allow-ragged-csv-input cat ragged.csv
```

```text
a,b,c
1,2,3
4,5,
6,7,8,9
```

Read the output as carefully as Miller
read the input. The short row is
padded with an empty field, so its
record at least has the header's shape.
The long row is passed through *with
its extra field still on it* — four
values under a three-name header —
which means the stream now holds two
shapes, and any strict output or
downstream step will meet them as
chapter 7's join met eli: one shape
will refuse. Forgiving the input is a
decision to handle the shapes yourself,
not a repair.
