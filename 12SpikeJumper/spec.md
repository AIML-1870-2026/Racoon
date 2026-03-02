# Rhythm Jump — Game Design Specification

## Overview

**Rhythm Jump** is a side-scrolling endless runner rhythm platformer where a ball bounces across a never-ending stream of musical notes on a treble clef staff in C major. The staff scrolls forever, generating notes procedurally with increasing complexity the longer the player survives. The player times their jumps to land on notes in rhythm, building combos and chasing a high score until they miss too many and the run ends.

---

## Core Concept

The game world is a scrolling musical staff — five horizontal lines representing the treble clef in C major. Notes (whole, half, quarter, eighth) appear on and between the staff lines at positions corresponding to the pitches C, D, E, F, G, A, B, generated procedurally and scrolling in from the right. The ball bounces automatically; the player controls the *timing and height* of each bounce to land on the correct note at the correct beat. The run never ends until the player runs out of lives.

---

## Game World

### The Staff
- A standard five-line treble clef staff fills the screen horizontally and scrolls left continuously, generating notes procedurally from the right edge.
- Staff lines are spaced evenly (e.g., 40px apart), spanning from C4 (middle C, one ledger line below) up to E5.
- The clef symbol and a C major key signature (no sharps or flats) anchor the left edge.

### Note Pitches & Positions (C Major Scale)
| Note | Staff Position         |
|------|------------------------|
| C4   | Ledger line below staff|
| D4   | Just below bottom line |
| E4   | Bottom line (1st)      |
| F4   | First space            |
| G4   | Second line            |
| A4   | Second space           |
| B4   | Third line             |
| C5   | Third space            |
| D5   | Fourth line            |
| E5   | Fourth space           |

### Note Durations as Platforms
| Duration     | Platform Width | Sustain Window |
|--------------|----------------|----------------|
| Whole note   | Wide (~120px)  | 4 beats        |
| Half note    | Medium (~80px) | 2 beats        |
| Quarter note | Standard (~50px)| 1 beat        |
| Eighth note  | Narrow (~30px) | 0.5 beats      |
| Sixteenth    | Tiny (~18px)   | 0.25 beats     |

### Rests as Platforms
Half and whole rests appear on the staff as silent platforms — the ball can land and bounce on them, but no pitch plays when it does. This creates moments of musical silence in the composition and adds rhythmic variety to level design.

| Rest Type  | Visual                                      | Platform Width | Sustain Window |
|------------|---------------------------------------------|----------------|----------------|
| Half rest  | Hat-shaped block sitting on a staff line    | Medium (~80px) | 2 beats        |
| Whole rest | Hanging block suspended below a staff line  | Wide (~120px)  | 4 beats        |

Landing on a rest inscribes the correct rest symbol into the background ghost score at the appropriate staff position, just like a note hit. Missed rests (falling through) are not inscribed.

---

## Player & Ball Mechanics

### The Ball
- A small musical note-head (filled circle) serves as the player avatar.
- The ball has a natural arc — it bounces automatically in rhythm with the BPM.
- **Passive bounce:** Without input, the ball bounces in place on its current note.
- **Jump input:** Spacebar / tap triggers a jump to the *next upcoming note* in the sequence. The jump arc's height is determined by the vertical distance between the current and target note.
- **Gravity** is tuned so a quarter-note jump naturally lands on the next beat.

### Controls
| Action         | Keyboard     | Mobile        |
|----------------|--------------|---------------|
| Jump           | `Space`      | Tap screen    |
| High jump      | `Space` hold | Press & hold  |
| Pause          | `Esc`        | Two-finger tap|

### Timing Windows
| Judgment | Window        | Score Multiplier |
|----------|---------------|-----------------|
| Perfect  | ±30ms         | 2×              |
| Good     | ±80ms         | 1×              |
| Early/Late| ±150ms       | 0.5×            |
| Miss     | Outside 150ms | 0×, lose combo  |

---

## Scoring & Progression

### Score
- Base points per note hit: 10
- Multiplied by judgment quality and current combo count.
- Combo multiplier: +0.1× per consecutive hit, capped at 4×.
- Landing on a note *on the correct beat* triggers the note's audio pitch — the player literally plays the melody.

