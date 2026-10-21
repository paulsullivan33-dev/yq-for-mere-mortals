# Reading YAML

Most of what yq does in the wild is reading: pull
one value out of a file too big to open, list
what's in a section, check what an image tag is
set to before a deploy script changes it. This
chapter is the reading toolkit — paths, indexes,
iteration — and every expression here also works
as the left-hand side of the edits in chapter 4.
Learn to name a thing precisely and you have
already learned how to change it.

## Paths

A path is a chain of keys, written the way you'd
hope:

```
$ yq '.services.web.image' compose.yaml
fieldnotes/web:1.4.2
```

Ask for a whole section and you get it, as YAML:

```
$ yq '.services.db' compose.yaml
image: postgres:16.3
environment:
  POSTGRES_DB: fieldnotes
  POSTGRES_USER: notes
volumes:
  - db-data:/var/lib/postgresql/data
```

Scalar answers print bare — no quotes, no key —
which makes them easy to feed to scripts
(`TAG=$(yq '.services.web.image' compose.yaml)`).

## Indexes and slices

Lists are indexed from zero, in square brackets:

```
$ yq '.services.web.ports[0]' compose.yaml
8080:80
```

`[]` with nothing in it *iterates* — it asks for
every element, one result after another:

```
$ yq '.services.web.environment[]' compose.yaml
APP_ENV=production
APP_PORT=80
```

And slices take a range, start-inclusive and
end-exclusive, just as in jq (and Python):

```
$ yq '.services.web.environment[0:1]' compose.yaml
- APP_ENV=production
```

Note the difference in shape between the last
two: `[]` produced a stream of bare strings,
while the slice produced a one-item *list*. The
slice is still a list because you asked for a
piece of one; the iteration unwrapped it.

Iteration works on maps too. `.services[]` walks
the values under `services`, and a pipe carries
each one onward:

```
$ yq '.services[] | .image' compose.yaml
fieldnotes/web:1.4.2
postgres:16.3
```

That pattern — iterate, then project one field —
is the single most-used yq shape there is.

## Asking about the container

`keys` lists a map's keys, and `length` counts a
map's entries or a list's items:

```
$ yq '.services | keys' compose.yaml
- web
- db

$ yq 'keys' compose.yaml
- services
- volumes

$ yq '.services | length' compose.yaml
2
```

`keys` preserves the file's order — `web` before
`db`, as written. Order is information in a
config file (the first environment entry, the
primary server), and yq treats it that way, for
JSON input as much as for YAML.

## Several documents, several files

A YAML file can hold more than one document,
separated by `---` lines — Kubernetes manifests
live on this. Reading expressions apply to *every*
document in the file, and multi-document results
are separated by `---` in the output:

```
$ yq '.kind' multi.yaml
Settings
---
Service
```

Chapter 7 is devoted to multi-document files;
for now, know that a naive read sees all of them.

Multiple *files* work the same way — name them
all, and the expression runs against each in
turn:

```
$ yq '.server.port' settings.yaml compose.yaml
8080
---
null
```

Two lessons in that small output. The `---`
separates the per-file results. And the second
result is `null`, not an error: `compose.yaml`
has no `.server.port`, and yq's answer to a
missing path is `null`, exit code 0. That is a
deliberate design choice (jq makes the same one),
and chapter 11 covers what it means for scripts
that trust the output.

When several files are in play, the `filename`
operator says which one an expression is looking
at:

```
$ yq 'filename' compose.yaml
compose.yaml
```

**Check:** in `examples/`, run
`yq '.services.web.image' compose.yaml`. You
should get `fieldnotes/web:1.4.2`, bare, one
line. If the value arrives wrapped in double
quotes — a JSON string — your `yq` is the Python
tool, and you should go back to chapter 1.
