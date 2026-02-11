# Production Notes

## FX Notes

## Compression

Compression controls the volume dynamics of audio by detecting when the volume exceeds or drops below a specified level, and then adjusting it by a specified amount. It narrows the difference in volume between the loudest and softest parts of a track so that it’s more consistent in level. It allows for a more consistent volume and prevents harsh spikes

### Parameters

* ***Threshold***: Defines the point at which the compressor kicks in.
* ***Ratio***: When the compressor detects that audio has exceeded the threshold, the ratio defines the *extent* of the the compression that is applied to the audio above it. Those that have high ratios (10:1 or higher) are called **limiters**. A limiter is essentially a compressor with an ‘infinite’ ratio. While a compressor (particularly one with a lower ratio) gently tames peaks, a limiter aggressively puts a halt on any level that exceeds the threshold (basically like a ceiling for the audio).

This is represented in Sonic Pi using the "slope_above" and "slope_below" functions.

* <img width="900" height="495" alt="image" src="https://github.com/user-attachments/assets/63dc6f16-0b70-4027-b47a-3c079798f53a" />

* ***Attack*** (clamp_time): Represents how fast the compressor kicks in when the audio exceeds the threshold. Fast attack = clamps down immediately (good for taming sharp transients like snares). Slow attack = lets initial hit through before compressing (good for punchy beats).
* ***Release*** (relax_time): Opposite of attack. Represents the time it takes for the audio to return to being uncompressed. Short release ➡️ lets go quickly. Long release ➡️ holds compression for longer. If release is too short, you might hear pumping. If too long, it keeps the signal squashed even when it doesn’t need to be.

## EQ

In using EQ, we thing about the balance of frequencies (the vibrations we hear). When we hear a sound like the strumming of a guitar, we hear an intricate mix of countless frequencies within a WIDE range that make up the timbre, or sonic character, of the sound. But, we dont want the frequencies of every track, because then they all bleed together. We want to give tracks their own space within the mix.

EQs include controls allowing you to boost/cut parts of the audio, which allows you to adjust the balance of frequencies to get the desired timbre (the characteristic quality, color, or "personality" of a sound that distinguishes it from others) we’re looking for. When we use EQ, we’re not making things lower or higher in pitch. Rather, we’re using it to make things sound brighter, bassier, clearer, etc. Like sorting the frequencies into their own categories so frequencies from many instruments aren't competing with each other.

These specific ranges aren’t set in stone, but when producers make observations like “the low end sounds muddy” or “the high mids are too bright,” these frequency ranges are generally what are being referred to:

* **Low**: 20 – 250 Hz. ‘Muddiness’ can occur when there’s excessive build up around 200 Hz
* **Low Mid**: 250 Hz – 1 kHz. To reduce ‘boxiness,’ consider attenuating around 400 – 500 Hz

Low band carries weight, power, size, and impact. This is the physical part of sound that carries things like kick, thump, bass guitar, sub bass, warmth of piano, fullness of a mix, chest impact of drums.

Too much makes it muddy, but too little makes it skinny and lifeless.

* **Mid**: 1 kHz – 4 kHz. ‘Tinny’ or ‘nasally’ characteristics can be targeted around 1 – 2 kHz
* **High Mid**: 5 kHz – 8 kHz. ‘Harsh’ frequencies can often lie between 6 – 8 kHz

Mids contain clarity, character, focus, and intelligibility. This band carries the information of the song, and if you mute it, the song sounds empty. It includes things like, guitar body, piano tone, snare crack, speech intelligibility, and instrument personality

* **High**: 8 kHz – 20 kHz. ‘Air’ and ‘brightness’ can be found around 10 kHz for instruments such as cymbals and piano

Highs include brightness and airiness. This is the “shine” on top and includes hi-hats, cymbal shimmer, vocal air, sparkle, crisp attack, and sizzle. If the music is piercing, static-y, dull, etc., then this is in the high band. Too much is fatiguing, and too little sounds like a blanket over the speakers. It emotionally controls polish and cleanliness.

When trying to create a full-sounding mix, it’s important to make sure all of these ranges have some amount of representation in your track. You can use EQ to fix unwanted frequency clashes, increase the presence of particular instruments, or improve the overall quality of a sound in the context of your track.

## How to Use EQ

1. **Correcting Something**: EQ is  used to bring out the best of a performance and to attenuate the less desirable elements. If your vocals lack definition, you can give the higher frequencies a boost to make it feel more clear and crisp. Alternatively, if it sounds too harsh or hissy, you can reduce these same frequencies for a more well-rounded sound.
2. **Creating Space for Tracks**: Instruments will sound clearer in your mix if they’re not competing for attention with a million other elements that reside in the same frequency range. If you have a bass guitar and a kick drum, for example, you can use EQ to boost and cut different frequency ranges within the low end (usually by just a few decibels at most—no need to overdo it) so that each sound has its own sonic range where it can shine.
3. **Achieving Balance**: Mixes sound the best when they have content across our full range of hearing. We often use EQ with this macro perspective in mind, making sure our overall track sounds full and engaging.

