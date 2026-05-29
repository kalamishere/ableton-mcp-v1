# Fork Notes — ableton-mcp-v1

`ableton-mcp-v1` is a **V1-focused fork** of the open-source **Ableton MCP** project.
It exists to give the V1 Ableton companion app a known-good, reliability-oriented MCP
build, and to stage clean improvements that can be upstreamed.

## Upstream attribution

Ableton MCP was **originally created by Siddharth Ahuja** —
[ahujasid/ableton-mcp](https://github.com/ahujasid/ableton-mcp) (MIT). That's where it
started. This fork is built on an expanded build of it:

- **Original project:** [ahujasid/ableton-mcp](https://github.com/ahujasid/ableton-mcp) — Siddharth Ahuja (the root, ~2.6k★)
- **Built directly on:** [jpoindexter/ableton-mcp](https://github.com/jpoindexter/ableton-mcp) — Jason Poindexter's 2.0.0 / 200+-tool expansion, which derives from the original
- **Forked from commit:** `fa4f9ec40dcf536f7d15794ed82f09e53fbda430` (jpoindexter)
- **License:** MIT — preserved verbatim, see [`LICENSE`](LICENSE) (© Siddharth Ahuja)

This fork is **additive**. No upstream tool is removed or renamed, so any client
written against upstream Ableton MCP keeps working unchanged. The intent is not to
hide the origin — it is to maintain a stable MCP build for V1 and upstream clean
improvements where possible.

## Why this fork exists

The raw upstream tool surface is low-level. Several V1 bugs were really MCP-spec gaps:

- Models confuse **bars vs beats** when creating clips, so clips play too short.
- Drum patterns can be **silent** because callers guess MIDI pitches instead of
  reading a kit's actual drum-pad mapping.
- V1 needs **many round-trips** to assemble session state.
- Tool results are **human-readable strings**, so a caller has to parse prose to know
  whether a call succeeded.
- The Live Object Model exposes **no native export**, so the MCP should report that
  honestly rather than fake it.

This fork moves those reliability concerns into the MCP.

## Fork-only changes

### Drum-pad mapping — Priority 2
- `get_drum_rack_pads(track_index, device_index)` — every drum-rack pad with its real
  MIDI note and name, so patterns land on actual pads instead of an assumed C1/36
  layout.

### Atomic clip writing — Priority 1
- `ensure_midi_clip(track_index, clip_index, length_beats, mode)` — create a MIDI clip,
  or extend an existing one to a target length. Length is in **beats**. Does not error
  when a clip is already present.
- `replace_clip_notes(track_index, clip_index, notes)` — clear and rewrite a clip's
  notes in a single atomic `set_notes` call.
- `upsert_midi_clip(track_index, clip_index, length_beats, notes, name?)` — ensure +
  replace + optional name, in one socket round-trip.
- Note payloads are validated MCP-side: pitch clamped to 0-127, timing/velocity
  coerced, unsalvageable notes dropped and reported in `warnings`. One malformed note
  can never wipe a clip.

### Audio clip import
- `create_audio_clip(track_index, clip_index, path)` — import a local audio file
  (e.g. a rendered WAV) into an empty slot on an audio track. Enables a file-based
  audio handoff into Ableton without streaming samples through MCP. The target must
  be an audio track and the slot empty. **Validated live on Ableton Live 12 Intro.**
  Returns the structured envelope.

### Session snapshot — Priority 3
- `get_session_snapshot(include_tracks, include_clips, include_devices, include_routing)`
  — tempo, transport, and every track's name/type/state/devices/clip summaries in
  **one** call instead of N per-track round-trips.

### Structured result schema — Priority 4
- New tools, and the most-used clip/note/session tools, return a consistent
  `{ ok, data, warnings, error_code, message }` JSON object.
- The other ~117 upstream tools keep their original string returns; clients should
  detect the shape per-tool.

### Honest export capability — Priority 5
- `get_export_capabilities()` — honestly reports that the LOM exposes no native
  render/export, and what *is* available (freeze, arrangement record).
- `prepare_session_for_export(duration_beats?)`,
  `record_session_to_arrangement(duration_beats?)` — help a host app guide a user
  through capture/export without faking a native render.
- No GUI automation lives in the MCP.

## Not changed (deliberately)

- **stdio transport only** — no HTTP/SSE yet (upstream behaviour).
- The ~117 non-critical upstream tools keep their original return strings.
- No app-side GUI export automation — that stays in the host app.

## Upstreaming

Changes here are written to be upstreamable. The drum-pad, atomic-clip,
session-snapshot and audio-import tools are general-purpose and not V1-specific.
`create_audio_clip` has already been proposed upstream as PRs to both
[ahujasid/ableton-mcp](https://github.com/ahujasid/ableton-mcp/pull/101) (the original)
and [jpoindexter/ableton-mcp](https://github.com/jpoindexter/ableton-mcp/pull/1).
