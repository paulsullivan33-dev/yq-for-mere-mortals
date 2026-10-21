# Changing YAML

Reading is safe. Changing is where yq earns its
place over a text editor in a script — and where
it can do real damage, because it writes whole
files. This chapter builds the editing habit the
way the lab practised it: copy the file, edit the
copy, and prove every change with `diff` before
believing it. Nothing here is quoted from
documentation; every diff below is real.

## Assignment

An assignment expression outputs the whole
document with one path set to a new value:

```
yq '.services.web.image = "fieldnotes/web:1.5.0"' work.yaml
```

Without `-i` that prints the changed document to
stdout and leaves the file alone — the safe way
to preview. With `-i` (in place) it rewrites the
file:

```
$ cp compose.yaml before.yaml
$ cp compose.yaml work.yaml
$ yq -i '.services.web.image = "fieldnotes/web:1.5.0"' work.yaml
$ diff before.yaml work.yaml
5c5
<     image: fieldnotes/web:1.4.2 # bump the tag when a release lands
---
>     image: fieldnotes/web:1.5.0 # bump the tag when a release lands
21d20
< 
```

The diff is the lesson, all three parts of it.
The image tag changed — exactly the edit asked
for. The trailing comment *on the edited line
itself* survived — chapter 5 takes that up. And
`21d20`: the blank line before `volumes:` is
gone, the same re-typesetting chapter 1 showed
in the identity filter. yq rewrites the whole
file in its own layout every time it edits.
Comments are preserved; blank lines and other
pure spacing are not guaranteed to be. Diff your
first edits on any file you care about, and
you'll know within one command whether yq's
layout changes are acceptable there.

Readers of the jq guide will notice what's
missing: the temp-file ritual. jq has no `-i`,
so its edits go through a temporary file and a
`mv`. yq's `-i` removes that ceremony — and the
safety net it accidentally provided. The copy
(`cp file before.yaml`) is now a step you choose
to take, and you should.

## Appending with +=

`+=` adds to a list in place. Two appends, then
the proof:

```
$ yq -i '.services.web.environment += ["APP_DEBUG=false"]' work.yaml
$ yq -i '.services.web.ports += ["8443:443"]' work.yaml
$ diff mid.yaml work.yaml
7a8
>       - 8443:443
10a12
>       - APP_DEBUG=false
```

(`mid.yaml` is a snapshot of `work.yaml` taken
between the changes, so this diff isolates just
these two edits.) Note the diff's line for the
port: the new entry prints as `8443:443`,
unquoted, where the existing `"8080:80"` in the
same list wears quotes — yq kept the original
entry's style and chose its own for the new one.
Both parse back as the same kind of string, but
if your file's quoting is load-bearing for other
tools, this is the sort of detail the diff habit
catches.

## Deleting with del()

`del(path)` removes a key (or a list item):

```
$ yq -i 'del(.services.db.environment.POSTGRES_USER)' work.yaml
$ diff mid2.yaml work.yaml
20d19
<       POSTGRES_USER: notes
```

One line removed, nothing else moved. Deleting
is the edit where the copy-first habit matters
most: there is no recycle bin, and the value is
gone from the only place it was written down.

## |= : update by computation

Plain `=` sets a literal. The update-assignment
`|=` computes the new value from the old one —
here, incrementing a counter in the settings
file:

```
$ yq -i '.limits.retries |= . + 1' limits.yaml
$ diff settings.yaml limits.yaml
15c15
<   retries: 3
---
>   retries: 4
```

Inside the right-hand side, `.` is the current
value at that path. (`limits.yaml` is a copy of
`settings.yaml`; the diff is against the
original.) The same operator drives chapter 3's
`with_entries` doubling, and any transformation
of the form "the new value is a function of the
old."

## The discipline, summarised

| Step | Why |
|---|---|
| `cp file.yaml before.yaml` | yq has no undo; the copy is it |
| Edit without `-i` first | Preview the whole reprinted document |
| Edit with `-i` | One change per command while you're learning |
| `diff before.yaml file.yaml` | Prove the change is exactly the change — including the layout changes you didn't ask for |

**Check:** after the first diff above, the only
*content* line changed is the image line, and
the only other difference is a deleted blank
line. If your diff shows reformatting across the
whole file, something else is going on — most
often a file whose indentation isn't the two
spaces yq emits — and that's worth knowing
before the file is a production one.
