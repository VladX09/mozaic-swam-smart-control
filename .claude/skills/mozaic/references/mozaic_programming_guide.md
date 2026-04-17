# Mozaic Programming Guide — Reference

**Version 1.3** | By Bram Bos

Mozaic is an event-based MIDI scripting language for iOS AUv3 plugins. Scripts respond to MIDI and UI events, process and transform MIDI data, and control the plugin GUI.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Getting Started](#2-getting-started)
3. [The Basics](#3-the-basics)
4. [Variables and Arrays](#4-variables-and-arrays)
5. [Operators](#5-operators)
6. [Control Flow](#6-control-flow)
7. [GUI Layouts](#7-gui-layouts)
8. [Events](#8-events)
9. [Commands and Functions](#9-commands-and-functions)
   - [MIDI Functions](#midi-functions)
   - [Sysex Functions](#sysex-functions)
   - [AUv3 and Host Functions](#auv3-and-host-functions)
   - [Timer and LFO Functions](#timer-and-lfo-functions)
   - [Musical Scale Functions](#musical-scale-functions)
   - [GUI and Interaction Functions](#gui-and-interaction-functions)
   - [Variables and Math Functions](#variables-and-math-functions)
   - [NoteState Matrix Functions](#notestate-matrix-functions)
10. [Function Overview by Category (Cheat Sheet)](#10-function-overview-by-category-cheat-sheet)

---

## 1. Introduction

Mozaic is a scriptable MIDI processor plugin for iOS. Scripts are written in Mozaic Script — a simple, readable, Pascal-inspired language. Scripts react to events (MIDI input, knob changes, pad taps, timer pulses, etc.) and can output MIDI, update the GUI, perform math, and maintain state.

Key characteristics:
- Not case-sensitive
- One instruction per line
- Comments start with `//`
- Event-driven architecture
- Variables are instance-wide (no local scope)
- No type declarations — variables are auto-typed

---

## 2. Getting Started

### Hello World

```mozaic
@OnLoad
  Log {Hello World!}
@End
```

### Sending MIDI Through

```mozaic
@OnMidiInput
  SendMIDIThru
@End
```

### Transposing Notes Up by 1 Semitone

```mozaic
@OnMidiNoteOn
  SendMIDINoteOn MIDIChannel, MIDINote + 1, MIDIVelocity
@End

@OnMidiNoteOff
  SendMIDINoteOff MIDIChannel, MIDINote + 1, MIDIVelocity
@End
```

---

## 3. The Basics

### Script Structure

A Mozaic script consists of **event handlers** (and optionally user-defined events/subroutines). Each handler begins with `@EventName` and ends with `@End`.

```mozaic
@OnLoad
  // runs once when script loads
  ShowLayout 0
@End

@OnMidiInput
  // runs on every MIDI message received
  SendMIDIThru
@End
```

### Comments

```mozaic
// This is a comment
x = 5  // inline comment
```

### String Literals

Strings are enclosed in curly braces `{}` and used only inside `Log` and `Label` commands.

```mozaic
Log {Hello}, myVar
LabelKnob 0, {Volume}, 0, 127
```

### Boolean Values

| True | False |
|------|-------|
| `YES` | `NO` |
| `true` | `false` |
| `1` | `0` |

### MIDI Channels

MIDI channels are numbered **0–15** internally (not 1–16).

### MIDI Commands (Hex)

| Constant | Value | Meaning |
|----------|-------|---------|
| `0x90` | 144 | Note On |
| `0x80` | 128 | Note Off |
| `0xB0` | 176 | Control Change (CC) |
| `0xC0` | 192 | Program Change |
| `0xD0` | 208 | Channel Aftertouch |
| `0xE0` | 224 | Pitch Bend |

---

## 4. Variables and Arrays

### Variables

Variables do not need to be declared. They are created on first assignment. Scope is instance-wide (accessible in all event handlers).

```mozaic
myVar = 42
result = myVar * 2
```

- Optimal accuracy up to **24 bits**
- Numbers exceeding 24-bit precision are treated as **32-bit float**

### GLOBAL Variables

`GLOBAL0` through `GLOBAL99` are **meta-variables** shared across all Mozaic instances in a session. Useful for inter-plugin communication.

```mozaic
GLOBAL0 = 127
x = GLOBAL5
```

### Arrays

Arrays hold up to **1024 cells**, indexed from 0.

```mozaic
myArray[0] = 10
myArray[1] = 20
val = myArray[0]
```

**Bulk initialization:**

```mozaic
myArray = [10, 20, 30, 40, 50]
```

**Copy and fill:**

```mozaic
CopyArray sourceArray, destArray
FillArray myArray, 0
```

### Safe Initialization with Unassigned

Use `Unassigned` to check if a variable exists (e.g., to avoid resetting state on preset reload):

```mozaic
@OnLoad
  if Unassigned myVar
    myVar = 0
  endif
@End
```

---

## 5. Operators

### Arithmetic Operators

| Operator | Description |
|----------|-------------|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division (floating point) |
| `%` | Modulo (remainder) |

### Bitwise Integer Operators

| Operator | Description |
|----------|-------------|
| `&` | Integer AND |
| `\|` | Integer OR |
| `^` | Integer XOR |
| `-` (unary) | Negate |

### Comparison Operators

| Operator | Description |
|----------|-------------|
| `=` | Equal |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal |
| `>=` | Greater than or equal |
| `<>` | Not equal |

### Logical Operators

| Operator | Description |
|----------|-------------|
| `AND` | Logical AND |
| `OR` | Logical OR |
| `NOT` | Logical NOT |

### Examples

```mozaic
x = 10 % 3       // x = 1
y = 5 & 3        // y = 1 (bitwise AND)
z = NOT YES      // z = NO (false)
if x > 0 AND y < 10
  Log {both true}
endif
```

---

## 6. Control Flow

### If / Elseif / Else / Endif

```mozaic
if x = 0
  Log {zero}
elseif x < 0
  Log {negative}
else
  Log {positive}
endif
```

### Repeat / Until

```mozaic
i = 0
repeat
  i = i + 1
until i >= 10
```

### While / Endwhile

```mozaic
while x > 0
  x = x - 1
endwhile
```

### For Loop

```mozaic
for i = 0 to 15
  Log {pad}, i
endfor
```

### User Events (Subroutines)

Define reusable code blocks with custom `@EventName` and call them with `Call`:

```mozaic
@MySubroutine
  Log {Running subroutine}
  x = x + 1
@End

@OnLoad
  x = 0
  Call @MySubroutine
  Call @MySubroutine
@End
```

### Exit

`Exit` immediately exits the current event handler.

```mozaic
@OnMidiInput
  if MIDICommand = 0xB0
    Exit  // ignore CC messages
  endif
  SendMIDIThru
@End
```

---

## 7. GUI Layouts

### Layout Options

| Layout | Number | Contents |
|--------|--------|----------|
| Mix | 0 | 4 pads, 10 knobs, XY pad |
| Knobs | 1 | 22 knobs |
| Pads | 2 | 16 pads, 4 knobs |
| Sliders | 3 | 10 sliders, XY pad |
| Minimal | 4 | 4 knobs, description text box |

Set layout in `@OnLoad`:

```mozaic
@OnLoad
  ShowLayout 2  // Pads layout
@End
```

### GUI Elements

| Element | Count | Range |
|---------|-------|-------|
| Knobs | 22 | 0–21 |
| Pads | 16 | 0–15 |
| XY Pad | 1 | 0–127 per axis |
| Shift Button | 1 | — |

### Pad Colors

| Color | Number |
|-------|--------|
| White (default) | 0 |
| Red | 1 |
| Yellow | 2 |
| Green | 3 |
| Light blue | 4 |
| Lavender/purple | 5 |
| Violet | 6 |
| Magenta | 7 |

### Plugin Description

Use `@Description` (not an event handler) to provide a text description shown in the Minimal layout:

```mozaic
@Description
  This script transposes incoming MIDI notes by a set interval.
  Use Knob 0 to set the transposition amount.
@End
```

---

## 8. Events

Event handlers are defined with `@EventName` ... `@End`. They execute when the corresponding event occurs.

### Event Reference

| Event | Trigger |
|-------|---------|
| `@OnLoad` | Script loaded, preset loaded, or host restores state |
| `@OnMidiInput` | Any MIDI input received (called first, before specialized handlers) |
| `@OnMidiNote` | Note On or Note Off received |
| `@OnMidiNoteOn` | Note On received |
| `@OnMidiNoteOff` | Note Off received |
| `@OnMidiCC` | MIDI CC received |
| `@OnNewBeat` | Host playback reaches a new beat |
| `@OnNewBar` | Host playback reaches a new bar |
| `@OnMetroPulse` | Internal metronome pulse (rate set by SetMetroPPQN) |
| `@OnKnobChange` | A knob value changed (check `LastKnob`) |
| `@OnPadDown` | A pad was tapped (check `LastPad`) |
| `@OnPadUp` | A pad was released (check `LastPad`) |
| `@OnTimer` | Timer interval elapsed |
| `@OnXYChange` | XY pad value changed |
| `@OnSysex` | Sysex message received (up to 1024 bytes) |
| `@OnShiftDown` | Shift button pressed |
| `@OnShiftUp` | Shift button released |
| `@OnHostStart` | Host transport starts playback |
| `@OnHostStop` | Host transport stops playback |
| `@OnAUParameter` | An AU parameter was changed by the host |
| `@OnPedalDown` | Sustain pedal pressed |
| `@OnPedalUp` | Sustain pedal released |

### Event Usage Examples

```mozaic
@OnLoad
  ShowLayout 0
  PresetScale {Major}
  SetRootNote 0   // C
@End

@OnMidiNoteOn
  if InScale MIDINote
    SendMIDINoteOn MIDIChannel, MIDINote, MIDIVelocity
  endif
@End

@OnKnobChange
  Log {Knob changed:}, LastKnob, {value:}, GetKnobValue LastKnob
@End

@OnPadDown
  ColorPad LastPad, 3   // green
  FlashPad LastPad
@End

@OnTimer
  // periodic tasks
@End

@OnNewBeat
  FlashUserLed
@End
```

---

## 9. Commands and Functions

---

### MIDI Functions

---

#### SendMIDIOut

```mozaic
SendMIDIOut <byte1>, <byte2>, <byte3> [,<delay>]
```

Send a raw 3-byte MIDI message. Mozaic automatically determines the correct message size (some MIDI messages are 1 or 2 bytes). The optional `<delay>` parameter delays sending by the specified number of milliseconds.

```mozaic
SendMIDIOut 0x90, 60, 100       // Note On, middle C, velocity 100
SendMIDIOut 0x80, 60, 0         // Note Off, middle C
SendMIDIOut 0xC0, 5, 0          // Program Change to patch 5 on ch 0
SendMIDIOut 0x90, 60, 100, 500  // Note On, delayed 500ms
```

---

#### SendMIDINoteOn

```mozaic
SendMIDINoteOn <chan>, <note>, <velocity> [,<delay>]
```

Send a MIDI Note On message.

- `<chan>`: MIDI channel 0–15
- `<note>`: note number 0–127
- `<velocity>`: velocity 0–127
- `<delay>`: optional delay in milliseconds

```mozaic
SendMIDINoteOn 0, 60, 100        // Note On ch 0, middle C, vel 100
SendMIDINoteOn MIDIChannel, MIDINote, MIDIVelocity   // echo input
SendMIDINoteOn 0, 60, 100, 250   // delayed 250ms
```

---

#### SendMIDINoteOff

```mozaic
SendMIDINoteOff <chan>, <note>, <velocity> [,<delay>]
```

Send a MIDI Note Off message.

- `<chan>`: MIDI channel 0–15
- `<note>`: note number 0–127
- `<velocity>`: velocity 0–127
- `<delay>`: optional delay in milliseconds

```mozaic
SendMIDINoteOff 0, 60, 0
SendMIDINoteOff MIDIChannel, MIDINote, 0, 100   // delayed 100ms
```

---

#### SendMIDICC

```mozaic
SendMIDICC <chan>, <controller>, <value> [,<delay>]
```

Send a MIDI Control Change (CC) message.

- `<chan>`: MIDI channel 0–15
- `<controller>`: CC number 0–127
- `<value>`: CC value 0–127
- `<delay>`: optional delay in milliseconds

```mozaic
SendMIDICC 0, 7, 100           // CC 7 (volume) = 100 on ch 0
SendMIDICC MIDIChannel, 64, 127  // sustain on
```

---

#### SendMIDIPitchbend

```mozaic
SendMIDIPitchbend <chan>, <value> [,<delay>]
```

Send a MIDI Pitch Bend message.

- `<chan>`: MIDI channel 0–15
- `<value>`: 14-bit pitch bend value, **0–16383**, center = **8192**
- `<delay>`: optional delay in milliseconds

```mozaic
SendMIDIPitchbend 0, 8192    // center (no bend)
SendMIDIPitchbend 0, 16383   // full up
SendMIDIPitchbend 0, 0       // full down
```

---

#### SendMIDIProgramChange

```mozaic
SendMIDIProgramChange <chan>, <patch> [,<delay>]
```

Send a MIDI Program Change message.

- `<chan>`: MIDI channel 0–15
- `<patch>`: program number 0–127
- `<delay>`: optional delay in milliseconds

```mozaic
SendMIDIProgramChange 0, 5   // switch to patch 5 on ch 0
```

---

#### SendMIDIBankSelect

```mozaic
SendMIDIBankSelect <chan>, <MSB>, <LSB> [,<delay>]
```

Send a MIDI Bank Select message (CC 0 + CC 32 + Program Change).

- `<chan>`: MIDI channel 0–15
- `<MSB>`: bank MSB (CC 0 value) 0–127
- `<LSB>`: bank LSB (CC 32 value) 0–127
- `<delay>`: optional delay in milliseconds

```mozaic
SendMIDIBankSelect 0, 0, 1   // bank 0:1 on channel 0
```

---

#### SendMIDIThru

```mozaic
SendMIDIThru [<delay>]
```

Forward the current incoming MIDI message to the output unchanged. Optional delay in milliseconds.

```mozaic
@OnMidiInput
  SendMIDIThru
@End
```

---

#### SendMIDIThruOnCh

```mozaic
SendMIDIThruOnCh <chan> [,<delay>]
```

Forward the current incoming MIDI message to the output, but change its channel to `<chan>` (0–15).

```mozaic
@OnMidiInput
  SendMIDIThruOnCh 2    // reroute everything to channel 2
@End
```

---

#### ConfigureMPE

```mozaic
ConfigureMPE <lowerzone>, <upperzone>
```

Send an MPE Configuration Message to configure MPE zones.

- `<lowerzone>`: number of channels for the lower zone (0 = disabled)
- `<upperzone>`: number of channels for the upper zone (0 = disabled)

```mozaic
ConfigureMPE 15, 0    // lower zone with 15 channels, no upper zone
ConfigureMPE 0, 0     // disable MPE
```

---

#### MIDIChannel

```mozaic
<result> = MIDIChannel
```

Returns the MIDI channel of the currently processed incoming MIDI event (0–15). Use inside any MIDI event handler.

```mozaic
@OnMidiInput
  ch = MIDIChannel
  Log {Channel:}, ch
@End
```

---

#### MIDICommand

```mozaic
<result> = MIDICommand
```

Returns the MIDI command byte of the current event, with the channel nibble stripped. E.g. 0x90, 0x80, 0xB0, etc.

```mozaic
@OnMidiInput
  if MIDICommand = 0x90
    Log {Note On received}
  endif
@End
```

---

#### MIDINote

```mozaic
<result> = MIDINote
```

Returns the MIDI note number (0–127) of the current note event. Same as `MIDIByte2` for note events. Use in `@OnMidiNote`, `@OnMidiNoteOn`, `@OnMidiNoteOff`.

```mozaic
@OnMidiNoteOn
  note = MIDINote
@End
```

---

#### MIDIVelocity

```mozaic
<result> = MIDIVelocity
```

Returns the note velocity (0–127) of the current note event. Use in `@OnMidiNote`, `@OnMidiNoteOn`, `@OnMidiNoteOff`.

```mozaic
@OnMidiNoteOn
  vel = MIDIVelocity
@End
```

---

#### MIDIByte1 / MIDIByte2 / MIDIByte3

```mozaic
<result> = MIDIByte1
<result> = MIDIByte2
<result> = MIDIByte3
```

Return raw MIDI data bytes (0–255) of the current MIDI event.

- `MIDIByte1`: status byte (command + channel)
- `MIDIByte2`: first data byte (e.g., note number, CC number)
- `MIDIByte3`: second data byte (e.g., velocity, CC value)

```mozaic
@OnMidiInput
  Log {Bytes:}, MIDIByte1, MIDIByte2, MIDIByte3
@End
```

---

#### MIDISustainPedalDown

```mozaic
<result> = MIDISustainPedalDown
```

Returns `YES` if any sustain pedal (CC 64 >= 64) is currently held down, `NO` otherwise.

```mozaic
if MIDISustainPedalDown
  Log {Pedal is down}
endif
```

---

### Sysex Functions

---

#### SendSysex

```mozaic
SendSysex <array>, <length> [,<checksum>, <startindex>]
```

Send a SysEx message from an array variable.

- `<array>`: array variable containing the SysEx data (do NOT include the 0xF0 start or 0xF7 end bytes — Mozaic adds them automatically)
- `<length>`: number of bytes to send
- `<checksum>` (optional): checksum algorithm: `0` = none, `1` = Roland/Boss, `2` = Fractal Audio (AxeFX)
- `<startindex>` (optional): start index within the array for checksum calculation

```mozaic
sysexData[0] = 0x41    // Roland manufacturer ID
sysexData[1] = 0x10    // device ID
sysexData[2] = 0x42    // model ID
sysexData[3] = 0x12    // command: DT1 (data transfer)
sysexData[4] = 0x40    // address MSB
sysexData[5] = 0x00
sysexData[6] = 0x7F
sysexData[7] = 0x00    // data byte
SendSysex sysexData, 8, 1, 4   // Roland checksum, starting at index 4
```

---

#### SendSysexThru

```mozaic
SendSysexThru
```

Forward the last received SysEx message to the output, unchanged.

```mozaic
@OnSysex
  SendSysexThru
@End
```

---

#### ReceiveSysex

```mozaic
ReceiveSysex <array>
```

Copy the incoming SysEx data into the specified array variable. Use in `@OnSysex`. The 0xF0 and 0xF7 bytes are NOT included.

```mozaic
@OnSysex
  ReceiveSysex myBuffer
  Log {Received}, SysexSize, {bytes}
  Log {First byte:}, myBuffer[0]
@End
```

---

#### SysexSize

```mozaic
<result> = SysexSize
```

Returns the size (in bytes) of the last received SysEx message (not including 0xF0 and 0xF7). Use in `@OnSysex`.

```mozaic
@OnSysex
  sz = SysexSize
  Log {SysEx size:}, sz
@End
```

---

### AUv3 and Host Functions

---

#### SetShortName

```mozaic
SetShortName {shortname}
```

Set the short name of this AU plugin instance as shown in the host. Typically 5–8 characters. Call in `@OnLoad`.

```mozaic
@OnLoad
  SetShortName {MyFX}
@End
```

---

#### ShowLayout

```mozaic
ShowLayout <layout>
```

Change the GUI layout. Recommended to call in `@OnLoad`.

| Layout | Number | Description |
|--------|--------|-------------|
| Mix | 0 | 4 pads, 10 knobs, XY pad |
| Knobs | 1 | 22 knobs |
| Pads | 2 | 16 pads, 4 knobs |
| Sliders | 3 | 10 sliders, XY pad |
| Minimal | 4 | 4 knobs, description text box |

```mozaic
@OnLoad
  ShowLayout 1   // knobs layout
@End
```

---

#### SetMetroPPQN

```mozaic
SetMetroPPQN <ppqn>
```

Set the metronome pulses per quarter note (1–384). This determines how often `@OnMetroPulse` fires. Setting to 4 gives 16th-note pulses; setting to 1 gives quarter-note pulses.

```mozaic
@OnLoad
  SetMetroPPQN 4   // fire every 16th note
@End
```

---

#### SetMetroSwing

```mozaic
SetMetroSwing <percentage>
```

Set the metronome swing percentage (0–100). At 50, there is no swing. Higher values push the off-beats later.

```mozaic
SetMetroSwing 65   // slight swing
```

---

#### SetAUParameter

```mozaic
SetAUParameter <parameter>, <value>
```

Set an AU parameter value. Parameters are numbered 0–7. Values have 32-bit float resolution (but are typically mapped 0–127 in practice).

```mozaic
SetAUParameter 0, 64    // set parameter 0 to 64
```

---

#### GetAUParameter

```mozaic
<result> = GetAUParameter <parameter>
```

Get the current value of AU parameter 0–7. Returns a value with 32-bit float resolution.

```mozaic
val = GetAUParameter 2
Log {AU Param 2:}, val
```

---

#### LastAUParameter

```mozaic
<result> = LastAUParameter
```

Returns the index (0–7) of the last AU parameter changed by the host. Use inside `@OnAUParameter`.

```mozaic
@OnAUParameter
  p = LastAUParameter
  v = GetAUParameter p
  Log {Param}, p, {changed to}, v
@End
```

---

#### HostTempo

```mozaic
<result> = HostTempo
```

Returns the current host tempo in BPM. Can have a fractional part.

```mozaic
bpm = HostTempo
Log {Tempo:}, bpm
```

---

#### HostBar

```mozaic
<result> = HostBar
```

Returns the current bar number from the host (during playback).

```mozaic
bar = HostBar
```

---

#### HostBeat

```mozaic
<result> = HostBeat
```

Returns the current beat number within the bar from the host.

```mozaic
beat = HostBeat
```

---

#### HostBeatsPerMeasure

```mozaic
<result> = HostBeatsPerMeasure
```

Returns the number of beats per measure from the host time signature.

```mozaic
bpm = HostBeatsPerMeasure   // e.g. 4 for 4/4
```

---

#### HostRunning

```mozaic
<result> = HostRunning
```

Returns `YES` if the host transport is currently playing, `NO` if stopped.

```mozaic
if HostRunning
  Log {Host is playing}
endif
```

---

#### CurrentMetroPulse

```mozaic
<result> = CurrentMetroPulse
```

Returns the current metronome pulse counter. Resets to 0 at the start of each bar. Useful for rhythmic patterns.

```mozaic
@OnMetroPulse
  pulse = CurrentMetroPulse
  if pulse % 4 = 0
    Log {Quarter note}
  endif
@End
```

---

#### QuarterNote

```mozaic
<result> = QuarterNote
```

Returns the duration of a quarter note in milliseconds at the current tempo.

```mozaic
dur = QuarterNote
Log {Quarter note is}, dur, {ms}
```

---

### Timer and LFO Functions

---

#### StartTimer

```mozaic
StartTimer
```

Start the timer. Once started, `@OnTimer` fires repeatedly at the interval set by `SetTimerInterval`.

```mozaic
@OnLoad
  SetTimerInterval 100   // 100ms
  StartTimer
@End

@OnTimer
  // fires every 100ms
@End
```

---

#### StopTimer

```mozaic
StopTimer
```

Stop the timer. No more `@OnTimer` events will fire until `StartTimer` is called again.

```mozaic
StopTimer
```

---

#### ResetTimer

```mozaic
ResetTimer
```

Reset the timer countdown to the full interval, without stopping it.

```mozaic
ResetTimer
```

---

#### SetTimerInterval

```mozaic
SetTimerInterval <ms>
```

Set the timer interval in milliseconds. The timer is sample-accurate. Can be called at any time to change the interval dynamically.

```mozaic
SetTimerInterval 500    // fire every 500ms
SetTimerInterval QuarterNote / 4  // 16th note intervals
```

---

#### SetupLFO

```mozaic
SetupLFO <lfo>, <min>, <max>, <sync>, <freq>
```

Configure an LFO (Low Frequency Oscillator).

- `<lfo>`: LFO index 0–15
- `<min>`: minimum output value
- `<max>`: maximum output value
- `<sync>`: `YES` to sync to host tempo, `NO` for free-running Hz
- `<freq>`: if synced = periods per measure (e.g., 1.0 = 1 cycle/bar); if free = frequency in Hz

```mozaic
SetupLFO 0, 0, 127, YES, 1     // tempo-synced, 1 cycle/bar
SetupLFO 1, 0, 127, NO, 2.5    // free-running at 2.5 Hz
```

---

#### SetLFOType

```mozaic
SetLFOType <lfo>, {wave}
```

Set the waveform type for an LFO.

| Waveform | Name |
|----------|------|
| `{Sine}` | Sine wave |
| `{Cosine}` | Cosine wave |
| `{Square}` | Square wave |
| `{Triangle}` | Triangle wave |
| `{RampUp}` | Ramp Up (sawtooth) |
| `{RampDown}` | Ramp Down (reverse sawtooth) |
| `{SH}` | Sample & Hold (random steps) |

```mozaic
SetLFOType 0, {Sine}
SetLFOType 1, {Triangle}
SetLFOType 2, {SH}
```

---

#### ResetLFO

```mozaic
ResetLFO <lfo> [,<startphase>]
```

Reset an LFO to a given phase position.

- `<lfo>`: LFO index 0–15
- `<startphase>`: optional phase position: `0` = start of cycle, `0.5` = midpoint, `1.0` = end of cycle

```mozaic
ResetLFO 0           // reset to beginning
ResetLFO 0, 0.5      // reset to midpoint
```

---

#### GetLFOValue

```mozaic
<result> = GetLFOValue <lfo>
```

Get the current output value of an LFO (0–15). Output range is as configured with `SetupLFO`.

```mozaic
lfoVal = GetLFOValue 0
SendMIDICC 0, 1, lfoVal   // use LFO for mod wheel
```

---

### Musical Scale Functions

---

#### PresetScale

```mozaic
PresetScale {scalename}
PresetScale <scalenumber>
```

Activate one of the 25 built-in preset scales.

| Number | Name |
|--------|------|
| 0 | Chromatic |
| 1 | Major |
| 2 | Minor |
| 3 | MinorMelodic |
| 4 | MinorHarmonic |
| 5 | MajorPentatonic |
| 6 | MinorPentatonic |
| 7 | Aeolian |
| 8 | Dorian |
| 9 | Lydian |
| 10 | Mixolydian |
| 11 | Phrygian |
| 12 | Blues |
| 13 | WholeTone |
| 14 | Diminished |
| 15 | Bhairavi |
| 16 | Gypsy |
| 17 | Klezmer |
| 18 | Octave |
| 19 | Andean |
| 20 | Iwato |
| 21 | InSen |
| 22 | HiraJoshi |
| 23 | Pelog |
| 24 | Yo |

```mozaic
PresetScale {Major}
PresetScale {MinorPentatonic}
PresetScale 12     // Blues
```

---

#### CustomScale

```mozaic
CustomScale <c>, <c#>, <d>, <d#>, <e>, <f>, <f#>, <g>, <g#>, <a>, <a#>, <b>
```

Define a custom scale using 12 boolean values (YES/NO or 1/0), one per semitone from C to B.

```mozaic
// Whole tone scale starting from C
CustomScale YES, NO, YES, NO, YES, NO, YES, NO, YES, NO, YES, NO
```

---

#### SetRootNote

```mozaic
SetRootNote <notenum>
```

Set the root note for the active scale.

| Note | Number |
|------|--------|
| C | 0 |
| C# | 1 |
| D | 2 |
| D# | 3 |
| E | 4 |
| F | 5 |
| F# | 6 |
| G | 7 |
| G# | 8 |
| A | 9 |
| A# | 10 |
| B | 11 |

```mozaic
SetRootNote 0    // C
SetRootNote 9    // A
```

---

#### InScale

```mozaic
<result> = InScale <notenum>
```

Returns `YES` if the given MIDI note number is in the currently active scale (taking root note into account), `NO` otherwise.

```mozaic
if InScale MIDINote
  SendMIDIThru
endif
```

---

#### ScaleQuantize

```mozaic
<result> = ScaleQuantize <midinote>
```

Quantize a MIDI note to the nearest in-scale note. Returns the next higher note that is in the current scale. If the input note is already in scale, it is returned unchanged.

```mozaic
quantizedNote = ScaleQuantize MIDINote
SendMIDINoteOn MIDIChannel, quantizedNote, MIDIVelocity
```

---

### GUI and Interaction Functions

---

#### LabelPad

```mozaic
LabelPad <pad>, {label}, <value>, ..., <value>
```

Set the label of a pad (0–15). The label is a string that can include variable values using `{label}` strings and numeric `<value>` parameters interleaved.

```mozaic
LabelPad 0, {Kick}
LabelPad 1, {Note }, MIDINote
```

---

#### LabelPads

```mozaic
LabelPads {label}, <value>, ..., <value>
```

Set the section title for the pads area.

```mozaic
LabelPads {Drum Pads}
```

---

#### LabelKnob

```mozaic
LabelKnob <knob>, {label}, <value>, ..., <value>
```

Set the label of a knob (0–21).

```mozaic
LabelKnob 0, {Volume}
LabelKnob 1, {Cutoff: }, GetKnobValue 1
```

---

#### LabelKnobs

```mozaic
LabelKnobs {label}, <value>, ..., <value>
```

Set the section title for the knobs area.

```mozaic
LabelKnobs {Filter Controls}
```

---

#### LabelXY

```mozaic
LabelXY {label}, <value>, ..., <value>
```

Set the label/title for the XY pad.

```mozaic
LabelXY {Pitch / Mod}
```

---

#### ColorPad

```mozaic
ColorPad <padnum>, <colornum>
```

Set the color tint of a pad (0–15).

| Color | Number |
|-------|--------|
| White (default) | 0 |
| Red | 1 |
| Yellow | 2 |
| Green | 3 |
| Light blue | 4 |
| Lavender/purple | 5 |
| Violet | 6 |
| Magenta | 7 |

```mozaic
ColorPad 0, 3    // pad 0 = green
ColorPad LastPad, 1  // last touched pad = red
```

---

#### LatchPad

```mozaic
LatchPad <pad>, <state>
```

Set the background LED of a pad on or off (0–15). State is `YES`/`NO` or `1`/`0`.

```mozaic
LatchPad 0, YES   // turn on LED
LatchPad 0, NO    // turn off LED
```

---

#### PadState

```mozaic
<result> = PadState <padnum>
```

Returns `YES` if the pad's latch LED is currently on, `NO` if off.

```mozaic
if PadState 0
  LatchPad 0, NO
else
  LatchPad 0, YES
endif
```

---

#### FlashPad

```mozaic
FlashPad <pad>
```

Briefly flash the pad (0–15) on screen to provide visual feedback.

```mozaic
@OnPadDown
  FlashPad LastPad
@End
```

---

#### FlashUserLed

```mozaic
FlashUserLed
```

Flash the USER LED indicator in the GUI.

```mozaic
@OnNewBeat
  FlashUserLed
@End
```

---

#### SetKnobValue

```mozaic
SetKnobValue <knob>, <value>
```

Programmatically set a knob's value (0–21, range 0–127). Note: this does **NOT** trigger `@OnKnobChange`.

```mozaic
SetKnobValue 0, 64    // set knob 0 to center
```

---

#### GetKnobValue

```mozaic
<result> = GetKnobValue <knob>
```

Get the current value of a knob (0–21). Returns a floating point value in the range 0–127.

```mozaic
vol = GetKnobValue 0
```

---

#### LastKnob

```mozaic
<result> = LastKnob
```

Returns the index (0–21) of the last knob that was changed. Use inside `@OnKnobChange`.

```mozaic
@OnKnobChange
  k = LastKnob
  v = GetKnobValue k
  Log {Knob}, k, {=}, v
@End
```

---

#### LastPad

```mozaic
<result> = LastPad
```

Returns the index (0–15) of the last pad that was touched or released. Use inside `@OnPadDown` or `@OnPadUp`.

```mozaic
@OnPadDown
  p = LastPad
  Log {Pad tapped:}, p
@End
```

---

#### LastPadVelocity

```mozaic
<result> = LastPadVelocity
```

Returns the velocity of the last pad tap. Tapping the center gives velocity 127; tapping near the edges gives a lower value.

```mozaic
@OnPadDown
  vel = LastPadVelocity
  SendMIDINoteOn 0, 60, vel
@End
```

---

#### SetXYValues

```mozaic
SetXYValues <xvalue>, <yvalue>
```

Programmatically set the XY pad position (0–127 each). Note: this does **NOT** trigger `@OnXYChange`.

```mozaic
SetXYValues 64, 64   // center
```

---

#### GetXValue

```mozaic
<result> = GetXValue
```

Get the current X axis value of the XY pad (0–127).

```mozaic
x = GetXValue
```

---

#### GetYValue

```mozaic
<result> = GetYValue
```

Get the current Y axis value of the XY pad (0–127).

```mozaic
y = GetYValue
```

---

#### GetXYMorphValue

```mozaic
<result> = GetXYMorphValue <topleft>, <topright>, <bottomleft>, <bottomright>
```

Perform bilinear interpolation across the XY pad. Returns a value interpolated between the four corner values based on the current XY pad position.

- `<topleft>`: value at top-left corner
- `<topright>`: value at top-right corner
- `<bottomleft>`: value at bottom-left corner
- `<bottomright>`: value at bottom-right corner

```mozaic
// morph between 4 MIDI CC values based on XY position
morphed = GetXYMorphValue 0, 127, 64, 32
SendMIDICC 0, 1, morphed
```

---

#### MotionPitch / MotionRoll / MotionYaw

```mozaic
<result> = MotionPitch
<result> = MotionRoll
<result> = MotionYaw
```

Return the current iOS device tilt sensor values.

- Range: 0–127
- Center/reference angle: **64**
- Tilt the device to change the values

```mozaic
pitch = MotionPitch
SendMIDICC 0, 74, pitch
```

---

#### ShiftPressed

```mozaic
<result> = ShiftPressed
```

Returns `YES` if the Shift button is currently held, `NO` otherwise.

```mozaic
@OnPadDown
  if ShiftPressed
    // alternate function
  else
    // normal function
  endif
@End
```

---

#### Log

```mozaic
Log <value>, <value>, {string}, ..., <value>
```

Print a message to the Mozaic Log window. Accepts any combination of numeric values and string literals. Useful for debugging.

```mozaic
Log {Hello World}
Log {Note:}, MIDINote, {vel:}, MIDIVelocity
Log {Knob}, LastKnob, {value is}, GetKnobValue LastKnob
```

---

#### LogTime

```mozaic
LogTime
```

Print timing information to the Log window for debugging performance.

```mozaic
LogTime
```

---

#### NoteName

```mozaic
{notename} = NoteName <note> [,<showoctave>]
```

A **string macro** — returns the name of a MIDI note number as a string. Can only be used inside `Log` or `Label` commands, not assigned to a variable.

- `<note>`: MIDI note number 0–127
- `<showoctave>`: optional; `YES` to include octave number (e.g., "C4"), `NO` for just note name (e.g., "C")

```mozaic
Log {Note name:}, NoteName MIDINote
Log {Note:}, NoteName MIDINote, YES
LabelPad 0, NoteName 60, YES   // "C4"
```

---

#### RootNoteName

```mozaic
{notename} = RootNoteName
```

A **string macro** — returns the name of the current scale root note. Can only be used inside `Log` or `Label` commands.

```mozaic
Log {Root note:}, RootNoteName
LabelKnobs {Scale: }, RootNoteName
```

---

#### ScaleName

```mozaic
{scalename} = ScaleName [<scalenum>]
```

A **string macro** — returns the name of a scale. If `<scalenum>` is provided, returns that scale's name; otherwise returns the current active scale's name. Can only be used inside `Log` or `Label` commands.

```mozaic
Log {Scale:}, ScaleName
Log {Scale 5:}, ScaleName 5    // "MajorPentatonic"
LabelKnobs ScaleName
```

---

### Variables and Math Functions

---

#### FillArray

```mozaic
FillArray <var>, <value> [,<numcells>]
```

Fill an array with a value. If the array does not yet exist, it creates it. Optionally specify the number of cells to fill (default fills all 1024).

```mozaic
FillArray myArray, 0         // fill all 1024 cells with 0
FillArray myArray, 127, 16   // fill first 16 cells with 127
```

---

#### CopyArray

```mozaic
CopyArray <sourcevar>, <destvar> [,<numcells>]
```

Copy an array from source to destination. Optionally specify the number of cells to copy.

```mozaic
CopyArray sourceArray, destArray
CopyArray sourceArray, destArray, 16   // copy first 16 cells
```

---

#### Inc

```mozaic
Inc <var> [,<max>]
```

Increment a variable by 1. If the optional `<max>` is specified and the variable reaches or exceeds `<max>`, it wraps back to 0. Default max is 65535.

```mozaic
Inc counter           // increment counter
Inc step, 16         // 0..15 counter, wraps to 0 after 15
```

---

#### Dec

```mozaic
Dec <var> [,<min>]
```

Decrement a variable by 1. If the optional `<min>` is specified and the variable reaches `<min>`, it stops decrementing. Default min is 0.

```mozaic
Dec counter          // decrement counter
Dec level, 0         // stop at 0
```

---

#### Unassigned

```mozaic
<bool> = Unassigned <var>
```

Returns `YES` if the variable does not yet exist (has never been assigned), `NO` if it already exists. Useful for safe initialization in `@OnLoad` that won't overwrite state on preset reloads.

```mozaic
@OnLoad
  if Unassigned myState
    myState = 0
  endif
@End
```

---

#### Round / RoundUp / RoundDown

```mozaic
<result> = Round <value>
<result> = RoundUp <value>
<result> = RoundDown <value>
```

Round a value to the nearest integer (Round), upward (RoundUp/ceiling), or downward (RoundDown/floor).

```mozaic
x = Round 3.7      // x = 4
x = RoundUp 3.2    // x = 4
x = RoundDown 3.9  // x = 3
```

---

#### Random

```mozaic
<result> = Random <min>, <max>
```

Return a random integer in the range `<min>` to `<max>` (inclusive).

```mozaic
note = Random 48, 72    // random note between C3 and C5
vel = Random 64, 127
```

---

#### Clip

```mozaic
<result> = Clip <value>, <min>, <max>
```

Clamp a value to the range `[<min>, <max>]`. If value is below min, returns min. If above max, returns max.

```mozaic
safe = Clip myVal, 0, 127
```

---

#### TranslateScale

```mozaic
<result> = TranslateScale <input>, <inputmin>, <inputmax>, <outputmin>, <outputmax>
```

Map a value from one range to another (linear scaling).

```mozaic
// map knob (0-127) to MIDI note range (48-72)
note = TranslateScale GetKnobValue 0, 0, 127, 48, 72

// map LFO output (0-127) to pitchbend range (0-16383)
pb = TranslateScale GetLFOValue 0, 0, 127, 0, 16383
```

---

#### TranslateCurve

```mozaic
<result> = TranslateCurve <input>, <curve>, <min>, <max>
```

Apply a non-linear power curve to map an input (0–127) to a range.

- `<input>`: value 0–127
- `<curve>`: `1.0` = linear; `< 1.0` = biased toward maximum (more high values); `> 1.0` = biased toward minimum (more low values)
- `<min>`: minimum output value
- `<max>`: maximum output value

```mozaic
curved = TranslateCurve MIDIVelocity, 2.0, 0, 127  // soft velocity curve
```

---

#### Abs

```mozaic
<result> = Abs <value>
```

Return the absolute value of a number.

```mozaic
x = Abs -5    // x = 5
x = Abs myVar
```

---

#### Div

```mozaic
<result> = Div <value1>, <value2>
```

Perform integer division (discards the fractional part). Equivalent to `RoundDown (value1 / value2)`.

```mozaic
x = Div 7, 2    // x = 3
octave = Div MIDINote, 12
```

---

#### Sin / Cos / Tan / Tanh

```mozaic
<result> = Sin <value>
<result> = Cos <value>
<result> = Tan <value>
<result> = Tanh <value>
```

Trigonometric functions. Input in radians.

```mozaic
s = Sin 1.5708    // sin(pi/2) = 1.0
c = Cos 0         // cos(0) = 1.0
```

---

#### Exp

```mozaic
<result> = Exp <value>
```

Return e raised to the power of `<value>`.

```mozaic
x = Exp 1    // x = 2.71828...
```

---

#### Sqrt

```mozaic
<result> = Sqrt <value>
```

Return the square root of a value.

```mozaic
x = Sqrt 16    // x = 4
```

---

#### Pow

```mozaic
<result> = Pow <base>, <exponent>
```

Return `<base>` raised to the power of `<exponent>`.

```mozaic
x = Pow 2, 8    // x = 256
x = Pow 10, 3   // x = 1000
```

---

#### Logn

```mozaic
<result> = Logn <value>
```

Return the natural logarithm (base e) of a value.

```mozaic
x = Logn 2.71828   // x ≈ 1.0
```

---

#### Log10

```mozaic
<result> = Log10 <value>
```

Return the base-10 logarithm of a value.

```mozaic
x = Log10 100    // x = 2
x = Log10 1000   // x = 3
```

---

### NoteState Matrix Functions

The NoteState matrix is a 16-channel × 128-note virtual "locker room" — a 2D array for storing per-note-per-channel state information. Useful for tracking which notes are on, storing note-specific data, etc.

---

#### SetNoteState

```mozaic
SetNoteState <chan>, <note>, <value>
```

Store a value in the NoteState matrix at position `[chan][note]`.

- `<chan>`: MIDI channel 0–15
- `<note>`: note number 0–127
- `<value>`: any numeric value

```mozaic
@OnMidiNoteOn
  SetNoteState MIDIChannel, MIDINote, MIDIVelocity
@End

@OnMidiNoteOff
  SetNoteState MIDIChannel, MIDINote, 0
@End
```

---

#### GetNoteState

```mozaic
<result> = GetNoteState <chan>, <note>
```

Get the value stored in the NoteState matrix at position `[chan][note]`.

```mozaic
vel = GetNoteState MIDIChannel, MIDINote
if vel > 0
  Log {Note is on with velocity}, vel
endif
```

---

#### ResetNoteStates

```mozaic
ResetNoteStates [<default>]
```

Reset the entire NoteState matrix to a default value (default is 0 if not specified).

```mozaic
ResetNoteStates       // fill all with 0
ResetNoteStates 0     // same
```

---

## 10. Function Overview by Category (Cheat Sheet)

### MIDI Functions

| Function | Description |
|----------|-------------|
| `SendMIDIOut b1, b2, b3 [,delay]` | Send raw 3-byte MIDI |
| `SendMIDINoteOn ch, note, vel [,delay]` | Send Note On |
| `SendMIDINoteOff ch, note, vel [,delay]` | Send Note Off |
| `SendMIDICC ch, cc, val [,delay]` | Send Control Change |
| `SendMIDIPitchbend ch, val [,delay]` | Send Pitchbend (0-16383, center=8192) |
| `SendMIDIProgramChange ch, patch [,delay]` | Send Program Change |
| `SendMIDIBankSelect ch, msb, lsb [,delay]` | Send Bank Select |
| `SendMIDIThru [delay]` | Forward MIDI input to output |
| `SendMIDIThruOnCh ch [,delay]` | Forward MIDI input to specific channel |
| `ConfigureMPE lower, upper` | Send MPE Configuration Message |
| `MIDIChannel` | Channel of current event (0-15) |
| `MIDICommand` | Command byte (0x90, 0x80, 0xB0, etc.) |
| `MIDINote` | Note number of current event (0-127) |
| `MIDIVelocity` | Velocity of current note event |
| `MIDIByte1` / `MIDIByte2` / `MIDIByte3` | Raw MIDI bytes |
| `MIDISustainPedalDown` | YES if sustain pedal held |

### Sysex Functions

| Function | Description |
|----------|-------------|
| `SendSysex arr, len [,checksum, startidx]` | Send SysEx from array |
| `SendSysexThru` | Forward received SysEx unchanged |
| `ReceiveSysex arr` | Copy received SysEx into array |
| `SysexSize` | Size of last received SysEx |

### AUv3 and Host Functions

| Function | Description |
|----------|-------------|
| `SetShortName {name}` | Set AU instance short name |
| `SetMetroPPQN ppqn` | Set metronome pulses per quarter note (1-384) |
| `SetMetroSwing pct` | Set metronome swing (0-100) |
| `SetAUParameter param, val` | Set AU parameter (0-7) |
| `GetAUParameter param` | Get AU parameter value |
| `LastAUParameter` | Last AU parameter changed by host |
| `HostTempo` | Current host BPM |
| `HostBar` | Current bar number |
| `HostBeat` | Current beat within bar |
| `HostBeatsPerMeasure` | Beats per measure from time signature |
| `HostRunning` | YES if host is playing |
| `CurrentMetroPulse` | Current metro pulse (resets each bar) |
| `QuarterNote` | Quarter note duration in milliseconds |

### Timer and LFO Functions

| Function | Description |
|----------|-------------|
| `StartTimer` | Start the timer |
| `StopTimer` | Stop the timer |
| `ResetTimer` | Reset timer countdown |
| `SetTimerInterval ms` | Set timer interval in milliseconds |
| `SetupLFO lfo, min, max, sync, freq` | Configure LFO |
| `SetLFOType lfo, {wave}` | Set LFO waveform |
| `ResetLFO lfo [,phase]` | Reset LFO phase |
| `GetLFOValue lfo` | Get current LFO output value |

### Musical Scale Functions

| Function | Description |
|----------|-------------|
| `CustomScale c,c#,d,d#,e,f,f#,g,g#,a,a#,b` | Define custom scale (12 booleans) |
| `PresetScale {name}` or `PresetScale num` | Activate a preset scale |
| `SetRootNote notenum` | Set scale root note (C=0 .. B=11) |
| `InScale notenum` | YES if note is in current scale |
| `ScaleQuantize note` | Quantize note to current scale |

### GUI and Interaction Functions

| Function | Description |
|----------|-------------|
| `Exit` | Exit current event handler immediately |
| `ShowLayout num` | Change GUI layout (0-4) |
| `LabelPad pad, {text}, ...` | Set pad label |
| `LabelPads {text}, ...` | Set pads section title |
| `LabelKnob knob, {text}, ...` | Set knob label |
| `LabelKnobs {text}, ...` | Set knobs section title |
| `LabelXY {text}, ...` | Set XY pad title |
| `SetKnobValue knob, val` | Set knob position (no event fired) |
| `GetKnobValue knob` | Get knob value (0-127, float) |
| `ColorPad pad, color` | Set pad color (0-7) |
| `LatchPad pad, state` | Set pad LED on/off |
| `PadState pad` | YES if pad LED is on |
| `FlashPad pad` | Flash pad briefly |
| `FlashUserLed` | Flash USER LED |
| `SetXYValues x, y` | Set XY pad position (no event fired) |
| `GetXValue` | Get XY X axis value (0-127) |
| `GetYValue` | Get XY Y axis value (0-127) |
| `GetXYMorphValue tl, tr, bl, br` | Bilinear interpolation on XY |
| `LastKnob` | Index of last changed knob |
| `LastPad` | Index of last touched/released pad |
| `LastPadVelocity` | Velocity of last pad tap |
| `ShiftPressed` | YES if Shift button is held |
| `MotionPitch` / `MotionRoll` / `MotionYaw` | iOS tilt sensor (0-127, 64=center) |
| `Log val, {str}, ...` | Print to Log window |
| `LogTime` | Print timing info |
| `NoteName note [,showoctave]` | String macro: note name |
| `RootNoteName` | String macro: root note name |
| `ScaleName [scalenum]` | String macro: scale name |

### Variables and Math Functions

| Function | Description |
|----------|-------------|
| `FillArray arr, val [,count]` | Fill array with value |
| `CopyArray src, dst [,count]` | Copy array |
| `Inc var [,max]` | Increment (wraps at max) |
| `Dec var [,min]` | Decrement (stops at min) |
| `Unassigned var` | YES if variable does not exist yet |
| `Round val` | Round to nearest integer |
| `RoundUp val` | Round up (ceiling) |
| `RoundDown val` | Round down (floor) |
| `Random min, max` | Random integer in range |
| `Clip val, min, max` | Clamp value to range |
| `TranslateScale in, imin, imax, omin, omax` | Map value between ranges |
| `TranslateCurve in, curve, min, max` | Apply power curve mapping |
| `Sin val` | Sine |
| `Cos val` | Cosine |
| `Tan val` | Tangent |
| `Tanh val` | Hyperbolic tangent |
| `Exp val` | e^val |
| `Sqrt val` | Square root |
| `Abs val` | Absolute value |
| `Logn val` | Natural logarithm |
| `Log10 val` | Base-10 logarithm |
| `Pow base, exp` | Power function |
| `Div val1, val2` | Integer division |

### NoteState Matrix Functions

| Function | Description |
|----------|-------------|
| `SetNoteState ch, note, val` | Store value in NoteState matrix |
| `GetNoteState ch, note` | Get value from NoteState matrix |
| `ResetNoteStates [default]` | Reset entire NoteState matrix |

---

## Appendix: Quick Reference

### Events Summary

| Event | When |
|-------|------|
| `@OnLoad` | Script/preset loaded, state restored |
| `@OnMidiInput` | Any MIDI received (always first) |
| `@OnMidiNote` | Note On or Off |
| `@OnMidiNoteOn` | Note On |
| `@OnMidiNoteOff` | Note Off |
| `@OnMidiCC` | Control Change |
| `@OnNewBeat` | Host new beat |
| `@OnNewBar` | Host new bar |
| `@OnMetroPulse` | Internal metro pulse |
| `@OnKnobChange` | Knob turned |
| `@OnPadDown` | Pad pressed |
| `@OnPadUp` | Pad released |
| `@OnTimer` | Timer interval |
| `@OnXYChange` | XY pad moved |
| `@OnSysex` | SysEx received |
| `@OnShiftDown` | Shift pressed |
| `@OnShiftUp` | Shift released |
| `@OnHostStart` | Host transport start |
| `@OnHostStop` | Host transport stop |
| `@OnAUParameter` | AU parameter changed by host |
| `@OnPedalDown` | Sustain pedal pressed |
| `@OnPedalUp` | Sustain pedal released |
| `@Description` | Plugin description text (not an event) |

### MIDI Note Numbers

| Note | Octave -1 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|------|-----------|---|---|---|---|---|---|---|---|---|---|
| C | 0 | 12 | 24 | 36 | 48 | 60 | 72 | 84 | 96 | 108 | 120 |
| C# | 1 | 13 | 25 | 37 | 49 | 61 | 73 | 85 | 97 | 109 | 121 |
| D | 2 | 14 | 26 | 38 | 50 | 62 | 74 | 86 | 98 | 110 | 122 |
| D# | 3 | 15 | 27 | 39 | 51 | 63 | 75 | 87 | 99 | 111 | 123 |
| E | 4 | 16 | 28 | 40 | 52 | 64 | 76 | 88 | 100 | 112 | 124 |
| F | 5 | 17 | 29 | 41 | 53 | 65 | 77 | 89 | 101 | 113 | 125 |
| F# | 6 | 18 | 30 | 42 | 54 | 66 | 78 | 90 | 102 | 114 | 126 |
| G | 7 | 19 | 31 | 43 | 55 | 67 | 79 | 91 | 103 | 115 | 127 |
| G# | 8 | 20 | 32 | 44 | 56 | 68 | 80 | 92 | 104 | 116 | — |
| A | 9 | 21 | 33 | 45 | 57 | 69 | 81 | 93 | 105 | 117 | — |
| A# | 10 | 22 | 34 | 46 | 58 | 70 | 82 | 94 | 106 | 118 | — |
| B | 11 | 23 | 35 | 47 | 59 | 71 | 83 | 95 | 107 | 119 | — |

Middle C = 60 (C4)

### Complete Example Script

```mozaic
// Chord Transposer — transposes chords based on active scale
// Knob 0: transpose amount (-12 to +12)
// Pads: toggle note modification on/off

@OnLoad
  ShowLayout 0
  SetShortName {ChTrns}
  PresetScale {Major}
  SetRootNote 0
  LabelKnob 0, {Transpose}
  SetKnobValue 0, 64
  transposeOn = YES
  LatchPad 0, YES
  LabelPad 0, {On/Off}
@End

@OnKnobChange
  if LastKnob = 0
    // map knob 0-127 to transpose range -12 to +12
    transpose = TranslateScale GetKnobValue 0, 0, 127, -12, 12
    transpose = Round transpose
  endif
@End

@OnPadDown
  if LastPad = 0
    if transposeOn
      transposeOn = NO
      LatchPad 0, NO
    else
      transposeOn = YES
      LatchPad 0, YES
    endif
  endif
@End

@OnMidiNoteOn
  if transposeOn
    transposed = MIDINote + transpose
    transposed = Clip transposed, 0, 127
    SendMIDINoteOn MIDIChannel, transposed, MIDIVelocity
    SetNoteState MIDIChannel, MIDINote, transposed
  else
    SendMIDIThru
    SetNoteState MIDIChannel, MIDINote, MIDINote
  endif
@End

@OnMidiNoteOff
  original = GetNoteState MIDIChannel, MIDINote
  if original > 0
    SendMIDINoteOff MIDIChannel, original, MIDIVelocity
    SetNoteState MIDIChannel, MIDINote, 0
  endif
@End

@Description
  Transposes MIDI notes by a set amount.
  Use Knob 0 to set transposition (-12 to +12 semitones).
  Use Pad 0 to toggle transposition on/off.
@End
```
