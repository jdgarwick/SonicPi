# Strudel Notes

---

## Orbits

An orbit is a **separate audio bus** (think: a dedicated channel on a mixing board).
By default, all patterns share **orbit 1**. Each orbit has its own independent reverb
and delay — patterns on the same orbit share those effects, which can cause them to
interfere with each other.

### Why It Matters

If two patterns share the same orbit but have different reverb settings, they will
conflict and produce unpredictable results:

```js
// BAD — both on orbit 1, reverb settings fight each other
$: s("triangle*4").room(1).roomsize(10)
$: s("bd*4").room(0.01).roomsize(0.01)

// GOOD — separated onto their own orbits
$: s("triangle*4").room(1).roomsize(10).orbit(2)
$: s("bd*4").room(0.01).roomsize(0.01)
```

### Other Uses for Orbits

- **Sidechain ducking** — target a specific orbit with `duckorbit`
- **DAW routing** — with multi-channel output enabled, each orbit maps to its own stereo out

### Example

```js
note("D3 f#2 C# A3".sub(12))
  .sound("supersaw")
  .slow(4)
  .orbit(4) // sends this pattern to its own dedicated bus
```

---

## Building Melodies That Fit

### The Core Idea: Scales
Your chords determine your **scale** — a specific set of notes that all sound good together.
Any melody that sticks to those notes will fit. For example, chords built from A minor
(Am, Dm7, Em, Em7) all belong to the **A minor scale: A B C D E F G**.

### Let Strudel Keep You In Key
Instead of memorizing which notes belong to a scale, use `n()` + `.scale()`.
Every number maps to a note in the scale — you cannot play a wrong note this way:

```js
n("0 1 2 3 4 5 6 7").scale("A:minor")
// 0=A, 1=B, 2=C, 3=D, 4=E, 5=F, 6=G, 7=A(octave up)
```

Swap numbers around freely — it will always sound in key.

### Four Rules for Natural-Sounding Melodies

**1. Move in steps, not jumps**
Melodies that move up or down one note at a time feel natural. Big jumps feel jagged.
```js
n("0 1 2 3 ~ 2 1 ~").scale("A:minor") // stepwise — natural
n("0 4 7 3 ~ 6 1 ~").scale("A:minor") // jumpy — harder to follow
```

**2. Repeat phrases before changing them**
Listeners need to hear something twice before they remember it.
```js
n("0 2 4 ~ 0 2 4 ~ 5 4 2 0").scale("A:minor")
// same phrase twice, different ending
```

**3. Land on strong notes**
Notes 0, 2, and 4 (A, C, E in A minor) are the safest landing spots —
they make up the Am chord itself, so ending a phrase on them feels resolved.

**4. Leave space with rests**
Silence makes the notes that do play hit harder.
```js
n("0 ~ 4 2 ~ ~ 0 ~").scale("A:minor") // spacious, memorable
n("0 1 2 3 4 5 6 7").scale("A:minor") // busy, hard to latch onto
```

### Practical Starting Point
Start with 3-4 numbers, find a phrase you like, then expand from there.
Swap one number at a time and re-evaluate. If something sounds off,
it's usually a **rhythm issue** rather than a wrong note — the scale keeps you safe.

```js
$MELODY:
n("0 ~ 4 2 ~ ~ 0 ~")
  .scale("A:minor")
  .sound("supersaw")
  .slow(2)
  .orbit(11)
._punchcard()
```

---

## slider()

`slider()` creates an **interactive slider widget** directly in the REPL that you can drag
in real time while the music plays, instead of hardcoding a fixed number.

```js
.acidenv(slider(0.5, 0, 1, 0.01))
//              ↑    ↑  ↑  ↑
//            start min max step
```

| Parameter | What it means |
|---|---|
| **start** | Where the slider begins when the code first runs |
| **min** | How far left you can drag — the lowest possible value |
| **max** | How far right you can drag — the highest possible value |
| **step** | How precisely it moves (0.01 = 100 possible steps between min and max) |

A smaller step like `0.001` gives finer control. A bigger step like `0.1` makes it snap
between fewer positions — useful for making large changes quickly.

---

## acidenv() — Custom Function (warm.strudel.cc only)

`acidenv()` is **not a built-in Strudel function**. It is a custom function created by
live-coder **Switch Angel**, pre-loaded in the warm.strudel.cc version of the REPL.
It will not work on standard strudel.cc.

Under the hood it applies a TB-303 style filter envelope — the filter that defines
classic acid house music. It controls how aggressively the filter sweeps open and
closed each time a note hits, like an automatic "wah" on every note.

```js
.acidenv(slider(0.78))
// low value (0.1)  → filter barely opens, dark and muffled
// mid value (0.5)  → subtle movement, adds life without drama
// high value (0.9) → filter rips open aggressively
```

Pair it with `slider()` to dial in the intensity live:
```js
note("A2 D2 G2 E2".sub(12))
  .sound("supersaw")
  .acidenv(slider(0.5, 0, 1, 0.01))
```

---

## Custom Filter Functions (warm.strudel.cc only)

