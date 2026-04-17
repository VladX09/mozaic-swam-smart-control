---
name: mozaic
description: >
  Use this skill for any task involving Mozaic Script — the MIDI scripting language for the Mozaic iOS AUv3 plugin.
  Activate whenever the user wants to write, edit, debug, or understand Mozaic scripts, asks about Mozaic functions
  or events, needs help with MIDI processing logic in Mozaic, wants to create a Mozaic preset or plugin,
  or shares code that uses @OnLoad, @OnMidiInput, SendMIDINoteOn, or any other Mozaic syntax.
  Mozaic Script is niche and not well-known to LLMs — always consult the reference when working with it.
---

# Mozaic Skill

Mozaic is an event-driven MIDI scripting language for the Mozaic iOS AUv3 plugin. It is Pascal-inspired, not case-sensitive, and designed specifically for MIDI manipulation.

**Always read the reference before writing or debugging Mozaic code.** The language is niche and LLMs will hallucinate syntax without it.

## Reference

The full language reference is in `references/mozaic_programming_guide.md`. Read it (or the relevant sections) before writing any Mozaic code.

Key sections:
- **Section 8 — Events**: All available event handlers (`@OnLoad`, `@OnMidiInput`, `@OnNewBeat`, etc.)
- **Section 9 — Commands and Functions**: Complete function reference with signatures and examples (MIDI, Sysex, GUI, Math, LFO, Scale, NoteState)
- **Section 10 — Cheat Sheet**: Quick function lookup by category

## Core Language Rules

- One instruction per line
- Not case-sensitive (`sendmidinoteOn` = `SendMIDINoteOn`)
- Comments: `// this is a comment`
- No type declarations — variables are auto-typed
- String literals use curly braces: `Log {Hello!}`
- Variables have instance-wide scope (no local scope)
- Initialize variables in `@OnLoad`

## Script Structure

```mozaic
@OnLoad
  // initialization
@End

@OnMidiInput
  // handle any MIDI
@End

@OnMidiNoteOn
  // handle note on specifically
@End
```

## Key Built-in Variables (in MIDI event handlers)

- `MIDIChannel` — channel (0-based)
- `MIDINote` — note number 0-127
- `MIDIVelocity` — velocity 0-127
- `MIDIByte1`, `MIDIByte2`, `MIDIByte3` — raw MIDI bytes
- `MIDICommand` — MIDI command type (hex, e.g. `0x90` = Note On)
- `HostTempo`, `HostBeat`, `HostBar` — host transport info

## Workflow

1. Read the relevant sections of the reference
2. Write the script using correct syntax from the reference
3. Check function signatures carefully — parameter order matters
4. Use `Log` for debugging: `Log {value: }, myVar`
