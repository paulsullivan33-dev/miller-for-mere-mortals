# Limits

Miller's limits are mostly not walls
but philosophies, and knowing them is
the difference between using the tool
and being used by it. This chapter
states them plainly; most were met
firsthand in the lab and are
cross-referenced to the chapters that
met them.

## It trusts your flags completely

Miller does not sniff formats. Feed it
a CSV while calling it DKVP and it will
happily produce numbered nonsense and
exit 0 (chapter 4). Feed it a file in a
format it has no flag for — a real
spreadsheet workbook, a PDF, a database
file — and there is no reading it at
all: Miller's world is delimited text
and JSON, and the route from a workbook
to Miller runs through an export, not
through Miller.

## It keeps the stream moving past bad data

This is the deep one, surfacing three
times in this guide: `(error)` stored
as a value instead of a halt
(chapter 5), summaries stepping over
non-numeric rows in silence
(chapter 6), and joins dropping
unpaired records without a count
(chapter 7). Miller's consistent
choice is that one bad row should not
kill a long job. The consequence is
equally consistent: **the tool will
not audit itself for you.** Every
pipeline that matters deserves the
chapter 11 treatment — counts before
and after, an `is_numeric` sweep, an
`is_error` hunt — because the failure
mode Miller protects you from least is
the plausible-looking wrong answer.

## Comparisons are by kind before value

A filter that tests a number also
passes every piece of text, because
text outranks numbers in Miller's
type-ordered comparisons (chapter 6).
Empty fields are their own case: an
absent value is not zero and not the
empty string, and expressions meet it
on its own terms, usually by producing
absent or `(error)`. Clean columns
make honest filters; dirty columns
make filters that need `is_numeric`
riding along.

## CSV is a strict shape

CSV writes one header for a whole
file, so a stream whose records change
shape mid-flight — an unpaired join
record, an `end`-block `emit` — is
refused, mid-stream, after partial
output has already flowed
(chapters 7 and 10). Ragged input is
the same wall from the other side
(chapter 8). JSON and DKVP tolerate
mixed shapes; choosing them for
shape-changing pipelines is a design
decision, not a workaround.

## Some jobs need the whole file

Streaming verbs are frugal, but `sort`
must see everything before its first
output row, grouped summaries hold
their groups, and `join` holds the
whole `-f` file in memory by design.
A joined lookup table of tens of
millions of rows is a database's job
wearing a file's clothes. Miller will
try; the limit is your machine's
memory and your patience, and it
arrives without ceremony.

## The table is flat

Records are name-and-value sets. JSON
with real nesting — objects within
objects, arrays as values — is read
into that flat world with flattening
options and good intentions, and what
the nest *meant* does not survive the
flattening intact (chapter 8). Miller
manipulates nested JSON the way a
mangle irons a suit: the result is
flat and usable and not the original
garment. Nested data's deep tool is
`jq` (chapter 12).

## Numbers are machine numbers

Arithmetic is binary floating point,
with the tails that implies
(chapter 5), and integers beyond
floating point's exact range lose
their last digits the way they do
everywhere else in computing. Money
that must be exact to the cent across
millions of rows belongs in a tool
that does decimal arithmetic, or in
integer cents by convention. Miller
will show you its tails honestly —
`9.600000000000001` is printed, not
rounded away — and that honesty is the
limit's saving grace: the seams are
visible if you look, and this guide's
standing advice is to look.