### EQ Opts and Workflow:

**EQ Workflow**

* If you want to change the **overall tone** (change the ratios of low to high. shifts the balance one way or another. "is the whole low/high side off?") → shelf.
* If you want to change **one specific annoying frequency area** (a sharp frequency that hurts or a boomy note)→ non-shelf band.
* If you want to remove an **entire range of unneccessary frequencies** (like if hi-hat has sub bass that you don't need, if guitar has useless low mud, or if sound has harsh fizz you don’t want at all) → filter.

**Filter Opts**
Use when you want to remove entire regions.

* **lpf**: Dampens the parts of the sound that are above the cutoff point (typically the crunchy fizzy harmonic overtones) and keeps the lower parts (typically the bass/mid of the sound). Choose a higher cutoff to keep more of the high frequencies/treble of the sound and a lower cutoff to make the sound more dull and only keep the bass. It is a type of shelf eq (use the shelf opt on eq if you want precise control)
* **hpf**: Dampens the parts of the signal that are below the cutoff point (typically the bass of the sound) and keeps the higher parts (typically the crunchy fizzy harmonic overtones). Choose a lower cutoff to keep more of the bass/mid and a higher cutoff to make the sound more light and crispy.It is a type of shelf eq (use the shelf opt on eq if you want precise control)

**Shelf Opts**

* **low_shelf**: A type of EQ that grabs the frequencies below the low_shelf_note, and boosts/cuts them by the set value
* **low_shelf_note**: Sets the boundary for which frequencies are boosted/cut. "Below what point do we boost/cut by the low_shelf value?"
* **low_shelf_slope**: Sets the rate at which the frequencies reach the boost/cut value (the low_shelf value)
* **high_shelf**: A type of EQ that grabs the frequencies above the high_shelf_note, and bossts/cuts them by the set value
* **high_shelf_note**: Sets the boundary for which the freequencies are boosted/cut. "Above what point do we boost/cut by the high_shelf value?"
* **high_shelf_slope**: Sets the rate at which the frequencies reach the boost/cut value (the high_shelf value)

**Non-Shelf Opts**

These define specific bumps within the range set by the shelf parameters. They allow you to grab ONE spot in the area and push it up or down. "Turn THIS specific bass point up or down"

* **low_note**: Defines where the bump happens within the shelf (lower = deeper bass)
* **low**: Defines the the amount by which the the bump (low_note) is boosted/cut
* **low_q**: "What area of frequencies around the low_note value should be booested/cut by the 'low' value?" The higher this value, the skinner the area of frequencies. The lower this value, the wider the area of frequencies that gets boosted/cut. Lower Q value = Make that whole bass area bigger. Higher Q value = Make THAT bass tone bigger. A decent range of Q factors for naturally sounding boosts/cuts is 0.6 to 1.
* **mid_note**: Defines where the bump happens in the middle of the sound. Lower mid_note = low-mids (warmth / boxy area). Higher mid_note = upper-mids (presence / bite area). "Which part of the middle am i touching?
* **mid**: Defines the amount by which the bump (mid_note) is boosted/cut
* **mid_q**: "What area of frequencies around the mid_note value should be boosted/cut by the 'mid' value?"
* **high_note**: Defines where the bump happens in the high part of the sound. Lower high_note = upper-mids (edge / sharpness). Higher high_note = highs (sparkle / air)
* **high**: Defines the amount by which the bump (high_note) is boosted/cut
* **high_q**: "What area of frequencies around the high_note value should be boosted/cut by the 'high' value?"

***Sample Workflow for Removing a Specific Frequency with Non-Shelf Opts:***

1. Before touching EQ, identify the problem. This will determine which band you should use to adjust the track:
    * Boomy? → Low problem
    * Muddy / boxy? → Low-mid problem
    * Nasal / honky? → Mid problem
    * Harsh / piercing? → Upper-mid problem
    * Static-y / brittle? → High problem
  
    * Does it feel too heavy or too thin? → Adjust LOW
    * Does it feel unclear or annoying? → Adjust MID
    * Does it feel too bright or too dark? → Adjust HIGH
  
2. Narrowly boost the part of the band you may suspect the problem to lie. With mids, for example:
    * This gives you a flashlight, rather than a floodlight into the band. (Allows you to be surgical in finding where the problem is in the band)

```
mid: 0.8
mid_note: 83
mid_q: 8
```

3. Move the low/mid/high_note value slowly. You are trying to listen for when it sounds worse/when the annoying quality of the track jumps out
4. Once you've identified where the problem is, flip the boost (the spotlight we were using to find the problem) into a small cut (removing that annoying quality from the track)
    * changed the 'mid:' value to cut the annoying quality out, and lowered the 'mid_q' value, widening it to blend the cut into the frequencies around it

```
mid: -0.3
mid_note: 83
mid_q: 2
```


