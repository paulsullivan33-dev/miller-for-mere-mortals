# Your First Commands

This chapter reads the lab's orders file
four ways: whole, narrowed to some
columns, narrowed to some rows, and in a
different order. Every command names its
columns, and every output below is a
real transcript (`miller-lab/transcripts/`)
run against `miller-lab/data/orders.csv`,
which you can repeat word for word.

The file, once more:

```text
order_id,customer,product,quantity,unit_price
1001,ana,notebook,3,4.50
1002,ben,pencil,12,0.80
1003,ana,mug,2,9.25
1004,cora,notebook,1,4.50
1005,ben,teapot,1,24.00
1006,dario,pencil,20,0.80
1007,cora,kettle,1,31.50
1008,ana,pencil,6,0.80
1009,eli,notebook,4,4.50
```

## First, the trap: no flag, no CSV

Miller does not guess formats
(chapter 2). Leave the flag off and the
default format — DKVP, `name=value`
pairs — is used to read your CSV, with
results that are not an error, just
quietly wrong (transcript `t00`):

```bash
mlr cat orders.csv
```

```text
1=order_id,2=customer,3=product,4=quantity,5=unit_price
1=1001,2=ana,3=notebook,4=3,5=4.50
```

Miller took each comma as a pair
separator and numbered the pieces,
because in DKVP an unnamed value gets a
number for a name. Nothing failed;
nothing warned. **The format flag is
not decoration** — when a Miller run
produces numbered fields, you forgot to
say what the file is.

## cat: the file, as records

With the flag, `cat` passes every record
through unchanged:

```bash
mlr --csv cat orders.csv
```

For human eyes there is a kinder output.
`--opprint` (with `--icsv` naming the
input) draws the records as an aligned
table:

```bash
mlr --icsv --opprint cat orders.csv
```

```text
order_id customer product  quantity unit_price
1001     ana      notebook 3        4.50
1002     ben      pencil   12       0.80
```

Most "let me look at this file" moments
in this guide end in `--opprint`.
Nothing about the records changes; only
the writing does.

## cut: fewer columns, by name

`cut -f` keeps the fields you name, in
the order you name them:

```bash
mlr --csv cut -f customer,product,quantity orders.csv
```

```text
customer,product,quantity
ana,notebook,3
ben,pencil,12
ana,mug,2
```

Compare the old way honestly: `cut -d,
-f2,3,4` says the same thing in
positions, and means it only until the
file changes shape. `-o` reverses the
cut, keeping everything *except* the
named fields — the tool for dropping a
column or two from a wide export.

## filter: fewer rows, by test

`filter` keeps the records whose
expression is true:

```bash
mlr --csv filter '$quantity > 2' orders.csv
```

Five of the nine rows survive
(chapter 1 showed them). The expression
is Miller's language (chapter 2):
comparisons with `>`, `<`, `>=`, `==`
and `!=`, text tested with `==` against
a quoted string —
`filter '$product == "pencil"'` — and
tests combined with `&&` and `||`, as in
`filter '$quantity > 2 && $customer ==
"ana"'`. The quotes around the whole
expression are the shell's, protecting
the `$` and spaces from it; inside them,
Miller's own string quotes are double.

## sort: a different order

`sort` takes field names too, and one
letter that decides everything:

```bash
mlr --csv sort -f customer -nr quantity orders.csv
```

```text
order_id,customer,product,quantity,unit_price
1008,ana,pencil,6,0.80
1001,ana,notebook,3,4.50
1003,ana,mug,2,9.25
1002,ben,pencil,12,0.80
...
```

Rows are grouped by `customer` (the
`-f` field, alphabetical), and within
each customer by `quantity`, largest
first — that is the `-nr`: **n**umeric,
**r**eversed. Leave the `n` off and
Miller sorts numbers as *text*, where
`20` files itself between `3` and `2`
because the character `2` is less than
the character `3`. This guide's lab
made exactly that slip in an early
chain (transcript `t10` keeps the
evidence), and chapter 14 files it
under its symptom: *the sort looks
drunk*. The rule is short: numbers you
care about get `-n`; text does not
need it; `-r` alone reverses whatever
order the comparison produced.

Check: the ana rows above read 6, 3, 2 —
descending numbers, not descending
characters. If yours read 3, 2, 6 or
6, 2, 3, a flag is missing.
