# What yq Is

YAML is the config format of the moment — Compose
files, CI workflows, Kubernetes manifests,
application settings — and it resists the usual
text tools. Its meaning lives in indentation, its
comments are load-bearing documentation, and a
`sed` one-liner that rewrites a version number
today mangles a nested key tomorrow. What you
want is a tool that parses the structure and works
on *that*.

**yq** is that tool. It is a filter in the Unix
sense, modelled deliberately on jq (this series'
*jq for Mere Mortals* teaches the original): YAML
goes in one end, an *expression* — a small program
in yq's own language — says what to do, and YAML
comes out the other end.

```
YAML in  →  expression  →  YAML out
```

Everything in this guide is a variation on that
shape: reading a value out, filtering a list,
rewriting a file in place, converting to another
format entirely.

## First, which yq do you have?

Two unrelated programs answer to the name `yq`,
and the confusion is the single most common yq
problem. This guide is about the one written in
Go by Mike Farah — version 4, a single binary
with its own jq-like expression language. The
other is a Python program by Andrey Kislyuk that
translates YAML to JSON and hands it to jq.

On the machine this guide was built on, the
distribution package is the *Python* one:

```
$ apt-cache policy yq
yq:
  Installed: 3.1.0-3
  Candidate: 3.1.0-3
```

(Debian and Ubuntu describe that package honestly,
if you read the fine print: "Command-line YAML
processor — jq wrapper for YAML documents.")
Debian and Ubuntu don't carry the Go one; it is
installed from the project's GitHub releases page
as one static binary you drop somewhere on your
`PATH`. That is how the copy used here was
installed.

Telling them apart takes one command. The Go yq
names itself and its version:

```
$ yq --version
yq (https://github.com/mikefarah/yq/) version v4.54.1
```

The Python yq's `--version` is a shrug (its real
version lives in its package manager):

```
$ /usr/bin/yq --version
yq 0.0.0
```

And if the version output is ambiguous, behaviour
isn't. The Python tool always answers in JSON —
YAML in, JSON out, because jq is doing the work.
The Go tool answers in YAML:

```
$ echo 'a: 1' | yq '.'
a: 1

$ echo 'a: 1' | /usr/bin/yq '.'
{
  "a": 1
}
```

**Check:** run `yq --version`. If it doesn't say
`mikefarah` and a version starting with `v4`, you
have the other tool, and this guide's commands —
starting with the next one — will not behave as
shown. Get the Go binary first.

## The identity expression

The simplest expression is a single dot, `.`,
"the input, unchanged." Like jq's, it earns its
keep as a validator and a normaliser: yq parses
the whole document — a parse failure stops here,
before any of your logic runs — and prints it
again. But "unchanged" needs an asterisk. Here is
the lab's Compose file passed through untouched:

```
$ yq '.' compose.yaml
# Fieldnotes stack — a small note-taking app.
# Start everything with: docker compose up
services:
  web:
    image: fieldnotes/web:1.4.2 # bump the tag when a release lands
    ports:
      - "8080:80"
    environment:
      - APP_ENV=production
      - APP_PORT=80
    depends_on:
      - db
    restart: unless-stopped
  db:
    image: postgres:16.3
    environment:
      POSTGRES_DB: fieldnotes
      POSTGRES_USER: notes
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

Compare it with the file in `examples/` and two
things have quietly happened. The comments all
survived — that is yq's party trick, and chapter 5
is devoted to it. But the blank line that separated
`services:` from `volumes:` in the original is
*gone*. yq doesn't remember your layout; it holds
the parsed document in memory and prints it in its
own typesetting — two-space indents, its own
choices about quotes. The *data* is unchanged. The
*text* is yq's.

That distinction — same data, re-typeset text —
runs through the whole guide. yq is safe to point
at files whose structure you care about and whose
comments you want to keep. It is not a tool for
preserving a file byte-for-byte, and chapter 4's
`diff`-checking habit exists precisely because of
that.

## It reads JSON too

YAML input is the home turf, but yq detects JSON
by file extension and parses it happily:

```
$ yq '.' data.json
{
  "app": "fieldnotes",
  "version": "1.4.2",
  "ports": [
    8080,
    5432
  ],
  ...
}
```

Two observations to file away. The output is JSON,
not YAML — yq chose the output format to match the
input file's extension (chapter 6 makes formats
explicit with `-p` and `-o` flags, which is how
real conversions are done). And the key order is
the file's own — `app`, `version`, `ports`,
`features`, `maintainers`, exactly as written.
JSON input is simply another document to yq, and
it is handled with the same respect for order as
YAML.

## The sample files

The lab runs against the files in `examples/`, an
invented little world: **fieldnotes**, a
note-taking app nobody has heard of.

| File | What it is |
|---|---|
| `compose.yaml` | A Compose-style stack: `web` and `db` services, with comments |
| `workflow.yaml` | A CI workflow in GitHub Actions shape |
| `settings.yaml` | The app's plain settings file |
| `countries.yaml` | League groups by country code; one code quoted, one not |
| `multi.yaml` | Two documents in one file, separated by `---` |
| `anchors.yaml` | Two displays sharing a settings block by anchor and merge |
| `data.json` | The app described in JSON, for conversions |
| `app.properties` | A Java-style properties file |
| `settings.xml` | The settings again, as XML |
| `broken.yaml` | A settings file with a formatting injury |
| `commands.txt` | The guide's commands, collected as a cheatsheet |

Every quoted output in the chapters ahead came
from running the shown command against these
files with yq v4.54.1. Nothing was tidied,
simulated, or borrowed from the documentation —
including the surprises, of which there are
several.