### Health
- The player starts with 5 lives (represented as music rests).
- Missing a note or falling off the staff costs one life.
- Hitting three Perfect judgments in a row restores half a life.

### Progression
There are no discrete levels. The staff generates notes procedurally and forever, with difficulty scaling automatically based on how long the player has survived (measured in beats elapsed):

**Early game (0–64 beats)** — Slow tempo (60–80 BPM), mostly quarter and half notes, stepwise melodic motion (e.g., C–D–E–F–G scale runs). Wide and medium platforms dominate.

**Mid game (65–256 beats)** — Tempo ramps to 100–120 BPM, eighth notes introduced, small melodic leaps (up to a 4th), occasional rests. Note widths tighten.

**Late game (257+ beats)** — Tempo pushes to 140–180 BPM, sixteenth notes appear, larger melodic leaps, syncopated rhythms, more frequent rests. Runs at this stage are a genuine feat of rhythm and reaction.

Tempo increases happen gradually as smooth ramps rather than hard jumps, so the player always feels in control of the escalation. There is no ceiling — the game simply keeps getting harder until the player dies.

---

## Audio

- Every note landed plays its corresponding pitch (C4–E5) using a synthesized piano timbre via Web Audio API.
- Missing a note produces a low thud sound effect.
- When the player loses their last life (death), a long low C (C2) note plays — a deep, resonant bass tone that fades out slowly over 3 seconds, signaling the end of the run. A dramatic screen shake coincides with the death note. This death note is also inscribed as a whole note at the end of the ghost score, becoming part of the player's composition.
- The overall effect is that a skilled player *performs* the melody they're creating.

---

## Visual Style

