# Privacy Policy — Memory Notes

`Memory Notes` is a **store app** — a quick notes list with an optional
per-note **Remember** action that asks the app's own assistant to keep a
note in its session memory. The notes list works without an assistant.

## Data this app stores

- `notes.json` in `.local-state/memory-notes/` — the notes the user typed
  on this device. Storage cap: 16 MiB.

## Data this app requests

- `storage` capability — used only for the on-device jail above.
- `octos.turn.start` host service — invoked **only when the user taps the
  Remember button on a specific note**. Each invocation sends exactly one
  short string to this app's own host-side agent:
  `Remember: <note text>`
  No other note text is sent, and nothing is sent on app open or on any
  background event.

## What this app does NOT do

- It does not request `network.hosts` and does not open any HTTP/HTTPS
  connection.
- It does not collect telemetry, analytics, or device identifiers.
- It does not auto-remember: every `octos.turn.start` call is gated by an
  explicit per-note button tap.
- It does not collect passwords, PINs, codes, or any account field.

## Agent profile

`read-only` — the app never asks the host to write, sign, or publish.

## Honest degradation

If the host has no assistant, the app falls back to the local note
without losing the user's data; the status line states this explicitly.

## Contact

https://github.com/Thneoly/memory-notes/issues

## Source

https://github.com/Thneoly/memory-notes