These are additional custom functions from Switch Angel's scripts available in warm.strudel.cc.
None of these appear in the standard reference panel.

### rlpf(0–1) — Normalized Lowpass Filter
Instead of thinking in Hz, pass a value between 0 and 1.
```js
note("A2 D2").sound("supersaw").rlpf(slider(0.5))
// 0 = very dark/closed, 1 = fully open/bright
```

### rhpf(0–1) — Normalized Highpass Filter
Same idea but cuts the low end instead of the high end.
```js
sound("cabasa:1!16").rhpf(slider(0.3))
// 0 = nothing cut, 1 = cuts almost everything below
```

### acid() — One-word Acid Preset
Bundles supersaw + filter + envelope + resonance into a single call. No arguments needed.
```js
note("A2 D2 G2 E2").acid()
```

### dly(0–1) — Smart Delay
A delay that automatically increases feedback as you raise the amount.
```js
note("A4 ~ C5 ~").sound("supersaw").dly(slider(0.4))
// 0 = no delay, 1 = heavy echoing delay with feedback
```

### trancegate(density, seed, length) — Rhythmic Gate
Randomly cuts the pattern in and out rhythmically, like a stutter effect.
```js
note("A2 D2 G2").sound("supersaw").trancegate(0.6, 1, 4)
```

> **Tip:** `rlpf` and `rhpf` are the most immediately practical — they are much easier
> to use with a slider than `lpf`/`hpf` since you do not have to think in Hz.

---

## Sidechaining (Ducking)

### What is Sidechaining?
Sidechaining is the effect where every time the kick drum hits, everything else dips in
volume for a split second then comes back up. That pumping, breathing quality in dance
music — that's sidechaining. It makes the kick cut through everything else cleanly.

The kick is the **trigger**. Every other pattern is the **target**. When the kick fires,
the targets lower their volume, then recover.

### How It Works in Strudel
The kick pattern carries all three ducking parameters:

```js
$KICK:
sound("bd:1!4").orbit(4)
  .duck("3:4:5:6")   // which orbits to duck
  .duckdepth(.1)     // how much the volume drops
  .duckattack(0.16)  // how fast the volume recovers
```

### Breaking Down Each Parameter

**`.duck("3:4:5:6")`**
Targets the orbits that will get quieter when the kick hits.
Each number is an orbit number — whatever patterns live on those orbits will duck.
```js
.duck("3:4:5:6")  // ducks orbits 3, 4, 5, and 6
.duck("3")        // would only duck orbit 3
```

**`.duckdepth(0–1)`**
Controls how much the volume drops when the kick hits.
```js
.duckdepth(0.1)  // subtle dip, gentle pumping feel
.duckdepth(0.5)  // noticeable drop
.duckdepth(1.0)  // everything goes almost completely silent on the kick
```

**`.duckattack(seconds)`**
Controls how quickly the volume recovers after the kick hits.
Despite the name, this is about the recovery speed back up — not how fast it ducks down
(that part is instant).
```js
.duckattack(0.01)  // snaps back almost instantly
.duckattack(0.16)  // smooth, natural pump (~1/6th of a second recovery)
.duckattack(0.5)   // slow recovery, very dramatic pumping effect
```

> **Tip:** To hear the effect clearly, try cranking `.duckdepth` to `0.8` and
> `.duckattack` to `0.3` — once you hear the exaggerated version, the subtler
> settings will make a lot more sense.

---

## stack()

`stack()` plays multiple patterns **simultaneously**, letting you combine them into one
block and apply shared settings to all of them at once instead of repeating yourself.

### Basic Usage
```js
stack(
  n("0 1 2 3").scale("A:minor"),
  n("0 1 2 3").scale("A:minor").rev()
)
  .sound("bytebeat")
  .release(0.1)
// both patterns play at the same time, both get the same sound and release
```

### Practical Example — Combining Two Identical Patterns
Instead of this:
```js
$MELODY:
n("0 1 2 4 4 3 1 1").scale("A:minor")
  .sound("bytebeat").release(0.1)

$MELODY2:
n("0 1 2 4 4 3 1 1").scale("A:minor").rev()
  .sound("bytebeat").release(0.1)
```

Do this:
```js
$MELODY:
stack(
  n("0 1 2 4 4 3 1 1").scale("A:minor"),
  n("0 1 2 4 4 3 1 1").scale("A:minor").rev()
)
  .sound("bytebeat")
  .release(0.1)
```

### Store Repeated Patterns in a Variable
If both patterns share the same note sequence, store it in a `const` so you only
have to change it in one place:

```js
const mel = "0 1 2 4 4 3 1 1"

$MELODY:
stack(
  n(mel).scale("A:minor"),
  n(mel).scale("A:minor").rev()
)
  .sound("bytebeat")
  .fast(2)
  .release(slider(0.131))
  .attack(slider(0.156))
  .acidenv(slider(0.596))
  .room(slider(0.444))
  .orbit(8)
._punchcard({labels:1})
```

