# The Expression Language

Chapters 5 and 6 used the expression
language one clause at a time. This
chapter gathers it: making choices,
working on text, the built-in numbers,
and — the section to read twice — the
variables that remember. Everything
below was run in the lab, including the
failures, which are kept because they
teach the rules faster than the
successes do.

## Choices: if, and the question mark

The everyday choice is the ternary
operator, `test ? value : value`:

```bash
mlr --csv put '$size = ($quantity >= 10) ? "bulk" : "small"' orders.csv
```

```text
1001,ana,notebook,3,4.50,small
1002,ben,pencil,12,0.80,bulk
1006,dario,pencil,20,0.80,bulk
```

(transcript `t11`). For choices that
do more than pick a value, the language
has a full `if` statement — and its
body must wear curly braces, even when
it is a single statement:

```text
if ($quantity >= 10) { $size = "bulk" }
```

Omitting the braces is a parse error,
not a style choice; the project's
documentation shows the failing and
passing forms side by side, and the
lab's scripts obeyed the braces.

Several statements travel together
separated by semicolons:
`$a = 1; $b = 2`. The semicolons are
required **even when the statements are
on separate lines** — a rule the lab
proved by omission: a `put -f` script
file whose second statement began on a
fresh line, no semicolon behind it,
failed with "cannot parse DSL
expression... unexpected field_name".
Long expressions belong in files
(`put -f script.mlr`) precisely so they
can be laid out readably — semicolons
and all.

## Working on text

The functions used most (transcript
`t11` holds each of these):

- `toupper($product)` and its sibling
  `tolower` — case, for matching and
  display.
- `sub($product, "pencil", "pen")` —
  replace the *first* match of a
  pattern in a string; `gsub` replaces
  every match. The pattern is a regular
  expression, not a literal, which is
  power and tripwire in one: a `.` in
  the pattern means "any character"
  until you escape it.
- `fmtnum($x, "%0.2f")` from chapter 5 —
  numbers as formatted text.
- The tests: `is_numeric($x)`,
  `is_string($x)`, `is_error($x)` —
  the audit toolkit of chapters 5 and 6,
  usable anywhere a true/false is
  wanted.

## Built-in numbers

A few names are always filled in, in
capitals, and cannot be assigned to:
`NR` is the record's number in the
whole stream, `FNR` its number within
the current file, `NF` the number of
fields in the current record, and
`FILENAME` the file being read. `NR`
is the one that earns its place weekly:

```bash
mlr --csv put '$n = NR' orders.csv
```

gives every record a row number that
survives into the output — the honest
way to say "the third line of the
result" to a colleague.

## The variables that remember

Here is the chapter's heart, and the
lab arrived at it by getting a running
total wrong. The awk habit says: keep
a plain variable, add to it per record.

```bash
mlr --csv put 'begin {t = 0} t = t + $quantity; $running = t' orders.csv
```

```text
1001,ana,notebook,3,4.50,3
1002,ben,pencil,12,0.80,12
1003,ana,mug,2,9.25,2
```

(transcript `t12b`). `$running` is just
`$quantity` wearing a coat: every
record starts with `t` forgotten,
because **a bare name in Miller is a
local variable, and a local variable
lives only for the record being
processed**. The `begin` block ran —
a `dump` in the lab showed its value —
but what it set was a local too, gone
before the first record arrived.

Memory across records is a different
kind of name: the **out-of-stream
variable**, spelled with `@`. Its home
is outside any single record, which is
what the name says. The same program,
translated:

```bash
mlr --csv put 'begin { @t = 0 } @t += $quantity; $running = @t' orders.csv
```

```text
1001,ana,notebook,3,4.50,3
1002,ben,pencil,12,0.80,15
1003,ana,mug,2,9.25,17
...
1009,eli,notebook,4,4.50,50
```

(transcript `t12c`). A true running
total, ending at 50 — which the reader
who has been following the numbers
will recognize as chapter 6's
`quantity_sum`, reached by a second,
independent road. That agreement is
the check.

The `@` variables pair with the
`begin` and `end` blocks this chapter
has been using informally:

- `begin { ... }` runs once, before
  the first record: where `@` variables
  are given their starting values.
- `end { ... }` runs once, after the
  last: where an `emit` can write the
  accumulated values into the stream as
  a final record.

With `put -q` — which suppresses the
pass-through of the input records —
the end block's record is the *only*
output, and the running-total program
collapses into a grand total:

```bash
mlr --csv put -q 'begin { @t = 0 } @t += $quantity; end { emit {"total_qty": @t} }' orders.csv
```

```text
total_qty
50
```

One warning rides with `emit`, and the
lab earned it: an emitted record has
its own fields, so under CSV output it
has a different shape from the data
records, and emitting it into the same
stream as the data re-opens chapter 7's
schema-change refusal. Emit into JSON,
or use `-q` and let the summary stand
alone.

That is the language's working core:
fields with `$`, memory with `@`,
choices with braces, statements with
semicolons. The project documents a
full language beyond it — user-defined
functions, maps, loops — and this guide
leaves those to that documentation
deliberately. What is here is what the
everyday commands are made of.
