# Markupdown

A native macOS app for reviewing markdown that Claude Code writes: it opens the document, you
select text and type your comment, and the feedback goes straight back to Claude Code.

It is not a markdown editor. You do not edit the prose — you comment on it. The document stays
the agent's to write; the feedback stays yours.

**[Download the latest release](https://github.com/plc/markupdown-releases/releases/latest)**

This repository holds the downloads. The source is private.

## Install

1. Download the zip and unzip it.
2. Drag `Markupdown.app` to `/Applications`.
3. Open it. The app is signed with a Developer ID and notarised by Apple, so it opens normally
   — no right-click, no `xattr`.

Optional, for the command line tool:

```sh
mkdir -p ~/.local/bin && cp mdreview ~/.local/bin/
```

Requires macOS 14 or later. Universal: runs on Apple silicon and Intel.

## Using it

Open a `.md` file, or copy markdown and press ⇧⌘N.

Select text and **just start typing** — the first character you type opens a comment and lands
in it. Press **Delete** on a selection to mark it for removal. Escape returns focus to the
document, ready for the next one.

At the bottom of the comment pane, **Export** writes the comments as markdown beside the
document, and **Copy to Clipboard** puts the same text on the clipboard. Each comment carries
the text it was left on and its line number, so it makes sense without the document in front of
you:

```markdown
### 2. line 22
> Token revocation becomes hard.

Answer this before phase 1, not phase 3.
```

## The tight loop

The `mdreview` CLI turns a human review into an ordinary blocking step an agent can call:

```sh
mdreview wait docs/plan.md
```

That opens the document, blocks until you send, and prints your feedback on stdout. Tell Claude
Code to run it and it will sit and wait for you.

```
mdreview open   <file>    open it and return immediately
mdreview wait   <file>    open it, block until you send, print the feedback
mdreview read   <file>    print the most recent feedback
mdreview list             list stored reviews
mdreview status <file>    is there feedback, and how many rounds so far
mdreview clear  <file>    delete the saved review
```

## Where your comments are kept

Reviews live in their own working directory, not scattered beside your documents:

```
~/Documents/Markupdown/Reviews/plan-a3f19c2b/
    review.json    what the app reads back
    review.md      the same review, readable
```

`review.md` is self-contained — every comment with its quoted text, line number, the document
path and the version it was written against — so it still means something after the document
has been rewritten or deleted.

## What changed

See [CHANGELOG.md](CHANGELOG.md).
