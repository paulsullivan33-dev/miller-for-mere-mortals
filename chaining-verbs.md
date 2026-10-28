# Chaining Verbs

Every command so far did one job. Real
questions take two or three: the orders
*over a quantity*, showing *only these
columns*, *largest first*. The shell
would answer with a pipeline of separate
commands and temporary re-readings of
the table between them. Miller answers
inside one command, with the word
`then` between verbs:

```bash
mlr --csv filter '$quantity >= 2' then cut -f customer,product,quantity then sort -nr quantity orders.csv
```

```text
customer,product,quantity
dario,pencil,20
ben,pencil,12
ana,pencil,6
eli,notebook,4
ana,notebook,3
ana,mug,2
```

(transcript `t10`). Records flow from
`filter` to `cut` to `sort` as a stream
of records — not as text to be
re-parsed at each step — so a field
`put` computes in step one is a real
field that step three can sort by,
group by, or total. That is the
difference between a chain and a shell
pipeline wearing a trench coat: the
table between the steps is already
understood.

## A chain that computes, then summarizes

Chapter 6's summary, with its input
manufactured one step earlier:

```bash
mlr --csv put '$total = $quantity * $unit_price' then stats1 -a sum -f total -g customer then sort -nr total_sum orders.csv
```

```text
customer,total_sum
ana,36.8
cora,36
ben,33.6
eli,18
dario,16
```

Spend per customer, largest first, in
one command with nothing written to
disk in between. Note that the last
`sort` sorts on `total_sum` — a field
that did not exist anywhere in the
input file, born in `put`, summed by
`stats1`, and named by the summarizer's
`<field>_<accumulator>` convention
(chapter 6). Chains are where the
naming conventions pay rent: every
step's output fields are the next
step's vocabulary.

## The order of steps is the meaning

A chain is read in the order it runs,
and the order is rarely negotiable:

- `filter` before `sort` is cheaper
  than after — fewer records to order —
  and identical in result.
- `cut` before `join` can delete the
  very field the join needs; `cut`
  after it cannot hurt it. Chains make
  such dependencies visible in one
  line, which is a point in their
  favour over three separate commands
  and two temp files.
- A summarizing verb (`stats1`, `uniq`,
  `count`) ends the record stream's
  first life: what flows after it is
  summary records, with summary field
  names. Chaining onward is normal —
  the example above sorts the summary —
  but no step after a summary can see
  the original rows, because they are
  gone.

Two more citizens of the chain, both
run in the lab: `head -n 2` after a
sort gives the top of the order (the
honest route to "the two largest
orders" that chapter 6 contrasted with
the `top` verb), and `head` is a
whole-stream citizen — given the same
file twice on one command line,
`mlr --csv head -n 2 orders.csv
orders.csv` printed two records total,
not two per file (transcript `t13`),
because to the chain there is only one
stream.

## When to stop chaining

A chain is one command doing one job.
When the line grows past what you can
read aloud in one breath, the honest
options are the shell's own: write an
intermediate file with one `mlr`
command and start the next from it, or
put the chain in a script where it can
have comments. Miller's `-f` script
files (chapter 10) cover the expression
language's half of that need. What
chains are for is the work in between:
too many steps for one verb, too
connected for temp files.
