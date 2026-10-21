# Writing Criteria

Every guide in this repo is written to the same
standard. If you're adding or editing a guide —
human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language,
   never dumbed-down concepts. Explain jargon on
   first use; collect terms in the glossary.
2. **No hype.** No marketing adjectives, no
   "revolutionary," no promises about what
   something will do for the reader.
3. **Honest about limits.** Say what the thing
   *can't* do. State uncertainty instead of
   smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults,
   and specs are either measured or qualified with
   "depends on your setup." If it wasn't tested,
   say so.
5. **Checkable.** Commands are meant to be run,
   with "Check:" blocks showing what success looks
   like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out
   Windows/Mac/Linux differences. Don't assume a
   GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get
   stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and
   points elsewhere for the rest. No guide
   pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after
   examples, symptom → fix. Scannable beats essay.

## Verification notes for this guide

Standard 3 and 4 applied to this guide,
specifically. The tool under discussion is the
**Go yq, v4.54.1** (mikefarah/yq), installed on
the build machine as the single static binary
from the project's official GitHub release page
and kept at `yq-lab/bin/yq` for the lab runs; the
lab copy's SHA-256 was
`8e34fc29…c702ea5f`. Every quoted output is a
capture from that binary. Working transcripts
are kept with the build (`yq-lab/transcripts/`,
eleven files, `t01`–`t11`, produced by the
harness `yq-lab/run-lab.sh`).

**The other yq.** The build machine's
distribution package (`apt-cache policy yq`:
candidate **3.1.0-3**, the kislyuk Python jq
wrapper) was installed to `/usr/bin/yq` solely
so chapter 1 could capture the two tools side by
side — the `yq 0.0.0` version string and the
JSON answer to the identity filter. It is used
nowhere else in the guide.

