# Recipes

Short jobs, whole. Each recipe names
the job, gives the command as the lab
ran its parts, and says what to check
before trusting the output. The files
are the lab's fixtures throughout;
your columns will differ, and the
shapes of the commands will not.

## Total spent per customer, tidied

*Job: a small report — each customer's
spend, largest first, money printed
like money.*

```bash
mlr --csv put '$total = $quantity * $unit_price' then stats1 -a sum -f total -g customer then sort -nr total_sum orders.csv
```

Then, because chapter 5's tails belong
in drafts and not in reports, format at
the end rather than the beginning —
formatting first would make the column
text, and `stats1` steps over text in
silence (chapter 6):

```bash
mlr --csv put '$total = $quantity * $unit_price' then stats1 -a sum -f total -g customer then sort -nr total_sum then put '$total_sum = fmtnum($total_sum, "%0.2f")' orders.csv
```

Check: the customer count in the
output equals the distinct customers
in the input — `uniq -g customer` on
the orders says five — and eli is
present with 18, because no join was
involved to lose him.

## Audit a file you just received

*Job: before any real work, learn what
a new file is hiding.*

Four commands, in order:

```bash
mlr --csv count incoming.csv
mlr --icsv --opprint head -n 5 incoming.csv
mlr --csv stats1 -a count -f amount incoming.csv
```

The first counts the rows; the second
shows you the shape of the thing. The
third counts the *numeric* values in
the column you care about — and if
that count is smaller than the row
count, the difference is rows whose
value is text, empty, or otherwise not
a number, each of which your summaries
would have stepped over silently. Find
them while you still can:

```bash
mlr --csv filter '!is_numeric($amount)' incoming.csv
```

Check: the two counts and the found
rows agree — rows in, equals numbers
counted, plus rows the last command
shows you.

## Rescue a computation from its errors

*Job: a `put` produced `(error)` cells;
produce the clean file and the list of
casualties separately.*

```bash
mlr --csv put '$double = $count * 2' then filter 'is_error($double)' stock.csv
```

run once to see the casualties, and
then the same chain with
`'!is_error($double)'` to keep the
sound rows. Check: the two outputs'
row counts add back to the input's.
(Chapter 5 explains why testing
`$double == "(error)"` cannot do this
job.)

## A spreadsheet-ready extract

*Job: three columns, one customer's
rows, tabs instead of commas so the
spreadsheet stops arguing.*

```bash
mlr --icsv --otsv filter '$customer == "ana"' then cut -f product,quantity,unit_price orders.csv > ana.tsv
```

Check: open the result with
`mlr --itsv --opprint cat ana.tsv` and
see a table of three rows — ana's —
with the columns in the order the
recipient asked for, because `cut`
emits in the order listed.

## Flatten an API answer for a report

*Job: the service answered JSON; the
report is a CSV sorted by the field
that matters.*

```bash
mlr --ijson --ocsv cat answer.json then sort -nr quantity
```

— in two steps if you prefer to look
between them, one chain if you do not;
chapter 8's conversion is the first
half either way. Check: the row count
matches the JSON array's length, and
spot-check one object's values against
its CSV row — conversion should move
values, never edit them.

## Who is missing from the second file?

*Job: before joining (chapter 7), list
the streamed records that have no
partner, so the drop is a decision and
not a surprise.*

```bash
mlr --csv --ojson join -j customer --ur -f customers.csv orders.csv
```

and read the tail of the output: the
records standing without the held
file's fields are the unpaired ones.
Check: their count, plus the plain
join's output count, equals the
streamed file's row count — in the
lab, 1 + 8 = 9.