> **Note:** Patterns inside a `stack()` share the same orbit, so they also share
> the same reverb and delay. If you want two patterns to have different effects,
> keep them as separate `$` patterns on different orbits.

---

## Modes

### What is a Mode?
A mode is what happens when you take the same set of notes but start from a different
one. Same ingredients, different starting point, completely different feeling.

You are already using a mode — **A minor is A Aeolian**. That's your current track.

### The 7 Modes — Dark to Bright

Think of modes on a spectrum:

```
DARK ←————————————————————————→ BRIGHT
Locrian  Phrygian  Aeolian  Dorian  Mixolydian  Ionian  Lydian
```

| Mode | Vibe | Famous example |
|---|---|---|
| **Locrian** | Very tense, unstable — rarely used | Extreme metal |
| **Phrygian** | Dark, exotic, Spanish/metal feel | Flamenco, Radiohead |
| **Aeolian** | Standard minor — dark, emotional ← you are here | Most pop/rock ballads |
| **Dorian** | Minor but with a hopeful lift | Daft Punk, old jazz |
| **Mixolydian** | Major but slightly bluesy/unresolved | Beatles, classic rock |
| **Ionian** | Bright, happy — this is just major | Happy Birthday |
| **Lydian** | Dreamy, floaty, slightly magical | Film scores, Simpsons theme |

### How to Use Modes in Strudel
Strudel has all 7 modes built in. Just swap the scale name — the numbers stay
the same, you don't have to relearn anything:

```js
n("0 2 4 3").scale("A:minor")       // Aeolian — dark (your current scale)
n("0 2 4 3").scale("A:phrygian")    // very dark, exotic
n("0 2 4 3").scale("A:dorian")      // minor but with a hopeful lift
n("0 2 4 3").scale("A:lydian")      // dreamy, floaty
n("0 2 4 3").scale("A:mixolydian")  // major but unresolved/bluesy
n("0 2 4 3").scale("A:ionian")      // straight up major/bright
```

### Best First Swap for Your Track
**Dorian** is the most practical mode to try from where you are. It keeps the minor
feel but adds a slightly more interesting, less resolved quality — used constantly
in electronic music:

```js
n("0 1 2 4 4 3 1 1").scale("A:dorian")
// compare back to A:minor and hear the subtle but real difference
```

> **Tip:** You only need to swap the scale on your melody and arp. Your chord
> pattern uses actual chord names so it stays the same regardless of mode.

---

## 🗺️ Music Theory Reference — For Any Future Song

This section is a reference you can come back to at the start of any creative
process to figure out what key you are in, what chords fit, and what modes
your melody can safely use.

---

### STEP 1 — Figure Out What Key You're In From Your Chords

Look at your chord progression and find it in the table below.
The key is in the left column.

| Key | The chords that naturally live in it |
|---|---|
| **C major** | C, Dm, Em, F, G, Am, Bdim |
| **G major** | G, Am, Bm, C, D, Em, F#dim |
| **D major** | D, Em, F#m, G, A, Bm, C#dim |
| **A major** | A, Bm, C#m, D, E, F#m, G#dim |
| **E major** | E, F#m, G#m, A, B, C#m, D#dim |
| **F major** | F, Gm, Am, Bb, C, Dm, Edim |
| **A minor** | Am, Bdim, C, Dm, Em, F, G |
| **E minor** | Em, F#dim, G, Am, Bm, C, D |
| **D minor** | Dm, Edim, F, Gm, Am, Bb, C |
| **G minor** | Gm, Adim, Bb, Cm, Dm, Eb, F |
| **B minor** | Bm, C#dim, D, Em, F#m, G, A |

> **How to use:** Look at your chords. Find the row where all (or most) of them appear.
> That's your key. It's okay if one chord doesn't appear — that's called a
> "borrowed chord" and is intentional in a lot of music.

**Example:** Your current progression is Dm F C G.
All four appear in the **D minor** row → you are in **D minor**.

---

### STEP 2 — What is the Root Note of Your Key?

The root note is just the letter name of your key.

- D minor → root is **D**
- A minor → root is **A**
- G major → root is **G**

This is what you put before the colon in Strudel's scale function:
```js
.scale("D:minor")   // D is the root
.scale("A:dorian")  // A is the root
.scale("G:major")   // G is the root
```

---

### STEP 3 — What Modes Can Your Melody Use?

Once you know your key and root, use this chart to pick a mode.
All modes in the same row share the same notes — they are all compatible
with each other and with the chords in that key.

| Key | Modes that all share the same notes (safe to mix) |
|---|---|
| **C major / A minor** | `C:ionian` `D:dorian` `E:phrygian` `F:lydian` `G:mixolydian` `A:minor` `B:locrian` |
| **G major / E minor** | `G:ionian` `A:dorian` `B:phrygian` `C:lydian` `D:mixolydian` `E:minor` `F#:locrian` |
| **D major / B minor** | `D:ionian` `E:dorian` `F#:phrygian` `G:lydian` `A:mixolydian` `B:minor` `C#:locrian` |
| **F major / D minor** | `F:ionian` `G:dorian` `A:phrygian` `Bb:lydian` `C:mixolydian` `D:minor` `E:locrian` |
| **Bb major / G minor** | `Bb:ionian` `C:dorian` `D:phrygian` `Eb:lydian` `F:mixolydian` `G:minor` `A:locrian` |
| **A major / F# minor** | `A:ionian` `B:dorian` `C#:phrygian` `D:lydian` `E:mixolydian` `F#:minor` `G#:locrian` |

