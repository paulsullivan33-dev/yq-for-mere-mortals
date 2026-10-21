# Filters and Operators

Reading paths (chapter 2) answers "what's there."
This chapter is for questions with shape: *which*
service restarts automatically, *all* the tags in
shouting capitals, every image anywhere in the
tree. The tools are operators — small verbs joined
by pipes — and the vocabulary is close enough to
jq's that readers of the jq guide can skim. Close,
not identical: the last section collects the
differences that bite.

## select: keep what matches

`select(condition)` passes through only the
values where the condition is true. Services in
the Compose file are a map, not a list, so first
`to_entries` turns it into a stream of
`{key, value}` pairs it can filter:

```
$ yq '.services | to_entries | .[] | select(.value.restart == "unless-stopped") | .key' compose.yaml
web
```

Read it left to right: services → entries →
each entry → keep the ones whose restart policy
matches → print the key. Three of this guide's
workhorse operators appear in that one line, and
the pattern generalises to most "find the thing
that…" questions.

## map, sort, unique

`map(f)` applies `f` to every element of a list.
The lab's settings file has a `tags` list —
`notes, web, club, web, api`, duplicate included
on purpose:

```
$ yq '.tags | map(upcase)' settings.yaml
- NOTES
- WEB
- CLUB
- WEB
- API
```

`sort` orders a list; `unique` removes
duplicates, keeping first occurrences in their
original order:

```
$ yq '.tags | sort' settings.yaml
- api
- club
- notes
- web
- web

$ yq '.tags | unique' settings.yaml
- notes
- web
- club
- api
```

`sort_by` orders a list of maps by a field
inside them — here, the maintainers from the
JSON fixture (converted on the fly; chapter 6
explains the `-p`/`-o` flags):

```
$ yq -p=json -o=yaml '.maintainers | sort_by(.name)' data.json
- name: J. Lindqvist
  role: docs
- name: R. Okafor
  role: lead
```

## with_entries: transform a whole map

`with_entries(f)` runs `f` over each
`{key, value}` pair of a map and reassembles the
results — the way to apply one rule to every
value in a section without naming the keys:

```
$ yq '.limits | with_entries(.value = .value * 2)' settings.yaml
max_upload_mb: 50
retries: 6
```

Every limit doubled, keys untouched. (This
printed a transformed copy; chapter 4 covers
making such changes permanent.)

## String functions

Strings get their own small toolkit. `split`
breaks on a separator, `sub` replaces the first
match of a regular expression:

```
$ yq '.services.web.image | split(":")' compose.yaml
- fieldnotes/web
- 1.4.2

$ yq '.services.web.image | sub("1.4.2"; "1.5.0")' compose.yaml
fieldnotes/web:1.5.0
```

Note `sub`'s semicolon between pattern and
replacement — that is yq's argument separator
throughout, and writing a comma there instead
is a reliable way to earn a lexer error of the
kind collected at this chapter's end.

## env(): values from the shell

Expressions can read environment variables with
`env(NAME)`, which is how scripts parameterise
an edit without string-splicing YAML by hand:

```
$ NEW_TAG=2.0.0 yq '.services.web.image = "fieldnotes/web:" + env(NEW_TAG)' compose.yaml
...
  web:
    image: fieldnotes/web:2.0.0 # bump the tag when a release lands
...
```

(The full reprinted document is elided here; the
version inside it is the one from the
environment.) `+` on strings concatenates — and,
as chapter 11 shows, `+` on a string and a number
concatenates too, which is not always what you
meant.

## Recursive descent: `..`

`..` means "this node and everything under it."
Combined with `select` and `has`, it finds a key
wherever it lives in the tree:

```
$ yq '.. | select(tag == "!!map") | select(has("image")) | .image' compose.yaml
fieldnotes/web:1.4.2
postgres:16.3
```

Every map anywhere that has an `image` key
contributed its image. Two new operators rode
along: `has(key)` tests presence, and `tag`
reports a node's YAML type — `!!map`, `!!str`,
`!!int` — which chapter 8 puts to serious work.

The reverse journey is `path`, which reports
where a value sits:

```
$ yq '.services.web.image | path' compose.yaml
- services
- web
- image
```

## Where yq is not jq

The language is a dialect, not a copy, and the
differences are not politely signposted — a
missing operator is a *lexer error* that quotes
your expression back at you, truncated:

```
$ yq '.tags | map(ascii_upcase)' settings.yaml
Error: 1:15: lexer: invalid input text "cii_upcase)"
```

jq's `ascii_upcase` doesn't exist here; the
operator is named `upcase` (with `downcase`
beside it), as used at the top of this chapter.
Likewise, jq's `getpath` — fetch a value by a
path given as data — has no counterpart:

```
$ yq 'getpath(["services", "db", "image"])' compose.yaml
Error: 1:1: lexer: invalid input text "getpath([\"servic..."
```

The position numbers (`1:15`, `1:1`) are the
line and column *within the expression*, and
the quoted fragment is what's left of your text
at the point the lexer gave up — mangled-looking,
but usually enough to identify the word it
choked on. Chapter 11 reads errors in full; the
lesson here is vocabulary: when you carry a
filter over from jq, expect most of it to work
and a few words to simply not exist.
