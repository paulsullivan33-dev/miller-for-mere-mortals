# Summarizing Data

Selection and computation work a row at
a time. Summaries work on the whole
file at once: how many, how much, per
what. Miller's summarizing verb is
`stats1`, and this chapter covers it,
its group-by, and the small counting
verbs around it — plus the two quiet
behaviours a summary can have: the row
it skips without mentioning, and the
comparison that lets text through a
numeric gate.

## stats1: sums, averages, counts

```bash
mlr --csv stats1 -a sum,mean,count -f quantity,unit_price orders.csv
```

```text
quantity_sum,quantity_mean,quantity_count,unit_price_sum,unit_price_mean,unit_price_count
50,5.555555555555555,9,80.64999999999999,8.96111111111111,9
```

(transcript `t05`). Read it as a form:
`-a` names the **accumulators** — the
statistics to compute — and `-f` the
fields to compute them over. The output
is one record whose fields are named
`<field>_<accumulator>`. `stats1` offers
many more accumulators than these three
— `min`, `max`, `stddev`, percentiles
and others; `mlr stats1 --help` lists
them, and the pattern never changes.

Two honest details ride along in that
one line. The mean of the quantities,
5.555555555555555, is a repeating
decimal printed to the precision Miller
keeps, not rounded for your comfort.
And `unit_price_sum` is
80.64999999999999 — chapter 5's
floating-point tail again, this time in
a sum of the prices themselves. Neither
is wrong; both are what the machine
actually holds. Reports that need tidy
decimals get them from `fmtnum` or from
the tool that prints the report, not
from pretending the tail is not there.

## Group-by: one summary per customer

Add `-g` and the single summary record
becomes one per group:

```bash
mlr --csv stats1 -a sum -f quantity -g customer orders.csv
```

```text
customer,quantity_sum
ana,11
ben,13
cora,2
dario,20
eli,4
```

This is the single most useful shape in
the chapter: *-a what, -f over, -g per*.
It is the command that answers "total
per customer", "average per product",
"count per city" for any file whose
columns have names — the question the
old text tools needed a sort and a small
program to answer. Group on several
fields by naming them all after `-g`,
and compute several statistics by
naming them all after `-a`; the output
grows columns, not cleverness.

## The row the summary skipped

Run the same verb over the stock file,
whose `count` column holds one `n/a`:

```bash
mlr --csv stats1 -a sum -f count stock.csv
```

```text
count_sum
57
```

57 is 40 + 17. The `n/a` row was not
counted, not averaged in as zero, and
not complained about: `stats1` simply
builds its statistics from the values
that are numbers and steps over the
rest in silence. Sometimes that is
exactly right — a sum of what was
actually counted. Sometimes it is a
missing delivery hiding in a total that
looks complete. Miller will not tell
you which; the defence is to pair the
summary with a count of what went in,
which is why `count` appears among the
accumulators above: `quantity_count`
said 9, the number of numeric values
the mean was built from, and comparing
that against the row count of the file
is the audit. Chapter 11 makes a recipe
of it.

## A word about comparisons and text

The same silence lives in `filter`,
from the other side. Filter the stock
file for large counts:

```bash
mlr --csv filter '$count > 10' stock.csv
```

```text
item,count
bolts,40
screws,n/a
washers,17
```

The `n/a` row **passed** a test for
"greater than 10" (transcript `t04`).
Miller's comparisons are *type-ordered*:
when the two sides are of different
kinds, they are compared by kind first,
and in that ordering every text value
stands after every number — so any
string at all is "greater than" any
number you test against. A numeric
filter is therefore not a numeric
*sieve* unless the column is clean: it
keeps numbers that pass, and it also
keeps every piece of text, because text
outranks numbers by kind. If a filter
must mean "numbers only, and large",
say both halves:
`filter 'is_numeric($count) && $count >
10'` — `is_numeric` is one of the
expression language's test functions,
and chapter 10 gathers its relatives.

## The small counters

Three verbs cover most remaining
counting questions, all run in the lab
(transcript `t13`):

- `mlr --csv count orders.csv` prints a
  single field, `count`, holding the
  number of records: 9. The denominator
  every audit in this chapter wants.
- `mlr --csv uniq -c -g product
  orders.csv` counts records per group
  and nothing more:

  ```text
  product,count
  notebook,3
  pencil,3
  mug,1
  teapot,1
  kettle,1
  ```

- `mlr --csv rename customer,buyer
  orders.csv` renames a field in flight
  — the verb for meeting a downstream
  tool that insists on its own column
  names, and a reminder that in Miller
  the header is data like any other,
  subject to verbs.

A caution about `top`, because its name
promises the wrong thing: `mlr --csv
top -n 2 -f quantity orders.csv` does
*not* print the two largest orders. It
prints a small summary of the two
largest *values* —

```text
top_idx,quantity_top
1,20
2,12
```

— which is a statistician's answer, not
a reader's. The two largest orders are
`sort -nr quantity` followed by
`head -n 2` (chapter 9 chains them in
one command). Chapter 14 lists this
under its symptom, because everyone
tries `top` first.