> **How to use:** Find your key in the left column.
> Every scale listed in that row is safe to use for your melody/arp.
> Your chords will not clash with any of them.

**Example:** You are in D minor → use the **F major / D minor** row.
Safe melody scales include `D:minor`, `D:dorian`, `A:phrygian`, `G:dorian`, `F:ionian` etc.

---

### STEP 4 — Choosing a Mode by Mood

Once you know which modes are safe, pick one based on the feeling you want:

| If you want... | Use this mode | Strudel example |
|---|---|---|
| Standard dark/emotional | `root:minor` (Aeolian) | `scale("D:minor")` |
| Dark but with a lift | `root:dorian` | `scale("D:dorian")` |
| Very dark, exotic, tense | `root:phrygian` | `scale("D:phrygian")` |
| Bright, standard happy | `root:ionian` (major) | `scale("F:ionian")` |
| Bluesy, unresolved | `root:mixolydian` | `scale("C:mixolydian")` |
| Dreamy, floating | `root:lydian` | `scale("Bb:lydian")` |
| Maximum tension (use carefully) | `root:locrian` | `scale("E:locrian")` |

---

### STEP 5 — Quick Strudel Scale Reference

All scale names usable in `.scale()` in Strudel:

```js
// MINOR FAMILY (darker)
.scale("X:minor")       // standard minor — most common dark sound
.scale("X:dorian")      // minor with a lift — common in electronic music
.scale("X:phrygian")    // dark and exotic
.scale("X:locrian")     // very tense, unstable

// MAJOR FAMILY (brighter)
.scale("X:major")       // standard bright/happy
.scale("X:ionian")      // same as major
.scale("X:mixolydian")  // major but bluesy
.scale("X:lydian")      // dreamy, magical

// PENTATONIC (5 notes — very safe, hard to go wrong)
.scale("X:minor:pentatonic")   // 5-note minor — great for melodies
.scale("X:major:pentatonic")   // 5-note major — very bright and safe
```

> Replace `X` with your root note e.g. `D:minor`, `A:dorian`, `G:major:pentatonic`

---

### Quick Reference — Your Current Song

| Element | Value |
|---|---|
| **Chord progression** | Dm F C G |
| **Key** | D minor |
| **Root note** | D |
| **Safe melody modes** | `D:minor`, `D:dorian`, `D:phrygian`, `A:phrygian`, `G:dorian`, `C:mixolydian`, `F:ionian` |
| **Current melody scale** | `A:dorian` + `A:minor` (works — closely related to D minor) |
| **Recommended upgrade** | `D:dorian` or `D:minor` to lock in more tightly |

---

## Repeating Arp Lines

To repeat a line of notes multiple times before switching to the next line,
combine `< >` (alternates each cycle), `[ ]` (groups notes), and `!` (repeats).

```js
n("<[1 2 3 5]!4 [0 3 5 2]!4>").fast(4)
// [1 2 3 5] = one line of 4 notes
// !4        = stay on this line for 4 full cycles before switching
// < >       = alternates between lines each cycle
// .fast(4)  = plays the 4 notes 4 times = 16 hits per cycle
```

### Result
```
Cycles 1–4:   1 2 3 5  ×4 per cycle  (16 hits)
Cycles 5–8:   0 3 5 2  ×4 per cycle  (16 hits)
...then loops
```

### Full Example — 4 Lines, Each Playing 4 Cycles
```js
$ARP:
n("<[1 2 3 5]!4 [0 3 5 2]!4 [2 5 3 6]!4 [1 4 2 5]!4>").chord("Dm F C G")
  .voicing().sub(12)
  .sound("sawtooth")
  .fast(4)
  .acidenv(slider(0.683))
  .orbit(7)
._punchcard()
```

> **Tip:** Change `!4` to `!8` to stay on a line for 8 cycles instead of 4.
> Add or remove lines freely inside the `< >` — just keep `!4` on each one.

---

## Controlling When Things Hit

### .beat() — Exact Step Placement
Specify exactly which steps something hits on out of a total number of steps.
The grid starts at **0**:

```js
sound("bd:1").beat("0, 4, 8, 12", 16)   // four on the floor
sound("bd:1").beat("0, 3, 8, 14", 16)   // syncopated, off-kilter
sound("bd:1").beat("0, 6, 10", 16)      // sparse, only 3 hits
sound("bd:1").beat("0, 2, 4, 6, 8", 16) // busy, double-time feel
//                  ↑ steps it hits   ↑ total steps in the grid (0–15)
```

Use `.beat()` when you have a specific pattern in mind and want exact control.

---

