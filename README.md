# yq for Mere Mortals

**yq** is a command-line processor for YAML — the
format of Docker Compose files, CI workflows,
Kubernetes manifests, and half the config files on
a modern system. YAML is pleasant to write and
miserable to edit mechanically: indentation is
meaning, comments matter to the humans who left
them, and `sed` knows none of that. yq reads YAML
structurally instead: you write a small *expression*
naming what you want — a path, a filter, an
assignment — and yq walks the document and prints,
selects, reshapes, or rewrites exactly that.

This guide is the YAML counterpart of the series'
*jq for Mere Mortals*. The expression language is
deliberately jq-like, and readers arriving from
there will find the shape familiar — with a few
vocabulary differences this guide flags where they
bite. Where jq has no in-place editing and makes
you overwrite files by hand, yq edits in place and,
unusually, keeps your comments while doing it.

One warning before anything else: **there are two
unrelated tools called yq.** This guide is about
the one written in Go by Mike Farah (version 4) —
a single binary with its own expression language.
The other, a Python program that wraps jq, is what
several Linux distributions install under the same
name. Chapter 1 shows how to tell which one you
have, because the commands are not interchangeable.

Every command in this guide was run for real
(Go yq **v4.54.1**) against the sample files in
`examples/` — an invented note-taking app's Compose
file, CI workflow, settings, and friends — so every
output can be reproduced, including the failures:
the document an in-place edit silently deleted,
the anchor an edit rewrote by accident, and the
values a round trip through JSON quietly changed.

## What this guide covers

- [**What yq is**](what-yq-is.md) — the two yqs
  and how to tell them apart; getting the Go one;
  the identity expression and what "unchanged"
  really means
- [**Reading YAML**](reading-yaml.md) — paths,
  indexes, slices, iteration, `keys` and `length`,
  several files and several documents at once
- [**Filters and operators**](filters-and-operators.md) —
  `select`, `map`, `sort_by`, `unique`,
  `with_entries`, string functions, `env()`, and
  recursive descent — plus the places the language
  is *not* jq
- [**Changing YAML**](changing-yaml.md) —
  assignment, `|=`, appending, deleting, and
  in-place editing proven with `diff`
- [**Comments and round trips**](comments-and-round-trips.md) —
  the comments yq keeps, the spacing it doesn't,
  and everything a trip through JSON throws away
- [**Converting formats**](converting-formats.md) —
  YAML to JSON and into `jq`, JSON to YAML,
  properties files, and a survey of XML
- [**Multiple documents**](multiple-documents.md) —
  files joined by `---`: selecting one document,
  the in-place trap that deletes the others, the
  safe pattern, and splitting
- [**YAML type traps**](yaml-type-traps.md) —
  `on:`, `no`, and Norway: what yq's parser makes
  of YAML's booby-trapped words, what a YAML 1.1
  parser makes of the same bytes, and the `tag`
  operator that tells you which you have
- [**Anchors and aliases**](anchors-and-aliases.md) —
  shared blocks and merge keys, read and edited —
  including the edit that lands in the anchor
  instead of the display you aimed at
- [**A worked example**](worked-example.md) — a
  release edit done entirely in yq: new image tag,
  new environment entry, new port, proven three
  ways
- [**Errors and debugging**](errors-and-debugging.md) —
  missing files, malformed YAML, bad expressions,
  and the failures that aren't: `null` answers and
  silent string arithmetic
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

The Python yq beyond identifying it in chapter 1 —
it is a different tool with a different manual.
yq as a programming language in the large: custom
operators, `eval-all` merging, and the full
operator catalogue are named where they belong, not
taught. CSV and TSV output, which yq offers for
array-shaped data, are named in chapter 6 but not
exercised. And YAML itself — what the format's
rules are, beyond the traps chapter 8 springs on
purpose — is the series' *YAML for Mere Mortals*;
this guide assumes you have met the format and
want to work on it mechanically.

## How to use this guide

Read chapters 1–4 in order; they install the
model, the reading expressions, and the editing
habit everything else stands on. After that,
chapters stand alone — go to the trap, conversion,
or error you have just met. Work from inside
`examples/`, keep `diff` beside every in-place
edit until you trust it, and check any value you
are about to rely on with `tag` before you rely
on it: in YAML, what a word *is* depends on who
is reading it.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [jq for Mere Mortals](https://paulsullivan33-dev.github.io/jq-for-mere-mortals/)
- [YAML for Mere Mortals](https://paulsullivan33-dev.github.io/yaml-for-mere-mortals/)
- [Kubernetes for Mere Mortals](https://paulsullivan33-dev.github.io/kubernetes-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/yq-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/yq-for-mere-mortals/blob/main/LICENSE-CODE).
