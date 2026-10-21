# Glossary

**Alias** — the `*name` half of YAML's reuse
mechanism: a reference to a block named by an
anchor. Chapter 9.

**Anchor** — a name given to a block of YAML
with `&name`, so other places can reuse it by
alias or merge. Editing the anchor edits the
shared value itself. Chapter 9.

**Assignment (`=`)** — an expression that
outputs the whole document with a path set to a
value. With `-i`, the file is rewritten.
Contrast **update-assignment**. Chapter 4.

**Document** — one YAML value-stream inside a
file; files can hold several, separated by
`---`. Chapter 7.

**Document index** — a document's position in
its file, starting at 0, exposed to expressions
as `document_index`. Chapter 7.

**Expression** — yq's name for its programs:
the jq-like filter text between the command and
the filenames. Takes a document in, produces
documents (or values) out.

**In place (`-i`)** — the flag that makes an
assignment expression rewrite its input file
instead of printing to stdout. Powerful,
unforgiving, and — with document-selecting
expressions — capable of deleting unselected
documents. Chapters 4 and 7.

**Lexer error** — the expression itself
contains something yq's language doesn't have —
a misspelling, or a jq operator that doesn't
exist here. Reported with the position in the
expression and the text it choked on. Chapters
3 and 11.

**Merge key (`<<`)** — the special key that
folds an anchored map into another map at read
time, with the local keys winning over the
inherited ones. Chapter 9.

**Node** — one value in the parsed document
tree: a map, a list, or a scalar, together with
its tag and any comments attached to it.

**Null** — yq's answer for a path that doesn't
exist: not an error, exit code 0, and a value
that will travel silently into whatever reads
it. Chapters 2 and 11.

**Operator** — one verb inside an expression:
`select`, `map`, `del`, `keys`, and the rest.

**Path** — the dotted address of a value,
`.services.web.image`; also, as an operator,
the expression that reports a value's address
as a list. Chapters 2 and 3.

**Properties format** — the flat `key=value`
format of Java `.properties` files, with dots
for nesting and no types. One of yq's `-p`/`-o`
formats. Chapter 6.

**Recursive descent (`..`)** — "this node and
everything beneath it." Chapter 3.

**Round trip** — converting a document away
from YAML and back (typically through JSON).
Faithful about data, fatal to comments and
quoting style. Chapter 5.

**Tag** — a node's YAML type as written in the
spec: `!!str`, `!!int`, `!!bool`, `!!map`. Also
the operator that reports it — the inspection
tool for YAML's type traps. Chapter 8.

**Update-assignment (`|=`)** — like assignment,
but the new value is computed from the old one:
`.retries |= . + 1`. Chapter 4.
