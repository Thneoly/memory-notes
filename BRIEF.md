# Memory Notes (memory-notes) — brief

The writer demo of the shared-memory system: a quick notes app whose every
note can be sent to the app's own agent with one tap ("Remember"), showing
how any store app writes to the memory layer. Base works without an AI;
the Remember action degrades honestly.

## Screens

1. **Main** — title "Memory Notes"; text entry + Add; the note list
   (newest first); under the list a status line for the last Remember
   result; hint line.

## Actions

- **Add / remove notes** — local, in `notes.json` (template behavior).
- **Remember** — a small button on each note row: sends
  `host.request("octos.turn.start", {text: "Remember: <note>"})` to this
  app's own agent. On success the status line reads "Remembered ✅
  \<text\>"; when the host has no assistant (card-host) it shows the
  honest one-liner (`No assistant on this device — kept locally: \<text\>`)
  and the local note list is unaffected.

## Data

- `notes.json` in the jail: array of strings (template format).

## States

- Empty list, populated list, restart persistence (notes.json).
- Remember with no assistant: status line reads `No assistant on this
  device — kept locally: \<the note that was tapped\>`. The local
  note itself is unaffected; only the assistant hop is skipped.

## Hosts

None.

## Capabilities and why

- `storage` — `notes.json` in the jail.
- `octos.turn.start` — the Remember action, this app's own agent only.
- `agent` block (`profile: read-only`) — declares the agent so the system
  agent can later gather what was remembered (the sync side of the demo).
  (`tools` omitted: this hub build offers no tools to contained apps.)

## Not in this app

- Reading any other app's data (impossible by isolation; that is the
  point the demo makes).
