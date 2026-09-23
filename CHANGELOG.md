# Changelog

All notable changes to Markupdown. Newest first. Downloads are on the
[releases page](https://github.com/plc/markupdown-releases/releases).

## 0.5.0 — 2026-09-23

### Added

- **Update checking.** On launch, at most once a day, the app asks GitHub whether there is a
  newer release and shows a bar across the top of the window if there is, with the release
  notes, a download button, and Skip for a version you do not want. **Check for Updates…** in
  the Markupdown menu forces a check and reports the result either way; it also overrides a
  previous Skip. Turn the automatic check off and nothing is sent anywhere.

  Deliberately a notification rather than a self-installing updater: downloading a replacement
  bundle, verifying it and swapping it out underneath a running app is the fiddly,
  security-sensitive part that Sparkle exists to get right, and hand-rolling that badly is
  worse than a banner.

## 0.4.1 — 2026-09-23

### Fixed

- **Send App Feedback filed issues against a repository nobody could see.** The source repo is
  private, and issues on a private repo return 404 to anyone without access — which is everyone
  who downloaded the app. Feedback now goes to the public downloads repository.
- The install notes bundled with the download still described the File/Clipboard toggle removed
  in 0.4.0.

## 0.4.0 — 2026-09-23

First published build: signed with a Developer ID and notarised, so it opens on any Mac
without the right-click dance. Reviews moved somewhere sensible and the comment pane lost the
parts that were asking you to choose between two things that did the same job.

### Added

- **A toolbar** carrying Open, New, From Clipboard and Reload. Built in AppKit, because
  SwiftUI renders no toolbar for these windows under any toolbar style. From Clipboard greys
  out unless the clipboard holds something worth opening.
- `mdreview list` shows every stored review; `mdreview status` reports where a review is kept.

### Changed

- **Reviews live in a working directory**, not in sidecar files beside the document:
  `~/Documents/Markupdown/Reviews/<name>-<hash>/` holds `review.json` and `review.md`. The
  readable copy carries every comment with its quoted text and line number, so a review still
  means something once the document has been rewritten or deleted. Filed by path, because a
  document under review gets rewritten repeatedly and you want your notes back when you reopen
  it; which version each comment was written against is recorded per comment. Sidecars written
  by earlier builds are adopted on first open and left in place.
- **The File/Clipboard toggle is gone.** Both delivered the same content to different places,
  so the comment pane now simply offers **Export** and **Copy to Clipboard**. The
  annotated-document export moved to the Review menu.
- **The comment pane is hidden until a document is open**, rather than accepting feedback with
  nothing to attach it to.
- **New from Clipboard checks what it has been given** and says why when it declines — a copied
  link, a JSON blob or a stray word is not a document. Plain prose still counts.
- Removed the App Feedback and Show File links from the comment pane. App feedback remains in
  the Help menu.

## 0.3.0 — 2026-09-23

Multi-window, and the two bugs that only showed up on a real document.

### Added

- **Delete marks a selection for removal.** Pressing Delete on selected text opens a comment
  seeded with "Remove" rather than beeping. Pressing it twice on one span does not repeat the
  word, and it never overwrites a note already written there.
- **One window per document.** New, New from Clipboard, Open and drag-and-drop each open a
  window instead of replacing what you were reviewing. Opening a document that is already open
  brings its window forward rather than showing it twice. Menu commands act on the frontmost
  window.
- **Send App Feedback** (Help menu) files a GitHub issue with the app and OS versions attached.
  Either a prefilled browser form needing no setup, or direct submission with a personal access
  token kept in the login Keychain. Deliberately no token in the app bundle or the repository —
  either is readable by anyone holding a copy.
- An app icon, generated from the source artwork by `Scripts/make-icon.swift`.
- `Scripts/build-app.sh --universal` builds for Intel as well as Apple silicon.
  `Scripts/release.sh` signs with a Developer ID, notarises and staples; `Scripts/package.sh`
  stages the result with install instructions that match whether the build was notarised.
  `mdreview` moved inside the app bundle so one signed artefact covers both binaries.

### Fixed

- **Comments from a previous version of a document passed themselves off as current.** When a
  document changes underneath open comments, re-anchoring finds the quoted text again and the
  notes reattach looking perfectly ordinary. Each comment now records the version it was
  written against; ones that predate the text on screen are badged "earlier version", the
  comment pane offers to archive them in one go, and the reload message says how many there
  are. An orphaned comment no longer shows a line number from a document that no longer exists.
- **Long documents were clipped.** Switching the reading view to TextKit 1, needed for tables,
  left `maxSize` unset, so the text view stopped growing at its initial frame height and cut
  the document off partway down with no seam to show for it. Comment editors had the same bug
  for long notes.
- **Inline markup spanning a line break rendered its syntax characters.** The scanner worked
  line by line, so `**bold across a\nline break**` never found its closing delimiter — and
  agents hard-wrap prose at 80 columns, so this hit ordinary output.
- The window title read "No document" while a document was open: `.navigationTitle` latched
  onto the first value it saw. Title and proxy icon are set on the window directly now.
- macOS was restoring the previous session's windows and adding blank ones at launch and on
  re-activation. Restoration is off, and unrequested blank windows close themselves.

### Changed

- Removed the SwiftUI window toolbar, which rendered nothing under any toolbar style. (An
  AppKit one returned in 0.4.0.)
- The reading view moved into `MarkupdownUI` as `ReadingTextView.make()`, so its geometry is
  covered by tests rather than only by looking at it.

## 0.2.0 — 2026-09-23

Renamed to **Markupdown**. Faster commenting, a document-shaped output format, real tables.

### Added

- **Type to comment.** With text selected, the first character you type opens a comment and
  lands in it. Escape returns focus to the document for the next one. The note editors moved
  from SwiftUI's `TextEditor` to an NSTextView wrapper to make that work, which also lets them
  grow with their content.
- **Annotated document output.** Writes the whole document back with each comment spliced in as
  a blockquote callout after the block it refers to, rather than a detached list of notes. The
  original text is reproduced byte for byte, so the annotated copy diffs cleanly against the
  original.
- **Real tables.** Pipe tables lay out as bordered columns with per-column alignment via
  `NSTextTable`, replacing the preformatted-text placeholder. Code blocks gained a full-width
  tint from the same text-block machinery.
- **New** and **New from Markdown Clipboard** (Cmd-N, Cmd-Shift-N), for markdown an agent
  printed in the terminal instead of writing to a file. Scratch documents are saved to
  `~/Documents/Markupdown/`, named after their first heading.
- `mdreview render <file> [--format annotated|comments]` re-renders saved comments without
  opening the app.
- Snapshot tests that lay the renderer out offscreen and write a PNG, so the rendering can be
  inspected without launching the app.

### Fixed

- The copied prompt could disagree with the comments visible in the pane: comments already
  sent, and comments with no note typed in them, were excluded silently. The pane now says how
  many will be skipped, and the status line reports what actually went out.
- The annotated header leaked its summary outside the HTML comment, so it showed up as body
  text when the file was rendered.
- A callout after a list item split the list in two, restarting an ordered list's numbering.
  Callouts are indented into the item now.

### Changed

- App, bundle and modules renamed from MarkupMarkdown to Markupdown.
- Rendering moved out of the app target into `MarkupdownUI` so it can be tested offscreen.

## 0.1.0 — 2026-09-23

First working version. Claude Code writes a document, you mark it up, the feedback goes back.

### Added

- Read-only markdown reading view with headings, lists, code blocks, blockquotes, links and
  inline emphasis.
- Anchored comments: select a span, write a note. Spans are highlighted in the document;
  clicking either side jumps to the other.
- Overall note, for feedback that isn't tied to a span.
- Writes `<doc>.review.md` next to the document and files sent comments under a collapsed Sent
  section.
- `mdreview` CLI with `open`, `wait`, `read`, `status`, `clear`. `wait` blocks until you send
  and prints the feedback on stdout, so a human review becomes an ordinary blocking step for an
  agent.
- Live reload: the app polls the document's mtime and re-anchors comments when it changes.
- `Integration/review-doc.md`, a Claude Code slash command that drives the whole loop.

### Decisions

- **Own markdown parser instead of a library.** Comments anchor to source offsets, and every
  available renderer discards source locations. The parser keeps a 1:1 map from every rendered
  character back to the byte it came from. This is the reason the app works at all, and the
  constraint any future parser change has to respect.
- **Files as the handoff, not IPC or MCP.** An MCP server was the obvious "tight integration"
  answer but needs configuring per project and dies with the session. Files are inspectable,
  git-diffable, survive crashes, and work with any agent. The CLI polls a counter in the state
  file; that is the entire protocol.
- **Read-only document.** Tracked-changes editing was on the table. Comments won because the
  point is to tell the agent *what* to change, not to make the change yourself — and because a
  read-only view sidesteps reconciling your edits with the agent's rewrite.
- **Sending marks comments as sent rather than deleting them.** The round trip is iterative;
  you want last round's notes visible while you read this round's draft.
- **mtime polling over `DispatchSource` file watching.** Agents write files by atomic replace,
  which swaps the inode and silently kills an fd-based watcher.

### Known gaps at this release

Since addressed: no app icon (0.3.0), tables rendered as preformatted text (0.2.0), ad-hoc
signing requiring a right-click to open (0.3.0).

Still open: a code block nested inside a list item renders flat.
