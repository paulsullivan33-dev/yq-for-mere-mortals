# Comments and Round Trips

YAML files accumulate comments the way kitchens
accumulate notes on the fridge: the reason a
value is what it is, written beside the value, by
someone who knew they'd forget. Most YAML tools
throw those away without mentioning it — they
parse to a data structure, and comments aren't
data. This chapter is about the two ways comments
fare in yq's world: editing *with* yq, which
keeps them, and passing *through JSON*, which
destroys them completely. The difference decides
how you should route any automated edit.

## Editing with yq: comments stay

The Compose file carries three comments — two
at the top, one trailing the image line. Edit a
*different* service's image, in place, and list
what comments remain:

```
$ yq -i '.services.db.image = "postgres:16.4"' commented.yaml
$ grep -n '#' commented.yaml
1:# Fieldnotes stack — a small note-taking app.
2:# Start everything with: docker compose up
5:    image: fieldnotes/web:1.4.2 # bump the tag when a release lands
```

All three, unmoved. Chapter 4's diff showed the
stronger version: the comment trailing the very
line being edited survived its own edit. yq
attaches comments to the nodes they annotate and
carries them through the rewrite.

What it does *not* promise is your layout.
Chapter 1's identity pass and chapter 4's diff
both lost the blank line before `volumes:`.
Think of it as: yq preserves everything that is
part of the document — values, comments, key
order — and re-typesets everything that is
merely presentation. For config files, that
trade is almost always worth making; for a file
a human has carefully formatted as documentation,
it might not be, and the diff habit will tell
you in one look.

## The JSON round trip: comments die

Now route the same file through JSON and back —
the path data takes when a pipeline includes a
tool that only speaks JSON (or the other yq,
which is one):

```
$ yq -o=json '.' compose.yaml > compose.json
$ yq -p=json -o=yaml '.' compose.json > roundtrip.yaml
$ grep -c '#' roundtrip.yaml
0
```

Zero comments. (`grep -c` prints the count, 0,
and itself exits 1 for "no matches" — the 1 is
grep's, and zero is the finding.) JSON has no
comment syntax, so there is nowhere for them to
live during the crossing. The full diff shows
they aren't the only casualties:

```
$ diff compose.yaml roundtrip.yaml
1,2d0
< # Fieldnotes stack — a small note-taking app.
< # Start everything with: docker compose up
5c3
<     image: fieldnotes/web:1.4.2 # bump the tag when a release lands
---
>     image: fieldnotes/web:1.4.2
7c5
<       - "8080:80"
---
>       - 8080:80
21d18
< 
23c20
<   db-data:
---
>   db-data: null
```

Read the losses in ascending order of subtlety.
The comments are gone, all of them — expected,
if you think about JSON for a moment. The blank
line went too, but yq itself does that anyway.
The port lost its quotes: `"8080:80"` came back
as `8080:80`. The *value* is the same string, and
yq chose not to quote it on the way out — but a
file's quoting style is sometimes a deliberate
signal to the next reader, human or parser, and
it's gone.

And the last hunk is the one that changes data:
`db-data:` — a key with an empty value, which
YAML reads as null — came back as `db-data: null`,
spelled out. Same meaning to a YAML parser. Not
the same file, and to a tool that distinguishes
"present and empty" from "present and null" in
its own way, not necessarily the same behaviour
either.

## The rule this teaches

| Route | Comments | Key order | Quoting style | Empty values |
|---|---|---|---|---|
| Read and write with yq | Kept | Kept | Mostly kept | Kept |
| YAML → JSON → YAML | **Lost** | Kept | Re-chosen | Spelled out as `null` |

So: do YAML edits in YAML, with a tool that
models comments — which is to say, with this
one, used the way chapters 4 and 10 use it.
Convert to JSON when another tool genuinely
needs JSON (chapter 6 does exactly that, into
jq), but treat the conversion as one-way for
editing purposes. A file that has been through
JSON is a *new* file with the same data, and
should be reviewed like one.
