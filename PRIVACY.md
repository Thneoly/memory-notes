# Privacy Policy — Memory Notes

`Memory Notes` is the **writer** in a Notes + Digest pair. Every note you
type is sent straight to your device's assistant through the
`octos.turn.start` host service, and the list shown back to you is read
from the same assistant's `octos.session.history`. The device itself
keeps no copy of the notes.

## Data this app stores

None. No file is written under `.local-state/memory-notes/` for the
notes themselves; the app does not call the `storage` capability.

## Data this app requests

- `octos.turn.start` host service — invoked when you tap **Remember**.
  Each invocation sends exactly one short string to your device's
  assistant:
  ```
  Remember: <note text>
  ```
- `octos.session.history` host service — invoked when the screen opens
  and after every Remember / Forget. The response is filtered locally
  for messages whose role is `user` and whose text starts with
  `Remember: `. The remaining text becomes one row in the list. Nothing
  else from the conversation is read, rendered, stored, or forwarded.

## What this app does NOT do

- It does not request `network.hosts` and does not open any HTTP/HTTPS
  connection.
- It does not request the `storage` capability and does not write any
  local file.
- It does not collect telemetry, analytics, or device identifiers.
- It does not auto-remember: every `octos.turn.start` call is gated by
  an explicit button tap.
- It does not collect passwords, PINs, codes, or any account field.

## Agent profile

`read-only` — the app never asks the host to write, sign, or publish.

## On devices without an assistant

If the host has no assistant service (today: every OctoSense shell,
every `card-host`), the app shows the message **Assistant unavailable**
and does nothing else. There is no local cache to fall back to.

## Contact

https://github.com/Thneoly/memory-notes/issues

## Source

https://github.com/Thneoly/memory-notes
