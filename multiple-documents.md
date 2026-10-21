# Multiple Documents

A YAML file can hold several documents, separated
by lines containing only `---`. Kubernetes
manifests are the famous case, but any tool that
bundles related objects does it. The lab's
`multi.yaml` holds two: a `Settings` document and
a `Service` document for a clubhouse kiosk.

Reading was chapter 2's preview: an expression
runs against *every* document, and multi-document
results come back separated by `---`. This
chapter is about aiming at one document in
particular — and it contains this guide's most
destructive surprise, placed here on purpose and
labelled, because meeting it in a lab is much
better than meeting it in production.

## Addressing documents

`document_index` is a built-in that reports
which document an expression is looking at:

```
$ yq 'document_index' multi.yaml
0
1
```

(Plain scalar results print one after another
without the `---` separator; document-shaped
results get it.) With it, `select` picks a
document the way chapter 3 picked list entries:

```
$ yq 'select(document_index == 1)' multi.yaml
# Document 2: the kiosk service itself.
kind: Service
metadata:
  name: kiosk
spec:
  port: 9000
  replicas: 2
```

Reading this way is perfectly safe. Print the
whole file with `yq '.' multi.yaml` and both
documents come back, comments and all.

## The trap

Now the natural next step — change the replica
count in that second document, in place, using
exactly the expression that just read it:

```
$ cp multi.yaml trap.yaml
$ yq -i 'select(document_index == 1) | .spec.replicas = 3' trap.yaml
$ cat trap.yaml
# Document 2: the kiosk service itself.
kind: Service
metadata:
  name: kiosk
spec:
  port: 9000
  replicas: 3
```

Read that `cat` again. **Document 1 is gone.**
Not damaged — *absent*, comments and all. The
`-i` rewrite wrote back what the expression
*output*: the selected document, edited. The
unselected document was never part of the
expression's result, so it was never part of
the file yq wrote. Exit code 0, no warning. If
`trap.yaml` had been a twelve-object manifest,
eleven objects would have vanished this quietly.

This is why chapter 4's discipline exists. The
`cp` before the edit is the only reason the
original survives anywhere, and the `cat`/`diff`
after it is the only reason anyone noticed.

## The safe pattern

The fix is to make the assignment's *path* do
the selecting, so the expression still returns
every document, with the change applied where
the path matches. Parenthesise the path:

```
$ cp multi.yaml fixed.yaml
$ yq -i '(. | select(document_index == 1) | .spec.replicas) = 3' fixed.yaml
$ diff multi.yaml fixed.yaml
15c15
<   replicas: 2
---
>   replicas: 3
```

One line changed. Both documents present, the
`Settings` document byte-identical, and the
diff — the proof, as always — shows exactly the
intended edit and nothing else. The shape to
memorise is: **select inside the path of an
assignment, never as a filter in front of one,
when `-i` is involved.** Filtering is for
reading; paths are for writing.

## Splitting a multi-document file

The deliberate way to take documents apart is
`--split-exp`, which evaluates an expression per
document and names a file after the result:

```
$ mkdir splitout
$ cd splitout
$ yq --split-exp '.metadata.name' '.' ../multi.yaml
$ ls
kiosk-settings.yml
kiosk.yml

$ cat kiosk.yml
---
# Document 2: the kiosk service itself.
kind: Service
metadata:
  name: kiosk
spec:
  port: 9000
  replicas: 2
```

Each document lands in its own file (`.yml`
extension — that's yq's choice, not a setting
this guide changed), comments carried along,
with a leading `---` marking it as a document
that once had company. Splitting is the honest
version of what the trap did by accident: all
documents accounted for, each in a named place,
and the original untouched because no `-i` was
anywhere near it.
