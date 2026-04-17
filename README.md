# SWAM Smart Control

A Mozaic AUv3 script for iOS that provides A/B state morphing for SWAM instruments. Stores two snapshots of 10 SWAM expression parameters (as MIDI CC) per scene, then smoothly interpolate between them with a compressable morph curve.

---

## Features

- **10 SWAM parameters** — each independently stored in State A and State B
- **4 scenes** — completely independent A/B pairs per scene
- **Morph with compression** — morph signal from A to B
- **Full MIDI passthrough** — non-reserved CCs and all other channels forwarded untouched
- **Hardware-friendly** — all UI features are duplicated as MIDI CC and AU parameters

---

## Layout

Mozaic **Mix layout** (Layout 0): 4 pads + 10 knobs + XY pad.

| Area   | Count | Function                            |
|--------|-------|-------------------------------------|
| Pads   | 4     | Scene select (1–4)                  |
| Knobs  | 10    | Edit A or B value for current scene |
| XY Pad | 1     | X = Morph Scale, Y = Morph Center   |
| Shift  | 1     | Toggle A/B + tiered reset gestures  |

---

## Usage
1. State A is enabled by default. Change parameter knobs (via UI or related input CCs) to set state A values
2. Tap "Shift" button to switch to the state B. Change parameter knobs again to set state B values
3. Morph between states with AU Param 0 or CC 11
4. Switch between 4 such separate pairs (scenes) with UI Pads / MIDI notes / AU Param 1 or CC 2
5. Use XY Pad to "Compress" the morphing sweep (see the following sections) 

---

## Parameter Map

All signals are on MIDI channel 1,. 
- You can customize `CONTROL_CHANNEL` and `OUTPUT_CHANNEL` variables
- `swam-midi-mappings/` contains MIDI mapping presets for SWAM instruments that already map these parameters to the CCs
- All messages on channels other than `CONTROL_CHANNEL`: forwarded as-is
- `CONTROL_CHANNEL` CC messages: blocked if they are reserved by the script (see the mapping table). All other CCs forwarded
- `CONTROL_CHANNEL` Note On/Off for notes 0–3: blocked (used for scene select). All other notes forwarded
- Parameter's CCs are never forwarded raw — the processed (morphed) value is sent instead

| Parameter            | Mozaic Knob | Input CC | Output CC |
|----------------------|-------------|----------|-----------|
| Vibrato Depth        | Knob 5      | CC 15    | CC 15     |
| Vibrato Freq         | Knob 0      | CC 16    | CC 16     |
| Tremolo / Mute Move  | Knob 6      | CC 17    | CC 17     |
| Tremolo Freq         | Knob 1      | CC 18    | CC 18     |
| Bow Position / Growl | Knob 7      | CC 19    | CC 19     |
| Bow Pressure         | Knob 2      | CC 20    | CC 20     |
| Flutter              | Knob 8      | CC 21    | CC 21     |
| Keys Sound           | Knob 4      | CC 23    | CC 23     |
| Breath Sound         | Knob 9      | CC 25    | CC 25     |
| Harmonics            | Knob 3      | CC 27    | CC 27     |

### Control Signals (input only, not forwarded)

| Function          | Mozaic Control | Input  | Notes                                  |
|-------------------|----------------|--------|----------------------------------------|
| State Morph (A→B) | AU Param 0     | CC 11  | 0=full A, 127=full B                   |
| Scene Select      | AU Param 1     | CC 2   | 0–31=S1, 32–63=S2, 64–95=S3, 96–127=S4 |
| Edit State A      | -              | CC 12  |                                        |
| Edit State B      | -              | CC 13  |                                        |
| Switch Edit State | Shift Button   | CC 14  | Also used for quick reset              |
| Scene 1           | Pad 0          | Note 0 |                                        |
| Scene 2           | Pad 1          | Note 1 |                                        |
| Scene 3           | Pad 2          | Note 2 |                                        |
| Scene 4           | Pad 3          | Note 3 |                                        |
| Compressor Scale  | XY Pad X       | CC 31  | 0=no sweep, 127=full                   |
| Compressor Center | XY Pad Y       | CC 30  | 0=State A, 64=center, 127=State B      |

---

## Morph Compressor

The morph compressor remaps the incoming morph signal before it drives the A→B blend. This lets you constrain how much of the A→B range is swept across the full 0–127 morph input. Basically, this allows quickly adjust the "intensity" of morphing without changing A/B states of each parameter separately.

### Parameters

- **Scale** (XY X / CC 31): width of the morph window. `127` = full range `[0, 127]`. `0` = no sweep (locked at center).
- **Center** (XY Y / CC 30): midpoint of the morph window in 0–127 space. `64` = 50%.

### Examples

| Scale | Center | morphStart | morphEnd | Effect                                   |
|-------|--------|------------|----------|------------------------------------------|
| 127   | 64     | 0          | 127      | Full sweep — no compression              |
| 64    | 64     | 32         | 96       | Middle 50% of A→B range swept            |
| 64    | 89     | 57         | 121      | Upper half, shifted toward B             |
| 0     | 64     | 64         | 64       | Frozen at 50% blend                      |
| 127   | 0      | 0          | 63       | Full morph input maps to lower half only |

---

## Resetting parameters

The Shift button performs different reset actions depending on hold duration. Pad colors give live feedback while holding.

| Hold Duration | Pad Color         | Action on Release                                                     |
|---------------|-------------------|-----------------------------------------------------------------------|
| < 1s (tap)    | Current A/B color | Toggle A/B edit mode                                                  |
| 1–2s          | Light blue        | Zero all 10 params for current state (A or B) in current scene        |
| 2–3s          | Yellow            | Zero both A and B for current scene                                   |
| > 3s          | Red               | Zero all 4 scenes + reset compressor (full reset to default settings) |

---

## Pad Colors

| Color            | Meaning                                           |
|------------------|---------------------------------------------------|
| White            | Editing State A (default)                         |
| Purple           | Editing State B                                   |
| Light blue       | Hold active — will reset current state on release |
| Yellow           | Hold active — will reset current scene on release |
| Red              | Hold active — will reset everything on release    |
| Latched (bright) | Currently selected scene                          |
