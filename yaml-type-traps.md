# YAML Type Traps

YAML's most famous booby trap is a list of
innocent words. In the original YAML 1.1 rules,
`y`, `yes`, `n`, `no`, `true`, `false`, `on`,
and `off` are all booleans — which means the
country code for Norway (`NO`), a CI file's `on:`
key, and a settings file's `signup: on` are all
one inattentive parser away from becoming
`false`, `true`, and `true`. The series' *YAML
for Mere Mortals* tells that story as a story
about the format. This chapter tells it as a
story about *readers* — because the same bytes
parse differently in different tools, and yq is
how you find out which reading you're holding.

## yq's reading: the words stay words

The lab's CI fixture opens with the classic
trap, a top-level `on:` key:

```
$ yq 'keys' workflow.yaml
- name
- on
- jobs

$ yq '.on.push.branches' workflow.yaml
[main]
```

yq kept `on` as a plain string key and `.on`
addresses it without ceremony. The settings file
is the same story, three traps deep:

```
$ yq '.features' settings.yaml
signup: on
comments: yes
beta_dashboard: off

$ yq '.features.signup' settings.yaml
on
```

Not *converted to* boolean and back — never a
boolean at all. The `tag` operator proves it, by
reporting the type the parser actually assigned:

```
$ yq '.features.signup | tag' settings.yaml
!!str
```

And the country codes, unquoted `NO` included:

```
$ yq -o=json '.group_a' countries.yaml
[
  "NO",
  "SE",
  "DK"
]

$ yq '.group_a[0] | tag' countries.yaml
!!str
```

Go yq's parser (the `yaml.v3` library) resolves
plain scalars the YAML 1.2 way: only `true` and
`false` are booleans, and the 1.1 word list —
`yes`, `no`, `on`, `off`, Norway — stays string.
For contrast, a genuine boolean and a genuine
integer, tagged:

```
$ yq '.server.tls | tag' settings.yaml
!!bool

$ yq '.server.port | tag' settings.yaml
!!int
```

## The other reading: same bytes, YAML 1.1

Now the *same files* through a YAML 1.1 parser.
Python's PyYAML is the one most likely to be
sitting in a pipeline near you, so the lab read
the fixtures with it (Python 3.12, PyYAML 6.0.3):

```
$ python3 -c "import yaml; print(list(yaml.safe_load(open('workflow.yaml')).keys()))"
['name', True, 'jobs']
```

The `on:` key became the boolean `True`. Any
program looking for the string `"on"` in that
dict finds nothing; the workflow's trigger
section is filed under a boolean. The country
codes:

```
$ python3 -c "import yaml, json; print(json.dumps(yaml.safe_load(open('countries.yaml'))))"
{"group_a": [false, "SE", "DK"], "group_b": ["NO", "FR", "DE"]}
```

Group A's Norway is now `false`. And the
settings:

```
$ python3 -c "import yaml, json; print(json.dumps(yaml.safe_load(open('settings.yaml'))['features']))"
{"signup": true, "comments": true, "beta_dashboard": false}
```

Three feature flags, all silently re-typed.

## What to do with this

Neither parser is lying; they're implementing
different editions of the spec, and the file
doesn't say which edition it was written for.
The practical rules:

1. **Quote the booby-trapped words when you
   write YAML by hand.** The fixture's group B
   has its code quoted — `"NO"` — and it
   survived *both* parsers as a string (group B
   reads `"NO"` in the PyYAML output above, and
   `!!str` under yq). Quotes are the one spelling
   every parser honours.
2. **Inspect with `tag` before you trust.** When
   a value matters — a flag a deploy branches
   on, a code a lookup indexes by — one
   `yq '.path | tag'` tells you what your tool
   made of it, instead of what you hoped.
3. **Remember yq can't show you the other
   reading.** yq reports its own parser's view,
   correctly. If a file also feeds a YAML 1.1
   tool — an older Python script, some Ruby and
   Java libraries — yq's clean bill of health
   doesn't cover them. The failure mode is a
   file that looks perfect in every yq check and
   still feeds `false` to the program that
   mattered.

**Check:** the quoted code in group B tags as
`!!str` under yq — `yq '.group_b[0] | tag'
countries.yaml` — and stays the string `"NO"` in
the PyYAML output above. Two parsers, one
survivor: the quoted word.