### .struct() — Euclidean Rhythms (Sonic Pi's spread equivalent)
Strudel's equivalent of Sonic Pi's `spread(hits, total)`. Automatically spaces
hits as evenly as possible across the total number of steps:

```js
sound("bd:1").struct("x(3,8)")   // 3 hits spread evenly across 8 steps
sound("bd:1").struct("x(5,8)")   // 5 hits across 8 — classic clave rhythm
sound("bd:1").struct("x(4,16)")  // 4 hits across 16
//                     ↑ ↑
//               hits  total steps
```

Add a third number to **rotate** (shift) the pattern forward:
```js
sound("bd:1").struct("x(3,8,2)")  // 3 hits across 8, shifted 2 steps
```

Use `.struct()` when you want mathematically even, organic-feeling spacing.

### Combining Both
```js
$KICK:
sound("bd:1").struct("x(4,16)").orbit(4)  // euclidean kick
  .duck("2:5:6:7:8")

$SNARE:
sound("sd:8").beat("4, 12", 16).orbit(5)  // exact snare placement
```

---

## .voicing()

`.voicing()` takes a chord name and spreads it into **multiple notes played simultaneously**,
arranged in a musically sensible way across the keyboard. Without it, `.chord()` only
plays the root note of the chord — a single note, not a full chord.

```js
chord("Dm")           // plays just D — root note only
chord("Dm").voicing() // plays D + F + A spread across the keyboard
```

### What "Voicing" Means
Voicing is how the notes of a chord are arranged vertically. The same chord can sound
completely different depending on which notes are on top, which are on the bottom,
and how spread out they are. `.voicing()` handles all of this automatically using
a built-in dictionary of common voicings.

### Why .sub(12) Often Follows It
`.voicing()` tends to place chords in the mid-to-upper range by default.
`.sub(12)` drops everything down one octave (12 semitones) to sit more
comfortably in the mix:
```js
chord("Dm F C G").voicing().sub(12)
```

### Do You Always Need .voicing() With .chord()?

| Code | What you hear |
|---|---|
| `chord("Dm")` | Just the root note D — basically a bass note |
| `chord("Dm").voicing()` | Full chord — D + F + A spread across the keyboard |
| `n("0 2 4").chord("Dm").voicing()` | Notes plucked from the voicing one at a time — arpeggio |
| `n("0 2 4").chord("Dm")` | ❌ Broken — `n()` has nothing to index into without `.voicing()` |

> **Rule:** Using `.chord()` alone → voicing is optional, you get root notes.
> Using `n()` + `.chord()` → **voicing is required**, otherwise `n()` has
> no notes to select from and nothing will play.

---

## Composition & Arrangement

By default all patterns play simultaneously forever. Arrangement is about
controlling **when things enter and exit** over time.

---

### Method 1 — Live Mute Toggle (best for live performance)
Add `_` before `$` to silence a pattern instantly. Remove it to bring it back.
Hit `Ctrl+Enter` and the change takes effect on the next cycle:

```js
$KICK:        // playing
_$VIOLIN:     // muted — silent
_$MELODY:     // muted — silent
```

This is how most live coders perform. Fast, flexible, no planning required.

---

### Method 2 — mask() (best for a composed structure)
`mask()` tells a pattern when to play and when to be silent across cycles.
`1` = play, `0` = silent:

```js
.mask("<0!8 1>")   // silent for 8 cycles, then plays forever
.mask("<1>")       // always plays
.mask("<0>")       // always silent (same as muting with _)
.mask("<0!16 1!8 0!8>")  // silent 16, plays 8, silent 8 — then loops
```

| Syntax | Meaning |
|---|---|
| `"<0!8 1>"` | Silent 8 cycles, plays forever after |
| `"<0!16 1!8 0!8>"` | Silent 16, plays 8, silent 8 — loops |
| `"<1>"` | Always plays |
| `"<0>"` | Always silent |

> ⚠️ **Important:** The mask cycle counter resets every time you hit `Ctrl+Enter`.
> For composed arrangements, let the track run from the top without re-evaluating.
> For live work, use the `_$NAME:` mute toggle instead.

---

### Suggested Arrangement Structure

Build your track in layers — bring elements in one at a time so each addition
feels like a moment. Here's a template based on the track built in these notes:

```
CYCLES      1–4      5–8      9–16     17–24    25–32    33–40
BASS        ████     ████     ████     ████     ████     ████
HIHAT       ░░░░     ████     ████     ████     ████     ████
KICK        ░░░░     ░░░░     ████     ████     ████     ████
SNARE       ░░░░     ░░░░     ████     ████     ████     ████
PAD         ░░░░     ░░░░     ░░░░     ████     ████     ████
ARP         ░░░░     ░░░░     ░░░░     ████     ████     ████
VIOLIN      ░░░░     ░░░░     ░░░░     ░░░░     ████     ████
MELODY      ░░░░     ░░░░     ░░░░     ░░░░     ░░░░     ████
```