- Clean, elegant music-paper aesthetic: pure white background, black ink staff lines, traditional note glyphs. A dark mode option inverts this to a deep off-black background (#121212) with light gray staff lines and white note glyphs. The faded ghost score in dark mode renders at a slightly lighter gray (~35% opacity) to remain visible against the dark surface. All gameplay mechanics are identical in both modes. The player can toggle dark mode from the start screen at any time, and the preference is saved to local storage.
- The ball glows in a color keyed to its current pitch (e.g., C = red, D = orange … B = violet — a chromatic color wheel mapped to the scale).
- Particle effects bloom outward on Perfect hits, mimicking a musical flourish.
- Ledger lines appear dynamically as the ball moves to C4 or above E5.
- As the player progresses through a level, every note successfully bounced on is inscribed onto the background as a faded gray musical score — a ghost composition that accumulates behind the live gameplay. The scribed notes use traditional notation (note heads, stems, beams for eighth/sixteenth groups) rendered at low opacity (~20–30%) so they don't compete visually with the active staff. Bar lines separate the ghost score every 4 beats (one measure in 4/4 time). Notes that would cross a measure boundary are rendered as tied notes, preserving correct rhythmic notation. This gives the background the feeling of handwritten sheet music being slowly filled in, and lets the player see the shape of the composition they are building. Missed notes are not inscribed, so the background score reflects only the player's successful performance.
- The ball squishes and stretches to sell the bounce physics: it squashes flat (wider, shorter) on landing impact, then stretches tall (narrower, taller) at the peak of a jump. Both deformations ease back to the default circle using a brief spring interpolation (~80ms), giving the movement a satisfying, cartoony snap.
- When the ball lands on a note, the note dips downward slightly (~6–10px) on impact, then springs back up to its original position over ~120ms. This pairs with the ball's squash frame to make the landing feel physical and responsive, as if the note has real give to it.

---

## UI / HUD

- **Top-left:** Score and combo counter.
- **Top-right:** Lives (rest symbols), beats survived counter, BPM display, and a compact controls reference box displaying the key bindings at a glance. The box is semi-transparent so it doesn't obscure the staff.
- **Bottom bar:** A small scrolling "upcoming notes" preview — a miniature lookahead of the next 4 beats.
- **Center:** The main staff and ball gameplay.
- On pause, a semi-transparent overlay shows the ghost score accumulated so far.

---

## Share My Composition

At the end of a run (on death), the player is presented with a **"Share My Composition"** button. This generates a shareable artifact representing the musical score they performed — built entirely from the notes and rests they successfully landed on.

### What Gets Shared
The shared composition reflects only the player's successful hits, in the order they occurred, with their correct pitches, durations, and rest placements. Missed notes are omitted. The result is a unique, playable piece of music in C major that is different for every run.

### Share Formats
Three export options are offered:

**Image (PNG)** — A rendered sheet music image of the composition using proper musical notation (treble clef, time signature, bar lines, tied notes) on a clean white (or dark, matching the player's current mode) staff. The notation displays 6 measures per line and wraps to multiple lines as needed. Includes a small watermark ("Composed in Rhythm Jump"). Ideal for sharing to social media.

**MusicXML** — A standards-compliant MusicXML file of the composition, importable into notation software such as MuseScore, Sibelius, or Finale, so the player can view, edit, or print their piece as real sheet music.

**Share Link** — A short URL encoding the composition as a compact note sequence string (e.g., `C4q D4q E4h R2 G4q ...` — pitch, octave, duration). Clicking "Copy Share Link" opens a link preview screen showing the full notation, the generated URL, and a play/pause button to preview playback. Recipients who open the link see the composition rendered in a lightweight read-only web viewer, with a "Play" button that performs the piece using a piano timbre. No account required to view.

### Share Sheet UI
The share sheet appears as a modal overlay after the run ends (on death), showing:
- A **back arrow** in the top-left corner to return to the game over screen.
- A full notation preview of the ghost score, rendered with proper musical notation including treble clef, time signature, bar lines every 4 beats, and tied notes where needed. The preview displays 6 measures per line and wraps to multiple lines as needed. If the composition exceeds 8 lines, the preview becomes scrollable.
- **Click-to-select playback:** The player can click on any note in the preview to select it (shown with a blue border). Pressing play will start playback from the selected note rather than the beginning. Clicking the same note again deselects it.
- A **Play/Pause button** that plays back the composition using the in-game piano timbre, with each note highlighted in red as it sounds. Playback starts from the selected note, or from the beginning if no note is selected.
- Three export buttons: **Download PNG** (renders the full notation as an image), **Download MusicXML** (exports a standards-compliant file for notation software), and **Copy Share Link** (opens a link preview screen with notation display and its own play/pause control).
- A **"New Composition"** button that dismisses the share sheet and immediately begins a fresh run from the beginning, resetting score, combo, lives, and the ghost score.

---

## Technical Notes

- Target: Web (HTML5 Canvas or WebGL) and mobile (iOS/Android).
- Frame rate: 60 FPS minimum.
- Audio latency compensation: Configurable offset calibration screen on first launch.
- Collision detection: AABB between ball and note-head platform hitboxes.
- BPM sync: All note positions are calculated in *beat space* and converted to pixel positions at render time, ensuring tempo changes don't break layouts.

---

## Level Editor

The game includes a built-in level editor accessible from the start screen, allowing players to create their own custom note sequences.

### Editor Interface
- **Staff Canvas:** A clickable, horizontally scrollable staff where players can place notes by clicking at the desired pitch position. The canvas expands dynamically as notes are added, with automatic scrolling to keep the latest note visible.
- **Type Selection:** Toggle between placing notes or rests.
- **Duration Selection:** Choose note duration (whole, half, quarter, eighth).
- **Visual Feedback:** Notes appear on the staff as they're placed, with pitch labels on the left for reference. Bar lines are drawn every 4 beats.
- **Note Counter:** Displays the current count of notes and total beats.

### Editor Controls
- **Preview:** Play back the custom composition to hear how it sounds before playing. Notes are highlighted in red as they play, and the canvas auto-scrolls to keep the current note visible.
- **Play Level:** Start the game using the custom note sequence (loops when complete).
- **Undo:** Remove the last placed note.
- **Clear:** Remove all notes and start fresh.

### Custom Level Gameplay
When playing a custom level, the procedural generation is replaced with the player's note sequence. The sequence loops continuously, allowing the player to practice their composition indefinitely. All other gameplay mechanics (scoring, combos, lives) remain unchanged.

---

## Out of Scope (v1.0)

- Sharps, flats, or keys other than C major.
- Multiplayer.
- Note recording / microphone input.
- Bass clef staff.
- Saving/loading custom levels to files.
