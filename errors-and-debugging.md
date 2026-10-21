# Errors and Debugging

yq's failures are fewer and quieter than jq's,
which is not entirely a compliment: where it
does fail it says so plainly, but two of the
most dangerous outcomes in this guide aren't
failures at all. This chapter collects the real
errors first — each produced by the command
shown, on yq v4.54.1 — then the non-errors that
deserve your suspicion, then the defensive
patterns.

The geography, as with jq: results go to
**stdout**, errors to **stderr**. In a pipeline,
only the results flow onward; an errored yq in
the middle of a pipe leaves the downstream tool
holding nothing, which is why noticing exit
codes matters.

## The failure classes

Unlike jq's several exit codes, everything in
this table exits **1**:

| Failure | Exit |
|---|---|
| File can't be opened (missing, unreadable) | 1 |
| Input won't parse as the named format | 1 |
| Expression won't compile (bad syntax, unknown operator) | 1 |

### The file isn't there

```
$ yq '.services.web.image' nope.yaml
Error: open nope.yaml: no such file or directory
```

Plain, complete, and the first thing to check
when a script's yq step "does nothing": the
relative path is being resolved from a different
working directory than you think.

### The YAML is broken

```
$ yq '.server.port' broken.yaml
Error: bad file 'broken.yaml': go-yaml load error in scanner at L4.C8: mapping values are not allowed in this context
```

`broken.yaml` lost the space after `port:`'s
colon and gained an indent problem; the scanner
points at line 4, column 8 — the start of the
line it couldn't make sense of — and names the
rule it broke. YAML parse errors are positional
for the same reason JSON's are: the parser knows
exactly where the structure stopped working.
Fix the file, not the expression.

### The expression is broken

Two shapes. A syntax error — an unclosed
bracket:

```
$ yq '.services[' compose.yaml
Error: bad expression, could not find matching `]`
```

And an unknown operator, reported by the lexer
with the position inside the expression and the
tail of your text it choked on:

```
$ yq '.tags | frobnicate' settings.yaml
Error: 1:9: lexer: invalid input text "frobnicate"
```

Chapter 3 met this class twice — `ascii_upcase`
and `getpath`, borrowed from jq, dying at
columns 15 and 1 respectively. The fragment
quoted can look mangled (`"cii_upcase)"` —
that's your text *from the failure point*, not
a typo in the error). Read it as "the first
thing I didn't recognise starts here," and it
usually names the culprit.

## The non-errors

### Missing paths answer null

```
$ yq '.missing.deep.path' compose.yaml
null
```

Exit 0. No complaint at any depth: yq walks as
far as the document goes and reports `null` for
the rest. For interactive use that's pleasant.
In a script it's the failure mode that ships —
a mistyped path becomes a `null` that becomes a
broken deploy two steps later, with every exit
code green. The defences are in the next
section; the awareness is most of it.

### + is not always arithmetic

```
$ yq '.services.web.image + 1' compose.yaml
fieldnotes/web:1.4.21
```

You might have wanted a type error. What you
got is string concatenation: `+` joins strings,
and a number standing next to a string joins as
text. `1.4.2` met `1` and produced `1.4.21`,
exit 0. yq's arithmetic is dynamically typed in
a way that prefers producing *something* — which
makes `tag` checks (chapter 8) part of writing
any expression whose inputs you don't control.

## Defensive patterns

Two operators carry most of the defensive
weight. `has` asks whether a key exists, as a
real boolean you can branch on:

```
$ yq '.services | has("web")' compose.yaml
true
```

And the alternative operator `//` replaces a
missing or `null` result with a default, at the
point of use:

```
$ yq '.services.web.missing // "not set"' compose.yaml
not set
```

Between them, most production expressions end
up shaped like: select what exists (`has`,
`select`), and never let a bare path's `null`
travel further than the expression that asked
for it (`//`). Add the habit from the errors
above — when a pipeline step surprises you,
rerun the yq command alone, read stderr, and
check the exit code the pipeline was hiding —
and you have this tool's whole debugging
discipline. It is shorter than jq's, because
yq fails less often. Stay suspicious of the
silence anyway: in this tool, the dangerous
outcomes are the ones that print something
plausible and exit 0.