Translated into mask() values:
```js
.mask("<1>")      // BASS    — always playing, foundation
.mask("<0!4 1>")  // HIHAT   — enters cycle 5
.mask("<0!8 1>")  // KICK    — enters cycle 9
.mask("<0!8 1>")  // SNARE   — enters with kick
.mask("<0!16 1>") // PAD     — enters cycle 17
.mask("<0!16 1>") // ARP     — enters with pad
.mask("<0!24 1>") // VIOLIN  — enters cycle 25
.mask("<0!32 1>") // MELODY  — enters last, the payoff
```

> **General rule:** Start with bass and rhythm, add harmony in the middle,
> save melody for last — each new layer should feel like a reward.

---

## arrange() — Timeline-Based Composition

`arrange()` plays patterns **in sequence** as a defined timeline — unlike `mask()`
which runs everything simultaneously and gates it on/off. Think of it as writing
out the structure of your song from start to finish.

### Syntax
```js
arrange(
  [cycles, pattern],
  [cycles, pattern],
  [cycles, pattern]
)
```

Each pair is `[how many cycles, what plays]`. Strudel plays them one after another.

### Full Example — Guitar Ambient Track
```js
let guitarArp = n(`<[0 1 4 2 6 4 2 4][−2 0 2 0 4 2 0 2]>`).scale("<F:Major>")
  .s("gm_acoustic_guitar_nylon").delay(slider(0.2656, 0.1, 1)).delayfeedback(.8)

let guitarEcho = n(`<[6 7][6 4] 2 1!12[−] >`).scale("<F:Major>")
  .s("gm_acoustic_guitar_nylon").room(0.9).size(8).hpf(2000).lpf(3000)
  .echo(16, 1/8, .7)

arrange(
  [4,  pad],                                              // intro — pad only
  [2,  stack(pad, effect)],                               // slight build
  [2,  stack(pad, effect, guitarArp.gain("<0 .05 .08 .1 .12>[.14 .16 .18 .2]>"))],  // guitar whispers in
  [4,  stack(pad, effect, guitarArp.gain("<[.45 .5 .55 .6 .65][.7 .75 .8 .85][.9 .95 1 1][1]>"))],  // guitar crescendos
  [4,  stack(pad, effect, guitarArp, guitarEcho)],        // echo layer enters
  [16, stack(pad, effect, guitarArp, guitarEcho, percussion)] // full arrangement
).scope()
```

### The Arc This Creates
```
Cycles    What plays                              Feel
1–4       pad only                               sparse intro
5–6       pad + effect                           slight build
7–8       pad + effect + guitar (whisper)        guitar teased
9–12      pad + effect + guitar (crescendo)      guitar arrives
13–16     pad + effect + guitar + echo           full texture
17–32     everything + percussion               main section
```

### Key Techniques Used

**Fading a pattern in using `.gain()` with increasing values:**
```js
guitarArp.gain("<0 .05 .08 .1 .12>")   // barely audible — sneaks in
guitarArp.gain("<.45 .5 .6 .8 1>")     // crescendos to full volume
```
This lets you introduce the same pattern gradually across multiple arrange steps
rather than having it appear at full volume.

**Defining patterns as variables before arrange():**
```js
let pad = chord("F C Am G").voicing().sound("gm_string_ensemble_1").slow(4)
let effect = s("hh*8").gain(0.3)
// then use the variable names inside arrange() — much cleaner
```

### arrange() vs mask()

| | `arrange()` | `mask()` |
|---|---|---|
| How it works | Plays sections in sequence | All patterns run, gated on/off |
| Best for | Composed song structure | Live performance muting |
| Can modify pattern per section | ✅ (e.g. different gain) | ❌ |
| Resets on Ctrl+Enter | ✅ starts from beginning | ✅ |
| Most like | A DAW timeline | A mixer with mute buttons |

> **Tip:** Define your patterns as `let` variables before the `arrange()` block.
> This keeps the arrange block clean and readable, and lets you reuse the same
> pattern across multiple sections with different modifications.

---

## Envelopes & Timing — How They Interact

### The Pianist Analogy
Think of every parameter as instructions to a pianist playing chords:

| Parameter | Analogy | What it actually controls |
|---|---|---|
| `.slow(8)` | How slowly they move from one chord to the next | How fast the pattern cycles through — rhythm |
| `.attack(0.3)` | How gradually they press the keys down | How long it takes for the note to reach full volume |
| `.decay(0.2)` | How quickly they ease off after the initial press | How fast the volume drops after the peak |
| `.sustain(0.6)` | How long they hold each chord before lifting their hands | The volume level held after attack and decay (0–1) |
| `.release(0.5)` | How slowly they lift their fingers after letting go | How long the note fades out after it ends |

None of these overpower each other — they are completely independent and work together.

---

### .slow() vs .sustain() — The Key Distinction

```js
chord("F Am F C").voicing()
  .slow(8)       // F→Am→F→C takes 8 full cycles to complete
  .sustain(0.6)  // each chord rings for 60% of its time slot before releasing
```

You can combine them in any way:

