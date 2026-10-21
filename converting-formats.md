# Converting Formats

Sooner or later YAML meets a tool that wants
something else: a script that speaks JSON, a Java
service that reads properties files, a human who
can only open XML. yq converts as it processes —
two flags name the input and output formats, and
every expression in this guide works unchanged
on either side of the conversion:

| Flag | Meaning |
|---|---|
| `-p` (or `--input-format`) | What to parse: `yaml`, `json`, `props`, `xml`, … |
| `-o` (or `--output-format`) | What to print: `yaml`, `json`, `props`, `xml`, … |

Without flags, yq guesses the input format from
the file extension (chapter 1's `data.json` read
itself as JSON) and prints YAML — except that a
`.json` input gets JSON out. Guessing is fine at
a terminal; in scripts and conversions, say what
you mean with the flags.

## YAML to JSON, and onward to jq

The workhorse conversion. The web service, as
JSON:

```
$ yq -o=json '.services.web' compose.yaml
{
  "image": "fieldnotes/web:1.4.2",
  "ports": [
    "8080:80"
  ],
  "environment": [
    "APP_ENV=production",
    "APP_PORT=80"
  ],
  "depends_on": [
    "db"
  ],
  "restart": "unless-stopped"
}
```

That's the bridge into the JSON ecosystem — and
the proof is to walk across it. The sibling
guide's tool takes it from there:

```
$ yq -o=json '.services.web' compose.yaml | jq '{image, ports, restart}'
{
  "image": "fieldnotes/web:1.4.2",
  "ports": [
    "8080:80"
  ],
  "restart": "unless-stopped"
}
```

Two specialists, each doing its half: yq got the
data out of YAML faithfully (this is exactly the
job the *other* yq exists to do by wrapping jq —
the Go tool just hands over the JSON and steps
aside), and jq reshaped it. Neither tool had to
understand the other's format beyond the pipe.

## JSON to YAML

The reverse crossing, used when a JSON-speaking
tool produces something a human has to keep:

```
$ yq -p=json -o=yaml '.' data.json
app: fieldnotes
version: 1.4.2
ports:
  - 8080
  - 5432
features:
  signup: true
  comments: false
maintainers:
  - name: R. Okafor
    role: lead
  - name: J. Lindqvist
    role: docs
```

Order survives the crossing intact — `app`,
`version`, `ports`, `features`, `maintainers`,
as the JSON listed them. This is the safe
direction for comments, too: there were none to
lose. The dangerous direction was chapter 5's.

## Properties files

Java-style `.properties` — flat `key=value`
lines, dots for nesting — parses into the same
document model:

```
$ yq -p=props -o=yaml '.' app.properties
server:
  port: "8080"
  host: 0.0.0.0
logging:
  level: info
features:
  signup: "true"
```

The quotes are the lesson: properties files have
no types, every value is text, and yq keeps them
text — quoting `"8080"` and `"true"` in the YAML
precisely so they *don't* come back as a number
and a boolean. Going the other way flattens a
document into dotted keys, lists becoming numbered
entries, and the file's head comment rides along:

```
$ yq -o=props '.' settings.yaml
# Fieldnotes application settings.
server.host = 0.0.0.0
server.port = 8080
server.tls = false
features.signup = on
...
tags.0 = notes
tags.1 = web
```

Note `features.signup = on` on the way out: the
YAML string `on` stays the string `on` (chapter 8
explains whose parser would disagree) — but a
properties file read by something expecting
`true` will not thank you for it. Conversions
preserve data; they don't reconcile what
different ecosystems *mean* by it.

## XML, at survey depth

XML parses and prints too. Predictably, the
mapping is the plain one — elements become keys,
text becomes values:

```
$ yq -p=xml -o=yaml '.' settings.xml
settings:
  server:
    host: 0.0.0.0
    port: "8080"
  logging:
    level: info
```

(Again the port is quoted into stringhood: XML
text is text.) Printing back to XML reproduces
the document element for element. What this
survey does *not* establish: how attributes,
namespaces, and mixed content survive — real XML
in the wild has all three, and yq's conventions
for them are a topic of their own. Treat XML
support as a way to *read* simple configuration
XML and to emit simple XML, and verify anything
more intricate against your own files before
trusting it. The same honesty applies to the
formats this chapter doesn't open at all: yq
also lists CSV and TSV among its outputs, for
array-shaped data; they are named here, not
exercised.
