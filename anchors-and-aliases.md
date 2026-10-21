# Anchors and Aliases

YAML lets a document write a block once, name
it with an *anchor* (`&name`), and reuse it
elsewhere with an *alias* (`*name`). The `<<`
*merge key* folds an anchored block into another
map, which then overrides individual keys. It's
how Compose files and CI workflows share
configuration without copy-paste — and it's
where yq's document model gets genuinely
interesting, because an edit has to decide *which
copy* of a value it means. This chapter reads
the lab's fixture, edits it the intended way,
and then makes the instructive mistake.

The fixture: two clubhouse displays sharing
brightness, volume, and sleep settings, with the
hall display overriding brightness for itself:

```yaml
defaults: &display_defaults
  brightness: 70
  volume: 20
  sleep_minutes: 10

lobby_display:
  <<: *display_defaults
  name: Lobby

hall_display:
  <<: *display_defaults
  name: Hall
  brightness: 85
```

## Reading: raw structure vs resolved values

Ask for a display as a whole and yq shows the
document *as written* — merge key unexpanded,
the alias still an alias:

```
$ yq '.hall_display' anchors.yaml
<<: *display_defaults
name: Hall
brightness: 85
```

Ask for a specific key, though, and the merge is
resolved for you — the lobby has no `brightness`
of its own, yet:

```
$ yq '.lobby_display.brightness' anchors.yaml
70
```

Seventy, inherited from the anchor. The hall's
own key wins over the inherited one:

```
$ yq '.hall_display.brightness' anchors.yaml
85
```

One wrinkle rides along with every read that
resolves a merge: yq prints a warning to
**stderr** about its merge handling not being
the YAML spec's default (`--yaml-fix-merge-anchor-to-spec
is false; causing merge anchors to override...`).
The results on stderr don't touch the values on
stdout — the reads above are quoted without it —
and passing the flag it names,

```
$ yq --yaml-fix-merge-anchor-to-spec '.hall_display.brightness' anchors.yaml
85
```

silences the warning and, on this file, changes
nothing: the display's own key wins under either
precedence. What the flag *would* change is
subtler merge orderings than this fixture's; the
guide records the warning and the flag, and
doesn't claim more than the capture shows.

## Editing the anchor: the intended lever

To change the shared default for everyone,
edit the anchor block itself:

```
$ yq -i '.defaults.brightness = 90' a1.yaml
$ yq '.lobby_display.brightness' a1.yaml
90
$ yq '.hall_display.brightness' a1.yaml
85
```

The lobby inherits the new 90; the hall keeps
its own 85. This is anchors working as designed
— one edit, every inheritor updated, every
override respected.

## The gotcha: editing through an inheriting key

Now the mistake. The lobby's brightness reads
as 70, so the obvious edit is to set it —
give the lobby 55, say, on its own:

```
$ yq -i '.lobby_display.brightness = 55' a2.yaml
$ cat a2.yaml
...
defaults: &display_defaults
  brightness: 55
  volume: 20
  sleep_minutes: 10
lobby_display:
  <<: *display_defaults
  name: Lobby
hall_display:
  <<: *display_defaults
  name: Hall
  brightness: 85
```

There is still no `brightness` key under
`lobby_display`. The 55 went into the **anchor**
— because the value the lobby shows for that key
*lives* in the anchor, and yq's assignment
followed the path to where the value actually
sits and changed it there. You asked to dim one
display and instead redefined the default for
every display that inherits it (the hall is
spared only by its own override). Nothing
warned; the exit code was 0; only reading the
file afterward shows what happened.

The rules this teaches:

- **Reading through a merge is safe** — yq
  resolves inherited values transparently.
- **To change a shared value, edit the anchor
  on purpose**, as in the previous section.
- **To give one map its own value for an
  inherited key, you cannot just assign through
  the merge** — yq will spend the edit on the
  anchor. The file needs restructuring instead
  (write the key explicitly into that map with
  an editor, then let yq maintain it), or the
  anchor needs dividing. Either way it's a
  deliberate act, not a one-line assignment.
- **After any edit anywhere near anchors,
  `cat` the file.** Chapter 4's diff habit is
  the whole defence here; this is the chapter
  that justifies it most.