```js
.slow(8).sustain(0.1)  // moves slowly between chords, cuts each one off quickly
.slow(1).sustain(0.9)  // moves quickly between chords, holds each one long
```

---

### The ADSR Envelope — Full Picture

ADSR describes the full shape of a note from start to finish:

```
Volume
  │        ╱╲
  │       ╱  ╲_______
  │      ╱           ╲
  │     ╱             ╲
  └────────────────────── Time
       ↑  ↑  ↑        ↑
    attack decay sustain release
```

```js
.adsr("0.1:0.2:0.6:0.4")
//      ↑   ↑   ↑   ↑
//   attack decay sustain release
//   (secs)(secs)(0-1) (secs)
```

| Stage | What happens | Analogy |
|---|---|---|
| **Attack** | Volume rises from 0 to peak | Pressing the key down |
| **Decay** | Volume falls from peak to sustain level | Easing off after the initial press |
| **Sustain** | Volume holds at this level while note is held | Holding the chord |
| **Release** | Volume fades to 0 after note ends | Lifting the fingers slowly |

### Common ADSR Shapes

```js
.adsr("0.01:0.1:0.1:0.1")  // plucky — sharp attack, quick decay
.adsr("0.01:0.2:0.4:0.3")  // natural piano feel
.adsr("0.3:0.5:0.7:0.8")   // pad — slow fade in, long sustain
.adsr("0.01:0.05:0:0.05")  // staccato — very clipped
.adsr("0.1:0.3:0.5:0.4")   // soft entry, gentle sustain
```

> **Rule:** `.slow()` controls rhythm — when notes happen.
> **ADSR** controls shape — what each note sounds like when it does happen.
> They never conflict.

---

## acidenv() — Under the Hood

`acidenv()` is a shorthand that expands into 5 standard Strudel filter parameters.
When you write `.acidenv(0.5)` it silently becomes:

```js
.lpf(100)        // filter starts nearly closed — 100Hz cuts almost everything
.lpenv(0.5 * 9) // = lpenv(4.5) — how far the filter opens on each note hit
.lps(0.2)        // filter sustain — stays 20% open after the decay
.lpd(0.12)       // filter decay — closes back down in 0.12 seconds (very fast)
.lpq(2)          // resonance — adds the nasal "wah" peak at the cutoff point
```

### What Each Part Does

**`.lpf(100)`** — The filter starts nearly closed. 100Hz is deep bass territory —
almost everything is cut before the envelope opens it up.

**`.lpenv(x * 9)`** — The main control. How far the filter sweeps open when a
note hits. Your slider value is multiplied by 9:
```
slider 0.1 → lpenv(0.9)  barely opens — dark and subtle
slider 0.5 → lpenv(4.5)  opens halfway — classic acid
slider 1.0 → lpenv(9.0)  rips fully open — aggressive
```

**`.lps(0.2)`** — After the sweep the filter settles at 20% open rather than
snapping fully shut. Gives a slight tail to the sound.

**`.lpd(0.12)`** — The filter sweep lasts only 0.12 seconds — very fast, which
gives acid its characteristic snappy, percussive quality.

**`.lpq(2)`** — Resonance at the cutoff point. This creates the slightly
aggressive, nasal wah quality that defines the 303 sound.

### The Shape of the Sweep

Every time a note triggers, the filter slams open then quickly closes back.
That repeating sweep on every note hit is the acid sound:

```
Filter
openness
  │    ╱╲
  │   ╱  ╲___
  │  ╱       ╲
  │ ╱          ╲___
  └──────────────── Time
    ↑  ↑    ↑
  note lpd  lps
  hit  decay sustain
```

### Why the Slider Feels So Dramatic

Because `x` is multiplied by 9 before being passed to `lpenv`, small slider
movements create large filter changes. Moving from `0.3` to `0.6` changes
`lpenv` from `2.7` to `5.4` — nearly double the sweep depth. This is
intentional — the TB-303 filter is famously sensitive and dramatic.

### Standard Strudel Equivalent (for Obsidian / standard strudel.cc)

```js
// replace .acidenv(0.5) with:
.lpf(100)
.lpenv(4.5)   // your slider value × 9
.lps(0.2)
.lpd(0.12)
.lpq(2)
```

---

## Wavetables

Wavetables are synthesized tones built from pre-recorded waveform shapes.
In Strudel they are available in the sounds panel under the **wavetables** tab
and are prefixed with `wt_`.

### Using Wavetables as a Sound Source
```js
.sound("wt_comp5")   // wavetable synth tone
.sound("wt_digital") // different wavetable character
```

### Kick Layering With Wavetables
A classic production technique — stack a wavetable tone underneath a drum sample
using a comma. The sample provides the attack and character, the wavetable adds
sub-bass weight and body:

```js
sound("wt_comp5,rolandcompurhythm78_bd!4")
// wt_comp5              = sustained low-frequency tone — adds weight and rumble
// rolandcompurhythm78_bd = the actual kick sample — snap, crack, transient
```

The comma stacks both sounds simultaneously on every hit. This is visible in
`.scope()` as a smooth sine-like wave ringing out beneath the kick transient.

