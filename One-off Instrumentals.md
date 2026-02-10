# Fun Sounds to Isolate

Here's an intro sound 

'''
# variables
guit_e_dur_half = 2.985

```
# intro slice - NOTE THE FX CHAIN: GATE/DYNAMICS (###) -> COMPRESSOR (CLEANS NOISE BY REDUCING LOUD AND RAISING QUIET) -> EQ (SHAPES THE TONE. CAN MAKE WARMER/BRIGHTER/THICKER/THINNER) -> DELAY FX (ECHO/DELAY/ETC.) -> REVERB
with_fx :ring_mod, amp: 5 do
  stop
  with_fx :panslicer do # ADD SLIDE OPTS TO CHANGE THE DURATION & WAVELENGTH OF THE SLIDE
    with_fx :slicer, phase: 0.05 do # slices whats inside and defines the length of the slices
      with_fx :echo, attack: guit_e_dur_half, release: guit_e_dur_half, mix: 0.15 do # adds echo, sets the attack/release length to 1/2 of the sample, & sets mix as 0 (amt of FX present in the sound)
        in_thread do # TRY DELETING THE THREAD AND SEE WHAT HAPPENS - MAY ONLY NEED THIS FOR LOOPS
          sample :guit_e_slide, attack: 5, sustain: 2, release: 2, amp: 2 # plays the sample with a sustain level, amp level, & pans from left to right
        end
      end
    end
  end
end
```
