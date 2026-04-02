# Project Reference - MPF "War" Pinball Machine

## Overview
This is a Mission Pinball Framework (MPF) project controlling a custom pinball machine. MPF is an open-source Python framework where the entire machine — hardware, rules, scoring, displays, modes — is defined through YAML configuration files. No Python code is required for standard functionality.

## Hardware Platform
- **Controller**: OPP (Open Pinball Project) hardware
- **Driver boards**: Gen2
- **COM Ports**: COM3, COM4
- **Displays**: NeoSeg light segment displays (two sets: orange and blue)

## Machine Layout
This appears to be a **dual-playfield** machine (components suffixed `_sd1` and `_sd2`):

### Per Side (sd1 & sd2)
- Left & right flippers
- Left & right slingshots
- Outhole and shooter lane (with coils)
- Spinner
- 1000-point and 100-point targets
- "Kill flip" target (10,000 pts)

### Shared
- Left & right pop bumpers
- Coin switch (tagged `start`)
- 1 ball installed, 3 balls per game, max_players=1 (MPF sees a 1-player game)
- Two-player scoring is handled entirely through machine variables (`p1_score`, `p2_score`), bypassing MPF's player system

## Ball Flow
Troughs are tagged `trough, drain`. Plungers are tagged `home` — the ball is considered "ready" once it's in the shooter lane.

1. Ball drains into either `bd_trough` (sd1) or `bd_trough_sd2` (sd2)
2. Trough auto-ejects to its plunger (`bd_plunger` or `bd_plunger_sd2`)
3. Ball is now home — player presses their right flipper to launch
4. Plunger confirms eject when ball reaches the opposing outhole
5. Ball plays across both sides until it drains again

## Modes
- **attract** — Cycles "GameOver" / "JOUST" text on segment displays
- **base** — Active during ball play; handles scoring for all switches

## Game Concept
This is a **competitive simultaneous two-player** pinball machine. The playfield is split in half — each player controls their own side. Hitting targets on your opponent's side scores points for you.

## Scoring (base mode)
SD1 = Player 1's side, SD2 = Player 2's side. Targets score for the **opposing** player.

| Target | Points | Scores For | Eyes | Other Effects |
|---|---|---|---|---|
| Slingshots (sd1) | 1,000 | Player 2 | ✓ | |
| Slingshots (sd2) | 1,000 | Player 1 | ✓ | |
| Kill flip (sd1) | 10,000 | Player 2 | | Disables P1 flippers for 2s |
| Kill flip (sd2) | 10,000 | Player 1 | | Disables P2 flippers for 2s |
| Spinners (sd1) | 500 | Player 2 | | |
| Spinners (sd2) | 500 | Player 1 | | |
| 1000pt targets (sd1) | 1,000 | Player 2 | | |
| 1000pt targets (sd2) | 1,000 | Player 1 | | |
| 100pt targets (sd1) | 100 | Player 2 | | |
| 100pt targets (sd2) | 100 | Player 1 | | |
| Outhole (sd1) | 10,000 | Player 2 | ✓ | |
| Outhole (sd2) | 10,000 | Player 1 | ✓ | |
| Pop bumper left | 50 | Player 1 | ✓ | |
| Pop bumper right | 50 | Player 2 | ✓ | |
| Flippers (sd1) | — | — | ✓ | |
| Flippers (sd2) | — | — | ✓ | |
| Plunger (sd1) | — | — | ✓ | |
| Plunger (sd2) | — | — | ✓ | |

## Segment Displays
- `neoSeg0` (orange 8-char) — Player 1's score (SD1 side)
- `neoSeg1` (blue 8-char) — Player 2's score (SD2 side)
- `neoSeg2d0` (orange 2-char) — Ball number during play, "P1" in attract (SD1 side)
- `neoSeg2d2` (blue 2-char) — Ball number during play, "P2" in attract (SD2 side)

Note: `neoSeg2d1` and `neoSeg2d3` exist in the LED chain configs (neoSeg_2dig_1.yaml, neoSeg_2dig_3.yaml) for daisy-chain addressing but have no physical display hardware. They are not referenced in display.yaml or attract mode.

## Other Hardware
- `c_viking_eyes` (coil 1-0-4) — 4 LEDs on a single driver, viking eye lights. Pulses 200ms. See Eyes column in scoring table above for triggers.

No turn-based flashing — both players play simultaneously. MPF's "current player" concept is still active under the hood (framework requirement) but has no visible effect on gameplay since scoring is explicitly assigned to player 1 or 2.

## Key MPF Concepts
- **config_version=5** — current MPF config format
- **switches** — physical inputs (buttons, targets, lanes)
- **coils** — solenoids (flippers, kickers, slingshots)
- **autofire_coils** — coils that fire automatically when their paired switch activates (slingshots, pop bumpers)
- **ball_devices** — manage ball routing (troughs, plungers, locks)
- **flippers** — special devices combining a coil + switch with hold power
- **variable_player** — scoring rules: "when this event fires, add X to score"
- **show_player** — triggers shows (display animations, light sequences) on events
- **modes** — separate YAML configs with their own priority, start/stop events, and rules
