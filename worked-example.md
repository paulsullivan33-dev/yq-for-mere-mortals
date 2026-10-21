# A Worked Example

Everything so far, assembled into one job of
the kind yq gets hired for: a release lands, and
the Compose file needs three changes before the
deploy — the web image moves from 1.4.2 to
1.5.0, a new environment entry turns the beta
features on, and the published port moves from
8080 to 9090. Doing it by hand is three chances
to break indentation; doing it with a text tool
is three chances to hit the wrong `8080`. Doing
it with yq is three named edits — and then,
because this guide doesn't trust edits it hasn't
proven, three checks.

Start the way chapter 4 insists: the working
copy and the snapshot are different files.

```
$ cp compose.yaml release.yaml
$ cp compose.yaml release-before.yaml
```

## The three edits

One command each, each saying exactly what it
changes:

```
$ yq -i '.services.web.image = "fieldnotes/web:1.5.0"' release.yaml
$ yq -i '.services.web.environment += ["APP_FEATURES=beta"]' release.yaml
$ yq -i '.services.web.ports[0] = "9090:80"' release.yaml
```

The first is a plain assignment (chapter 4).
The second appends to the environment *list* —
note the fixture models web's environment as a
list of `KEY=value` strings, so `+=` is the
right verb. The third rewrites the first port
mapping in place — the container side, `:80`,
is untouched because the whole string is the
value and the new string carries it along.

## Proof one: the diff

```
$ diff release-before.yaml release.yaml
5c5
<     image: fieldnotes/web:1.4.2 # bump the tag when a release lands
---
>     image: fieldnotes/web:1.5.0 # bump the tag when a release lands
7c7
<       - "8080:80"
---
>       - "9090:80"
10a11
>       - APP_FEATURES=beta
21d21
< 
```

Three content changes, each exactly the edit
asked for — the image, the port, the added
entry — and the one expected intruder: the
blank line before `volumes:` that yq's
re-typesetting always takes (chapters 1 and 4).
Nothing else moved. The db service, the volumes,
the comments: intact.

## Proof two: read the result back

Diffs compare text; also ask the tool to read
its own work. Building a small report object
from the edited section:

```
$ yq '.services.web | {"image": .image, "ports": .ports, "environment": .environment}' release.yaml
image: fieldnotes/web:1.5.0 # bump the tag when a release lands
ports:
  - "9090:80"
environment:
  - APP_ENV=production
  - APP_PORT=80
  - APP_FEATURES=beta
```

All three changes are in the data, not just in
the text — and there's a small honest curiosity
in the first line: the image value carried its
trailing comment into this freshly built object.
Comments attach to nodes, and the node came
along. Harmless here; worth knowing before you
build reports from commented files.

Direct evidence the comment survived the whole
operation, since chapter 5 made a promise about
it:

```
$ grep -n 'bump' release.yaml
5:    image: fieldnotes/web:1.5.0 # bump the tag when a release lands
```

## Proof three: a different parser agrees

The file's real consumer won't be yq. Final
check — hand the result to an independent YAML
parser (the lab used Python's PyYAML) and ask
for the two values the deploy depends on:

```
$ python3 -c "import yaml; d = yaml.safe_load(open('release.yaml')); print(d['services']['web']['image'], d['services']['web']['ports'])"
fieldnotes/web:1.5.0 ['9090:80']
```

It parses, and it agrees. Three edits, three
proofs, zero hand-edited YAML. Had any proof
failed, `release-before.yaml` is sitting right
there — the copy-first habit means a failed
worked example is an anecdote, not an incident.
