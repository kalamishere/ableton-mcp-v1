# Changelog

All notable changes to AbletonMCP will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-05-22 — ableton-mcp-v1 fork

V1-focused reliability fork of Ableton MCP. See [FORK.md](FORK.md) for upstream
attribution (forked from `fa4f9ec40dcf…`). The fork is additive — all upstream
tools are kept intact.

### Added
- **Drum-pad mapping (P2):** `get_drum_rack_pads` — every drum-rack pad with its real
  MIDI note and name, so patterns land on actual pads, not an assumed C1/36 layout.
- **Atomic clip writing (P1):** `ensure_midi_clip`, `replace_clip_notes`,
  `upsert_midi_clip` — clip length is in beats; an existing clip is extended, not
  re-errored; notes are validated MCP-side (pitch clamped 0-127, malformed notes
  dropped) so one bad note can never wipe a clip.
- **Session snapshot (P3):** `get_session_snapshot` — tempo, transport and every
  track's state/devices/clip summaries in one call instead of N round-trips.
- **Honest export (P5):** `get_export_capabilities`, `prepare_session_for_export`,
  `record_session_to_arrangement` — no fake native render; no GUI automation in MCP.

### Changed
- **Structured results (P4):** new tools and ~12 V1-critical clip/note/session tools
  now return a consistent `{ok, data, warnings, error_code, message}` JSON object.
  Other upstream tools keep their original string returns.

## [2.0.0] - 2026-01-27

### Added

#### New Tool Categories (67 new tools)
- **Scene Management** (8 tools): `get_all_scenes`, `create_scene`, `delete_scene`, `fire_scene`, `stop_scene`, `set_scene_name`, `set_scene_color`, `duplicate_scene`
- **Track Operations** (3 tools): `delete_track`, `duplicate_track`, `set_track_color`
- **Device Control** (2 tools): `toggle_device`, `delete_device`
- **Clip Enhancements** (3 tools): `duplicate_clip`, `set_clip_color`, `set_clip_loop`
- **Note Editing** (3 tools): `remove_notes`, `remove_all_notes`, `transpose_notes`
- **Undo/Redo** (2 tools): `undo`, `redo`
- **Return/Send Tracks** (5 tools): `get_return_tracks`, `get_return_track_info`, `set_send_level`, `set_return_volume`, `set_return_pan`
- **View Control** (5 tools): `get_current_view`, `focus_view`, `select_track`, `select_scene`, `select_clip`
- **Recording** (6 tools): `start_recording`, `stop_recording`, `toggle_session_record`, `toggle_arrangement_record`, `set_overdub`, `capture_midi`
- **Arrangement View** (6 tools): `get_arrangement_length`, `set_arrangement_loop`, `jump_to_time`, `get_locators`, `create_locator`, `delete_locator`
- **I/O Routing** (6 tools): `get_track_input_routing`, `get_track_output_routing`, `get_available_inputs`, `get_available_outputs`, `set_track_input_routing`, `set_track_output_routing`
- **Performance & Session** (5 tools): `get_cpu_load`, `get_session_path`, `is_session_modified`, `get_metronome_state`, `set_metronome`
- **AI Music Helpers** (6 tools): `get_scale_notes`, `quantize_clip_notes`, `humanize_clip_timing`, `humanize_clip_velocity`, `generate_drum_pattern`, `generate_bassline`

#### Multi-Provider AI Support
- **REST API Server**: New FastAPI-based server for non-MCP clients
- **Ollama Support**: Free, local AI via REST API
- **OpenAI Support**: GPT-4 and other models via REST API
- **Claude API Support**: Direct API access (not MCP) via REST API
- **Groq Support**: Fast inference via REST API

#### Max for Live Device
- **AbletonMCP.amxd**: Native M4L device with multi-provider support
- **Visual UI**: Provider selection, model input, chat interface
- **Direct Integration**: No external server required

#### AI Music Generation
- **Drum Patterns**: 8 styles (basic, house, techno, hip_hop, trap, dnb, jazz, funk)
- **Basslines**: 6 styles (basic, walking, synth, funk, octave, arpeggiated)
- **Scale Reference**: 13 scales (major, minor, modes, pentatonic, blues)
- **Humanization**: Timing and velocity variation
- **Quantization**: Snap notes to any grid size

#### Examples
- `ollama_example.py`: Interactive CLI chat with Ollama

### Changed
- Improved error handling with retry logic
- Better socket connection management
- Extended timeouts for complex operations
- Updated documentation with comprehensive tool reference

### Fixed
- Connection stability improvements
- Better handling of large responses
- Thread-safe command execution

## [1.0.0] - Original Release

### Features
- Basic Ableton Live integration via MCP
- Track creation (MIDI and audio)
- Clip creation and note editing
- Device parameter control
- Browser navigation
- Transport control
- Tempo setting

---

## Migration Guide

### From 1.0.0 to 2.0.0

No breaking changes. All existing tools work as before. New tools are additions.

To use the new REST API for Ollama:
```bash
pip install ableton-mcp[rest]
python MCP_Server/rest_api_server.py
```

To use the Max for Live device:
1. Copy `AbletonMCP_M4L` folder to your M4L presets
2. Rename `.maxpat` to `.amxd`
3. Load in Ableton
