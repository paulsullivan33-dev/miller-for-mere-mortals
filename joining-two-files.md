# Joining Two Files

Two files that share a column are one
question waiting to be asked. The
orders know what each customer bought;
the customers file knows where each
customer lives. Neither answers "what
did Lisbon buy?" alone. A database
would join the tables; Miller's `join`
verb does the same job on files, on the
command line, and this chapter runs it
— including the order it silently drops
and the output format that refuses to
carry the dropped row when you ask for
it back.

The second file:

```text
customer,city,since
ana,Lisbon,2019
ben,Porto,2021
cora,Lisbon,2018
dario,Braga,2023
```

## The plain join

```bash
mlr --csv join -j customer -f customers.csv orders.csv
```

```text
customer,city,since,order_id,product,quantity,unit_price
ana,Lisbon,2019,1001,notebook,3,4.50
ben,Porto,2021,1002,pencil,12,0.80
ana,Lisbon,2019,1003,mug,2,9.25
cora,Lisbon,2018,1004,notebook,1,4.50
ben,Porto,2021,1005,teapot,1,24.00
dario,Braga,2023,1006,pencil,20,0.80
cora,Lisbon,2018,1007,kettle,1,31.50
ana,Lisbon,2019,1008,pencil,6,0.80
```

(transcript `t06`). Read the command
inside out: `-j customer` names the
**join field**, the column both files
share. `-f customers.csv` names the
file Miller loads first and holds —
call it the *held* file — and the file
named last, `orders.csv`, is the one
that streams past. For each streamed
record, Miller finds held records with
the same `customer` and emits the two
bolted together: the join field once,
then the held file's other fields, then
the streamed record's. Nine orders went
in. **Eight rows came out.** The join
has already made its first quiet
decision, and the next section is about
catching it.

## The missing order

Order 1009 belongs to `eli`, and `eli`
is not in the customers file. A plain
join is an *inner* join: it emits pairs,
and a record with no partner is not a
pair, so it vanishes — no warning, no
count, no footnote. This is correct
database behaviour and a standing
invitation to fool yourself: a joined
file is shorter than its input, and
every summary you run on it is a
summary of the customers you happened
to have a second file for.

Ask for the unpaired records back with
`--ur` — the streamed (right-hand)
file's unpaired records; `--ul` does
the same for the held file, and in this
lab's data it adds nothing, because all
four customers do appear in the orders.
But watch what happens when you ask
under CSV output:

```bash
mlr --csv join -j customer --ur -f customers.csv orders.csv
```

```text
mlr: CSV schema change: first keys "customer,city,since,order_id,product,quantity,unit_price"; current keys "order_id,customer,product,quantity,unit_price"
mlr: exiting due to data error
```

(transcript `t06c`). The paired rows
streamed out happily; then the unpaired
order arrived — a record *without* the
`city` and `since` fields, a different
shape — and CSV, which writes one
header for the whole file, refused to
change shape mid-file and stopped the
run. This is chapter 2's warning made
real: CSV is the strict format. The
remedies are the formats that tolerate
mixed shapes. The same command with
JSON out —

```bash
mlr --csv --ojson join -j customer --ur -f customers.csv orders.csv
```

— ends with eli's order standing alone,
its record simply lacking the two
fields it never acquired:

```text
{
  "order_id": 1009,
  "customer": "eli",
  "product": "notebook",
  "quantity": 4,
  "unit_price": 4.50
}
```

The lesson generalizes past joins:
whenever a Miller pipeline can emit
records of more than one shape, write
JSON or DKVP, or make the shapes agree
before you ask CSV to carry them.

## House rules for joining

- **Know which file is held.** The `-f`
  file is loaded whole and kept; the
  streamed file can be enormous. Hold
  the small file.
- **Sort is not required.** Unlike the
  old Unix `join`, Miller's builds an
  in-memory index of the held file, so
  input order does not matter. (The
  held file's size is the memory bill —
  chapter 13 notes it.)
- **Count before and after.** `count`
  on the input, `count` on the output:
  the difference is the number of
  records the join decided not to pair,
  and you want to have decided that was
  acceptable before anyone reads the
  report. Chapter 11 builds the audit
  into a recipe.
- **Duplicates multiply.** If the held
  file lists a customer twice, every
  matching order pairs with *both*
  entries and comes out twice. A join
  is a pairing, not a lookup, and a
  duplicated key on either side is
  faithfully multiplied.
