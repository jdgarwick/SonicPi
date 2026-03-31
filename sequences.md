# The Math Behind Organic Music in Sonic Pi

A reference guide for applying mathematical sequences, rhythmic theory, and humanization techniques to Sonic Pi compositions. All examples are drawn from a real production track and are ready to paste.

---

## Table of Contents

1. [Core Rhythmic Math](#1-core-rhythmic-math)
2. [Mathematical Sequences — Personality & Application](#2-mathematical-sequences--personality--application)
3. [Humanization Techniques](#3-humanization-techniques)
4. [Advanced Rhythm Structures](#4-advanced-rhythm-structures)
5. [Full Layer Mapping](#5-full-layer-mapping)

---

## 1. Core Rhythmic Math

### Binary Pulse — The 0/1 Grid

Drum arrays are binary on/off matrices. 16 steps = 16th-note resolution in 4/4 time.

```ruby
drum1_var = [1,0,1,0,1,0,0,0,1,0,1,1,0,1,0,1]
# Each index = one 16th note. 1 = hit, 0 = silence.
```

### Sleep Values as Time Fractions

| `sleep` value | Note value     |
|---------------|----------------|
| `0.25`        | 16th note      |
| `0.5`         | 8th note       |
| `1.0`         | Quarter note   |
| `4.0`         | Whole note / one bar |

A **7/8** feel can be approximated by grouping steps `2+2+3`, where sleeps within a bar sum to 7 beats instead of 8.

### Polyrhythm & LCM

Playing 3 against 4 requires **LCM(3, 4) = 12** subdivisions before the cycle repeats. Neither layer sounds dominant — they trade the downbeat back and forth.

```ruby
# 3:4 polyrhythm skeleton
live_loop :layer_a do
  play :e4
  sleep 1.0 / 3.0   # fires 3 times per bar
end

live_loop :layer_b do
  play :b3
  sleep 0.25         # fires 4 times per bar
end
# They share a downbeat every 12 subdivisions = 3 bars
```

### The Golden Ratio (φ ≈ 1.618)

In a 16-step loop, the golden ratio peak falls at step `16 × 0.618 ≈ 9.9` — meaning **index 9** (0-based). Placing your loudest hit, filter open, or melodic apex here instead of the expected midpoint (index 8) creates asymmetric weight that reads as human feel rather than grid lock.

```ruby
phi_step = (16 * 0.618).round   # => 10 (1-indexed), index 9 (0-indexed)
```

---

## 2. Mathematical Sequences — Personality & Application

Each sequence type has a distinct rhythmic and melodic personality. Choose based on the emotional role of the layer.

---

### 2.1 Fibonacci — Organic, Breathing

`1, 1, 2, 3, 5, 8, 13, 21...`

Each number is the sum of the two before it. Phrase lengths built on Fibonacci numbers feel organic because the ratios between adjacent terms approach φ. A `3+5` grouping sums to 8, preserving loop length while breaking the mechanical 8×equal feel.

**Best for:** Melody, counter-melody — anything that needs to breathe.

```ruby
# Phrase A: first 3 notes — shorter sustain, forward motion
3.times do |i|
  use_synth :prophet
  play melody_notes[i],
    attack: 0.001, decay: 0.05,
    sustain: 0.15, release: 0.2,
    amp: 0.12
  sleep 0.5
end

# Phrase B: last 5 notes — longer sustain, settling/resolving
5.times do |i|
  use_synth :prophet
  play melody_notes[i + 3],
    attack: 0.001, decay: 0.1,
    sustain: 0.4, release: 0.35,
    amp: 0.17
  sleep 0.5
end
# Total = 8 steps. Loop length preserved.
```

---

### 2.2 Euclidean Rhythms — Natural Groove

`spread(hits, steps)` distributes `n` hits as evenly as possible across `m` slots. This is the mathematical basis for nearly every world percussion pattern — clave, tresillo, bossa nova, Afrobeat.

**Best for:** Kick, hats, claps — any rhythmic layer that feels too robotic on a binary grid.

```ruby
spread(5, 16)   # 5 hits across 16 steps — sounds like a clave
spread(3, 8)    # classic tresillo
spread(7, 16)   # dense, African-style groove
spread(9, 16)   # busy hi-hat pattern
spread(3, 16)   # sparse backbeat
```

**Replacing a binary array with Euclidean spread:**

```ruby
# Before — handwritten binary array:
drum1_var = [1,0,1,0,1,0,0,0,1,0,1,1,0,1,0,1]

live_loop :drums1 do
  16.times do |i|
    sample :bd_haus, amp: 4 if drum1_var[i] == 1
    sleep 0.25
  end
end

# After — Euclidean spread:
live_loop :drums1 do
  16.times do |i|
    sample :bd_haus, amp: 4 if spread(5, 16)[i]
    sleep 0.25
  end
end
```

The Euclidean version produces a groove that sits between familiar and surprising — no hit feels arbitrary because they are distributed at maximum rhythmic evenness.

---

### 2.3 Lucas Numbers — Angular Fibonacci

`2, 1, 3, 4, 7, 11, 18...`

Same recursive structure as Fibonacci (`each = sum of previous two`), but starting from `2, 1` instead of `1, 1`. The ratios still converge toward φ, but the sequence takes longer to settle — producing a slightly more angular, unpredictable feel before it finds its groove.

**Best for:** Hook melody, counter-melody — Fibonacci's edgier sibling.

```ruby
lucas_seq  = (ring 2, 1, 3, 4, 7, 11)
lucas_base = 0.125   # base unit — multiply each value by this for sleep duration

live_loop :lucas_hook do
  lucas_seq.length.times do |i|
    play hook_notes.tick,
      attack: 0.01, sustain: lucas_seq[i] * lucas_base,
      release: 0.1, amp: 0.4
    sleep lucas_seq[i] * lucas_base
  end
end
```

---

### 2.4 Prime Numbers — Maximum Unpredictability

`2, 3, 5, 7, 11, 13...`

Primes share no common factors, so prime-spaced rhythms never settle into a repeating sub-pattern. This creates a sense of perpetual forward motion — nothing arrives where you expect it.

**Best for:** Hi-hats, glitchy percussion, any layer that should feel unstable in a productive way.

```ruby
prime_seq  = (ring 2, 3, 5, 7, 11, 13)
prime_base = 0.125   # shortest hit = 0.25 beats, longest = 1.625 beats

live_loop :prime_hats do
  prime_seq.length.times do |i|
    sample :drum_cymbal_closed,
      amp: rrand(0.3, 0.9),
      rate: 1.5,
      cutoff: 130
    sleep prime_seq[i] * prime_base
  end
end
```

> **Note:** Prime-based timing will drift against a fixed 4/4 grid. This is intentional — it creates the effect of a human performer pushing and pulling against the beat.

---

### 2.5 Geometric Growth (×φ) — Exponential Swells

Each term multiplies by φ ≈ 1.618. Produces an exponential curve — slow at first, then accelerating. Ideal for amplitude or filter automation that needs to feel like a natural swell rather than a linear ramp.

**Best for:** `vocal_swells` amplitude, filter cutoff automation, any build that should feel physically inevitable.

```ruby
# Amplitude values growing by φ each bar
geo_seq = (ring 0.1, 0.16, 0.26, 0.42, 0.68, 1.0)

# Applied to vocal_swells — replaces hardcoded bar-by-bar amp values:
live_loop :geo_swells do
  geo_seq.length.times do |i|
    use_synth :hollow
    play chord(:e4, :major),
      attack: 2.0,
      sustain: 1.5,
      release: 2.0,
      amp: geo_seq[i],
      cutoff: 70 + (geo_seq[i] * 30)   # cutoff also grows with amplitude
    sleep 4
  end
end
```

---

### 2.6 Palindrome — Built-In Arc

`3, 5, 8, 13, 8, 5, 3` — a sequence that mirrors itself. Builds to a peak then releases symmetrically. Feels like a breath in and out.

**Best for:** Synth chords, middle harmony — any layer that needs an internal arc within each phrase.

```ruby
palindrome_seq  = (ring 3, 5, 8, 13, 8, 5, 3)
palindrome_base = 0.125

live_loop :palindrome_chords do
  palindrome_seq.length.times do |i|
    use_synth :tech_saws
    play chord(:e4, :major),
      attack: 0.01,
      sustain: palindrome_seq[i] * palindrome_base,
      release: 0.2,
      amp: 0.6
    sleep palindrome_seq[i] * palindrome_base
  end
end
```

The palindrome naturally creates tension (ascending toward `13`) and release (descending back to `3`) without any conditional logic — the shape is built into the sequence itself.

---

## 3. Humanization Techniques

The following techniques move layers from mechanically quantized to perceptibly human-performed, without breaking the tempo grid.

---

### 3.1 Velocity Curves — Shaped Dynamics

Real drummers don't hit randomly louder or softer. They follow a velocity **curve** — beat 1 strong, beat 3 medium, subdivisions tapered in between. A sine wave mapped to the 16-step index produces this automatically.

```ruby
define :hat_func do
  live_loop :hats do
    16.times do |i|
      # Sine curve: peaks at steps 0 and 8, dips between
      velocity_curve = 0.6 + (Math.sin(i * Math::PI / 8) * 0.3)

      # Ghost notes: steps 5 and 13 drop to a whisper
      ghost = [5, 13].include?(i) ? rrand(0.2, 0.4) : velocity_curve

      sample :drum_cymbal_closed,
        amp: ghost,
        rate: 1.5,
        cutoff: 130,
        pan: rrand(-0.3, 0.3) if hat_var[i] == 1
      sleep 0.25
    end
  end
end
```

The `Math.sin` curve gives shaped dynamics on every pass. Ghost notes at steps 5 and 13 are common in real drumming — completely absent from a flat binary grid.

---

### 3.2 Golden Ratio Accent — The Off-Center Peak

Accent the φ step (index 9) instead of the obvious midpoint (index 8). The accent lands slightly late relative to expectation, which reads as feel rather than error.

```ruby
define :drum1_func do
  live_loop :drums1 do
    16.times do |i|
      phi_boost = (i == 9) ? 1.6 : 1.0   # accent the φ step

      sample :bd_haus,
        cutoff: 80 + rrand(-10, 10),
        attack: 0.005,
        release: 0.2,
        amp: 4 * phi_boost if drum1_var[i] == 1
      sleep 0.25 + rrand(-0.0001, 0.0001)
    end
  end
end
```

The same `phi_boost` logic can be applied to any 16-step loop — hats, hook samples, synth stabs.

---

### 3.3 Steve Reich Phasing — Slow Drift

Two identical (or near-identical) loops running at slightly different `sleep` values drift apart and slowly realign. At 140 BPM, a `0.002` offset accumulates to ~32ms drift per bar — subtle enough to read as feel, but over 8 bars the two lines are meaningfully displaced.

This is why dense electronic music with perfectly quantized layers can sound "cold" — nothing is ever 32ms late.

```ruby
# Original — locked to grid
define :high_harmony do
  live_loop :high_counter do
    16.times do |i|
      use_synth :sine
      play counter_notes[i],
        attack: 0.1, sustain: 0.1, release: 0.2, amp: 0.35
      sleep 0.25
    end
  end
end

# Phased version — drifts against hook_func_samp
define :high_harmony do
  live_loop :high_counter do
    16.times do |i|
      use_synth :sine
      play counter_notes[i],
        attack: 0.1, sustain: 0.1, release: 0.2, amp: 0.35
      sleep 0.252   # +0.002 — phase drift against the hook
    end
  end
end
```

Let this run for at least 16 bars before judging it — the effect is cumulative.

---

## 4. Advanced Rhythm Structures

### 4.1 True 3:4 Polyrhythm

**LCM(3, 4) = 12** — the patterns realign every 12 steps. In a 16-step bar, they won't realign within a single pass, creating a "wandering" feel that resolves across multiple bars.

```ruby
define :mid_poly do |s3|
  live_loop :mid_polyrhythm do
    12.times do |i|
      use_synth s3
      # Fires every 3rd step in a 12-step cycle
      play :b3,
        attack: 0.001, decay: 0.04,
        sustain: 0.0, release: 0.08,
        amp: 0.5 if i % 3 == 0
      sleep 1.0 / 3.0   # 12 × (1/3) = 4 beats = one bar
    end
  end
end

mid_poly :dnoise
```

`:b3` is the fifth of E major — harmonically consonant even when rhythmically displaced. Bring this layer in after the main groove is established, not from bar one.

---

### 4.2 Fractal Motifs — Self-Similar Patterns

A 3-note cell that expands into itself at each level. Each iteration sounds like the previous one, but longer — creates recognition and surprise simultaneously.

```ruby
# Level 1: [S, T, R]
# Level 2: [S T R,  T T R,  R T R]
# Level 3: Each of Level 2's cells becomes its own 3-note phrase

cell = [:e5, :gs5, :b5]

define :fractal_melody do |depth|
  if depth == 0
    cell.each { |n| play n; sleep 0.25 }
  else
    cell.each { fractal_melody(depth - 1) }
  end
end

live_loop :fractal do
  fractal_melody(1)   # depth 1 = 9 notes, depth 2 = 27 notes
end
```

---

## 5. Full Layer Mapping

A complete reference for which sequence to apply to each layer in the track, and why.

| Layer | Sequence | Implementation | Reason |
|---|---|---|---|
| `drum1_func` | Euclidean | `spread(5, 16)` | Natural groove, not robotic |
| `hat_func` | Euclidean + Velocity Curve | `spread(9, 16)` + `Math.sin` | Dense but shaped dynamically |
| `claps` | Euclidean | `spread(3, 16)` | Sparse, musical backbeat |
| `mel_func` | Fibonacci | `3+5` phrase grouping | Breathing, organic phrasing |
| `hook_func_samp` | Lucas | `(ring 2,1,3,4,7,11)` | Similar to Fibonacci, edgier |
| `high_harmony` | Palindrome + Phase | `(ring 3,5,8,13,8,5,3)` + `sleep 0.252` | Arcs up and back; drifts against hook |
| `vocal_swells` | Geometric (×φ) | `(ring 0.1, 0.16, 0.26, 0.42, 0.68, 1.0)` | Amplitude grows by the golden ratio |
| `middleharm` | Prime | `(ring 2,3,5,7,11,13)` | Unpredictable punch hits |
| `synth_chords` | Powers of 2 | `(ring 1, 2, 4, 8)` | Simple, structural, grounding |
| `drum1_func` accent | Golden Ratio | `phi_boost` at index 9 | Off-center peak feels like feel |

---

## Quick Reference — Sequence Values

```ruby
# Fibonacci
fib_seq = (ring 1, 1, 2, 3, 5, 8, 13, 21)

# Lucas
lucas_seq = (ring 2, 1, 3, 4, 7, 11, 18)

# Primes
prime_seq = (ring 2, 3, 5, 7, 11, 13)

# Geometric (×φ) — amplitude
geo_seq = (ring 0.1, 0.16, 0.26, 0.42, 0.68, 1.0)

# Palindrome
palindrome_seq = (ring 3, 5, 8, 13, 8, 5, 3)

# Euclidean spreads
spread(5, 16)   # clave-like kick
spread(3, 8)    # tresillo
spread(7, 16)   # dense Afrobeat
spread(9, 16)   # busy hi-hat
spread(3, 16)   # sparse backbeat

# Base unit for sleep durations
base = 0.125    # one 32nd note at 4/4
```

---

*Reference built from a 140 BPM Sonic Pi production in E major. All sequence math is independent of key and tempo — adjust `base` values to fit your BPM.*