### Tune the Wavetable to Your Key
For the wavetable layer to sit well in the mix, tune it to your root note:
```js
sound("wt_comp5,rolandcompurhythm78_bd!4")
  .note("D1")  // tunes the wavetable to D — matches D minor key
  .gain(1.2)
```

### Wavetable Parameters
```js
.wt(0.5)      // which point in the wavetable to start from (0–1)
.wtrate(2)    // how fast it scans through the wavetable
.wtdepth(0.5) // how much of the wavetable scan is applied
```

> **Tip:** Kick layering with a wavetable is most effective when the wavetable
> tone is tuned to the root note of your key. This way the low-end energy
> of the kick reinforces the harmonic foundation of the track.

---

## warm.strudel.cc Only Functions

These functions are from Switch Angel's custom scripts and only work in
**warm.strudel.cc** — they will not work on standard strudel.cc or the
Obsidian plugin.

### Full List

| Function | What it does | Standard equivalent |
|---|---|---|
| `acidenv(0-1)` | TB-303 filter envelope | `.lpf(100).lpenv(4).lpd(.12).lps(.2).lpq(2)` |
| `rlpf(0-1)` | Normalized lowpass (0–1 scale) | `.lpf()` in Hz |
| `rhpf(0-1)` | Normalized highpass (0–1 scale) | `.hpf()` in Hz |
| `acid()` | One-word acid preset | `.s("supersaw").lpf(100).lpenv(2).lpq(12)` |
| `dly(0-1)` | Smart delay with auto-feedback | `.delay().delayfeedback()` separately |
| `trancegate()` | Random rhythmic gating/stutter | — |
| `tgate()` | Better trancegate with presets | — |
| `solo(...orbits)` | Solo specific orbits by number | — |
| `mute(...orbits)` | Mute specific orbits by number | — |
| `grab("e:f#:c")` | Quantize notes to specific values | — |
| `swap("bd","lt")` | Swap one sample value for another | — |
| `glide(time)` | Polyphonic portamento/glide | — |
| `trancearp()` | Arpeggiator with direction presets | — |
| `track()` | Tracker-style visual arrangement | `mask()` per pattern |
| `spinor(alive, dead)` | Pre-built spinning ambient texture | — |
| `noisehat()` | Pre-built noise hi-hat sound | — |
| `zap()` | Pre-built blip/zap sound | — |

### Standard Replacements for Obsidian

For use in the Obsidian plugin or standard strudel.cc, replace warm-only
functions like this:

```js
// acidenv(0.5) becomes:
.lpf(100).lpenv(4).lpd(0.12).lps(0.2).lpq(2)

// rlpf(0.5) becomes:
.lpf(2000)  // use Hz values — 500=dark, 2000=mid, 8000=bright

// rhpf(0.3) becomes:
.hpf(500)
```

---

## track() — Visual Arrangement (warm.strudel.cc only)

A tracker-style arranger — define your patterns first, then write a visual
sequence of which ones play when using `1` (play) and `0` (silent):

```js
const bd   = sound("bd:4!4")
const sd   = sound("sd:2").beat("4,12", 16)
const hat  = sound("hh!16")
const bass = note("D2 F2 C2 G2").sound("supersaw").slow(4)

$: track(
  [bd, sd, hat, bass],      // patterns in order
  4, "1---0---1---0---",    // 4 cycles: bd+hat play, sd+bass silent
  4, "1---1---0---1---",    // 4 cycles: bd+sd+bass play, hat silent
  8, "1---1---1---1---"     // 8 cycles: everything plays
)
```

Each character in the string = one pattern. `1` = play, `0` = silent,
`-` = continue. More visual and readable than adding `.mask()` to every
pattern individually.

---

## trancearp() — Arpeggiator With Presets (warm.strudel.cc only)

A pre-built arpeggiator with direction presets. Instead of building your
own `n()` pattern, give it notes and it handles the movement:

```js
$: n(trancearp("<0:2:6@3 2:4:8>", "F@5 B@3", "<1 1 1@2 1 1@2 1>*16"))
  .scale("D:minor")
  .sound("sawtooth")
  .acidenv(slider(0.5))
```

| Argument | What it means |
|---|---|
| First | Notes to arpeggiate — `:` separates chord tones |
| Second | Direction: `F` = forward, `B` = backward |
| Third | Rhythm pattern — which steps trigger |

---

## spinor() — Spinning Ambient Texture (warm.strudel.cc only)

A pre-built atmospheric sound that creates a swirling, evolving texture.
Set it and forget it — useful as a background layer:

```js
// subtle background wash
$TEXTURE:
spinor(0.3, 0)
  .slow(8)
  .room(1)
  .gain(0.4)
  .orbit(9)

// more aggressive, moving texture
spinor(0.9, 0.5)
  .slow(2)
  .room(0.5)
  .orbit(9)
```

| Parameter | What it controls |
|---|---|
| `alive` (0-1) | How active and chaotic the texture is |
| `dead` (0-1) | How much FM/gritty modulation is added underneath |
