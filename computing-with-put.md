# Computing with put

`cut`, `filter` and `sort` select from
what a file already holds. `put` is the
verb that *adds*: it evaluates an
expression for every record and writes
the result into a field — a new one, or
an existing one replaced. This chapter
computes the line total of every order,
builds a sentence out of two fields, and
then walks into the two honest surprises
that arithmetic on real-world files
produces: a number with fifteen digits
of apology after it, and a cell that
says `(error)`.

## A new column from two old ones

```bash
mlr --csv put '$total = $quantity * $unit_price' orders.csv
```

```text
order_id,customer,product,quantity,unit_price,total
1001,ana,notebook,3,4.50,13.5
1002,ben,pencil,12,0.80,9.600000000000001
1003,ana,mug,2,9.25,18.5
1004,cora,notebook,1,4.50,4.5
```

The new field `total` appears at the
end of every record (transcript
`t03`). Read the expression as: for
each record, multiply the field named
`quantity` by the field named
`unit_price`, and store the answer in
the field named `total`. Assigning to a
field that already exists replaces it —
`put '$unit_price = $unit_price * 1.2'`
is a price rise — and assigning is the
only thing `put` does; a `put` command
prints every record, changed.

**The first surprise is in row 1002.**
Twelve pencils at 0.80 should be 9.60,
and is — plus a tail: `9.600000000000001`.
This is not a Miller bug; it is binary
floating-point arithmetic, the same one
your programming language does, because
0.80 cannot be written exactly in
binary and the error survives into the
printout. Know it exists, because a
column of money will grow these tails
wherever the true answer is a "round"
decimal. The remedy is to format the
result as the text you actually want,
with `fmtnum`:

```bash
mlr --csv put '$total = fmtnum($quantity * $unit_price, "%0.2f")' orders.csv
```

```text
1001,ana,notebook,3,4.50,13.50
1002,ben,pencil,12,0.80,9.60
1003,ana,mug,2,9.25,18.50
```

`fmtnum(value, format)` returns a
*string* built to the format — `"%0.2f"`
means two digits after the point — so
the `total` column is now text wearing a
number's clothes: fine for a report or
an invoice extract, and worth knowing
before you hand the file to a tool that
will add the column up. (Chapter 14
tells the fuller story, including why
`round($x, 2)` is not the answer you
expected: Miller's `round` takes one
argument and rounds to a whole number.)

## Text is joined with a dot

Arithmetic has `+`; text has its own
operator, `.`, which glues strings
together:

```bash
mlr --csv put '$label = $customer . " bought " . $product' orders.csv
```

```text
1001,ana,notebook,3,4.50,ana bought notebook
1002,ben,pencil,12,0.80,ben bought pencil
```

Numbers handed to `.` are written out
as text first. The distinction matters
because of what `+` does to text — which
is the second surprise, and it needs a
dirtier file.

## The (error) cell

The lab's stock file has a hole in it —
one count that was never taken:

```text
item,count
bolts,40
screws,n/a
washers,17
```

Double every count:

```bash
mlr --csv put '$double = $count * 2' stock.csv
```

```text
item,count,double
bolts,40,80
screws,n/a,(error)
washers,17,34
```

(transcript `t04`). Three things to
learn from this small output:

- **Miller did not stop.** The run's
  exit status was 0; the healthy rows
  computed fine. Where the expression
  could not be evaluated — text times a
  number — the *value* `(error)` was
  stored in the field, as data.
- **`(error)` is a value, not a
  message.** It will sit in your output
  file, pass into the next verb, and be
  written to disk looking like content.
  So check for it — with the test built
  for the purpose, because the obvious
  one fails: `filter '$double ==
  "(error)"'` compares a value against
  a piece of text, trips over the error
  value itself, and stops the run
  ("filter expression did not evaluate
  to boolean"). The working hunt is
  `filter 'is_error($double)'`, which
  returns exactly the casualties
  (transcript `t15`), and chapter 11
  turns it into a recipe.
- **Text with `+` fares no better.**
  `put '$x = $item + 1'` produced
  `(error)` on *every* row of the stock
  file: adding a number to a word is not
  a thing, even when the word is
  `bolts`. Miller does not guess that
  you meant concatenation; `.` is for
  text, `+` is for numbers, and the
  border between them is enforced by
  producing `(error)`, not by stopping
  the line.

This is a design philosophy, stated
plainly because chapter 13 returns to
it: Miller prefers to keep the stream
moving and mark the bad cell, rather
than halt a long job on one bad row.
Whether that is mercy or a trap depends
on whether you look. This guide's
position is that you look — a computed
column that no one has scanned for
`(error)` is a draft, not a result.