**Supporting tools in captures:** jq **1.7**
(the sibling guide's tool) for the chapter 6
pipe proof, and Python **3.12.3** with PyYAML
**6.0.3** for the independent-parser reads in
chapters 8 and 10 — the YAML 1.1 readings and
the worked example's third proof are PyYAML's
actual output, not paraphrase.

**The lab.** Fixtures are the files in
`examples/` — an invented note-taking app
("fieldnotes"), its Compose-style stack, CI
workflow, settings, a two-document file, an
anchors file, and small JSON/properties/XML
companions — kept pristine in the lab; every
mutating command ran against a copy, and every
claimed change is proven in the transcripts by
`diff`, `cat`, or `grep` against the untouched
original rather than by exit code.

**Verified by running, per chapter:**

- *Ch. 1:* `--version` from both yq's; the
  identity filter through both (YAML out vs JSON
  out); the identity pass over `compose.yaml`
  (comments kept, the blank line before
  `volumes:` dropped) and over `data.json` (JSON
  out, key order preserved).
- *Ch. 2:* path, index, slice, and `.[]`
  reads; `keys` and `length`; a multi-document
  read returning both `kind`s; a two-file read
  whose second result is `null`; `filename`.
- *Ch. 3:* `select` over `to_entries`;
  `map`/`sort`/`unique`; `with_entries` doubling
  a map's values; `sort_by` over the JSON
  fixture; `split` and `sub`; `env()` driving an
  assignment; recursive descent `..` with `has`
  and `tag`; `path`. The two jq-isms that fail —
  `ascii_upcase`, `getpath` — are captured
  lexer errors, not assertions.
- *Ch. 4:* set, append (`+=` on two lists),
  `del`, and `|=` edits, each closed out by a
  `diff` against a pre-edit snapshot. The diffs
  show the asked-for changes and the layout
  changes that came free: the dropped blank line
  (every rewrite) and the unquoted new port
  entry beside its quoted sibling.
- *Ch. 5:* an in-place edit elsewhere in the
  file leaving all three comments in place
  (`grep -n '#'`), then the JSON round trip:
  `grep -c '#'` returns **0**, and the diff
  enumerates the losses — comments, the port's
  quotes, and `db-data:` returning as
  `db-data: null`.
- *Ch. 6:* YAML→JSON; the pipe into jq 1.7
  returning the reshaped object; JSON→YAML with
  key order intact; properties in (values
  quoted back as strings: `"8080"`, `"true"`)
  and properties out (dotted keys, indexed list
  entries, the head comment carried); XML in
  and XML out on the element-only fixture.
- *Ch. 7:* `document_index`; selecting document
  1 for reading; **the trap, captured**: the
  same select expression with `-i` leaves a
  file containing only document 2 — document 1,
  comments and all, deleted, exit 0; the
  parenthesised-path assignment changing one
  line with both documents surviving, proven by
  a one-hunk diff; `--split-exp` writing
  `kiosk-settings.yml` and `kiosk.yml`, the
  latter `cat`ed.
- *Ch. 8:* `on`, `yes`, `off`, and unquoted
  `NO` all tagging `!!str` under yq, with `.on`
  addressing the workflow's trigger key
  directly; the same three files under PyYAML
  producing the `True` key, the `false` Norway,
  and three re-typed feature flags; the quoted
  `"NO"` surviving both parsers; genuine
  `!!bool` and `!!int` tags for contrast.
- *Ch. 9:* the anchor block and a merged map
  read raw (`<<:` unexpanded), then resolved
  reads through the merge (lobby 70, hall 85);
  the anchor edit flowing to the inheritor
  (90) while the override holds (85); **the
  gotcha, captured**: assigning through the
  lobby's inherited key rewrote
  `defaults.brightness` to 55 in the file,
  leaving the lobby with no key of its own.
  The `--yaml-fix-merge-anchor-to-spec` read
  is captured too (warning gone, value 85,
  unchanged on this fixture).
- *Ch. 10:* the release job end to end —
  three `-i` edits, a three-hunk-plus-blank-line
  diff, a constructed report object read back
  (with the image's trailing comment visibly
  attached to the value), `grep` confirming the
  comment, and PyYAML parsing the result and
  agreeing on image and ports.
- *Ch. 11:* missing file, malformed YAML
  (`broken.yaml` — a missing space after a
  colon, plus its knock-on indent), an unclosed
  bracket, an unknown operator, all verbatim
  with exit **1**; the two non-errors captured
  at exit 0: a missing path printing `null`, and
  a string plus a number concatenating to
  `fieldnotes/web:1.4.21`; `has` and `//`
  behaving as the defences claim.

**A rejected hypothesis, for the record:** an
early build of the lab appeared to show yq
sorting JSON keys alphabetically on output.
That was an artefact of the fixture file itself
having been written in alphabetical order. The
fixture was rewritten deliberately unordered
(`app, version, ports, features, maintainers`)
and the lab re-run: order is preserved, and the
guide says so. The chapters' earlier drafts
were corrected before publication.

**Capture hygiene (declared trims):** the
`[exit N]` lines annotating each transcript
block are the harness's, not the tool's. stderr
is merged into the transcripts, so yq's
merge-anchor warning appears in `t09` where it
actually occurred — repeatedly, with the run's
timestamps; the anchors chapter describes the
warning once, in prose, and quotes values from
stdout only. Two chapter blocks elide with a
marked `...`: chapter 3's `env()` result (the
middle of the reprinted document) and chapter
9's `cat` of the gotcha file (its unchanged
head comments). No output line was altered.

**Documented-only (not run), and why:**

- *The Python yq beyond chapter 1's
  identification.* It is a separate tool whose
  language is jq itself; evaluating it belongs
  to the jq guide's territory. Here it is
  named, versioned, and behaviourally
  distinguished — no more.
- *CSV and TSV output.* yq lists them; the
  guide names them in chapter 6 and exercises
  only the four formats the lab captured (YAML,
  JSON, properties, XML).
- *XML attributes, namespaces, and mixed
  content.* The fixture XML is element-only by
  design; chapter 6 states the survey's depth
  in place rather than implying coverage.
- *The merge-anchor spec flag beyond the
  captured read.* What the flag does to deeper
  merge orderings is upstream documentation
  territory; the guide claims only what `t09`
  shows — warning silenced, this read
  unchanged.
- *`eval-all`, custom operators, and the full
  operator catalogue* — named in the README's
  scope, not taught.
- *Installation on Windows and macOS.* The
  binary is cross-platform and the release page
  is the route, but the builds and captures here
  are Linux; nothing platform-specific is
  asserted beyond that.
- *The workflow fixture is never executed.* It
  exists to be parsed — including by the YAML
  1.1 reader in chapter 8 — and no claim is
  made about it as a working CI definition.

Specimen discipline, for the record: every name,
app, and value in the fixtures is invented for
the guide; no real system, service, or person
appears. The lab ran entirely on local files.
Nothing in the guide requires a network, a
container runtime, or an account.
