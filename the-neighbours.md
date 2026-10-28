# The Neighbours

Miller's shelf is crowded, by design —
it was built to sit among tools people
already know. This chapter characterizes
the neighbours rather than benchmarking
them: what each is for, and where the
border with Miller honestly lies.

## awk, sed, cut, sort, join — the ancestors

The classic text tools remain superb at
what they are: line-oriented, positional,
present on every Unix-like system ever
shipped, and fast in the hand of someone
who thinks in field numbers. For a file
whose shape is fixed and whose columns
are nobody else's business, `awk '$4 >
2'` is shorter than anything in this
guide.

Miller's claim on the same work is
threefold: names instead of positions,
so commands survive a column moving;
formats understood rather than
approximated — awk reads JSON the way a
piano reads a novel — and summaries and
joins as verbs rather than as programs
you write inside a one-liner. The cost
is a binary to install, where awk is
simply *there*. The series' *Text
Processing* guide teaches the ancestors
on their own terms; this guide's
position is only that the two toolboxes
answer different shapes of the same
questions.

## jq — the JSON specialist

`jq` is to JSON what Miller is to
tables: a small language for selecting
and reshaping, by name. For JSON that
is *nested* — objects inside objects,
arrays of arrays, structure that is the
point of the data — jq is the deeper
tool by far, and Miller's flat-record
view is the wrong shape for it. For
JSON that is a table wearing a JSON
coat — an array of flat objects, which
is most API answers and nearly all data
exports — Miller's verbs (`stats1`,
`join`, `sort`) do in one step what jq
expressions do in several, and the
output can leave as CSV without a
second tool. The series' *jq* guide
teaches the specialist; the working
rule is nesting depth: nested belongs
to jq, tabular belongs here.

## yq — the YAML sibling

`yq` does for YAML what jq does for
JSON: path-based reading and editing of
a format whose structure matters. Its
home ground is configuration files —
changing a value inside a nested
document without breaking its comments
and shape — which is document surgery,
not table work. The overlap with Miller
is the conversion corner: both can
cross between formats, and for a flat
table the choice is a matter of which
tool is already in your hand.

## The spreadsheet

A spreadsheet shows you the table, and
for a few hundred rows that a human
will *look* at — checking a figure by
eye, trying a formula, making the
chart — it is the right tool and Miller
has nothing to say. The borders are
repetition and size: the job that runs
every Friday, the file with a million
rows, the answer that must be
reproducible by running a command
again. Those belong to a scriptable
tool, and a Miller command in a script
is the audit trail a spreadsheet
session never leaves. The two meet
politely at TSV (chapter 11's extract
recipe), each doing its own half.

## The database

SQLite, and servers beyond it, hold
data *as* the long-term state of a
system: many readers, writers who must
not corrupt each other, queries indexed
for speed, joins across tables that
stay joined. Miller holds nothing: it
reads files, answers, and exits. When
the data's home is a database, ask the
database — its SQL is this guide's
questions in another dialect (`GROUP
BY` is `-g`, `ORDER BY` is `sort`,
`SELECT` is `cut` and `put`). Miller's
ground is the wide territory before
and beside databases: files in transit,
exports, answers, the table that exists
for an afternoon.

## pandas and the programming languages

A data frame in Python or R is a table
with a whole language attached, and for
analysis that needs statistics beyond
sums, models, plots, or logic no verb
catalogue holds, that is the correct
escalation. Miller is the smaller
commitment: no project, no imports, one
command, done — for the large class of
jobs whose entire logic is *select,
compute, summarize, convert*. The
honest summary of this whole chapter:
Miller is rarely the most powerful tool
for any single job on its shelf. It is
frequently the smallest tool that
finishes it.
