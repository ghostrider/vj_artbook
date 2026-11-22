---
tags:
  - visual-music
  - art-history
  - harmonics
  - geometry
---

# Whitney Digital Harmonics

> A ring of dots turns at whole-number speeds, scatters into a cloud and snaps back into a figure on the downbeat.

![Black and red spirograph art: looping round lines on a dark ground](https://images.unsplash.com/photo-1603124314041-15e7fd60a23d?w=1400)
*Similar look, not the original work. Photo: [Glen Carrie](https://unsplash.com/photos/black-and-red-round-illustration-buTmRpLR1qY) on [Unsplash](https://unsplash.com/license).*

## The idea

Give every dot its own speed. Make the speeds whole-number multiples of one base rate: 1x, 2x, 3x and so on. Give each dot its own direction. Let no randomness in. The field then forms and reforms recognisable patterns. John Whitney called this harmonic resonance ([Digital Harmony text](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt)).

Order shows most clearly when the speed ratios are whole numbers. The motion resolves like a dissonance resolving into consonance. Whitney carried the low-order Pythagorean ratios (octave, fifth, fourth, third) into visual motion. He proposed that ratio, interference and resonance laws work in vision as they do in music (same source).

On a club screen this is tension and release without a single cut. The field blooms away from a clean figure, smears into spirals and rosettes, then locks back. The rhythm comes from the cycle time of the ratio. Set that cycle to 1, 2 or 4 bars and the resolution lands on the downbeat.

## Origins and references

- John Whitney Sr. (1917-1995), American animator, composer and inventor, often called a pioneer of computer animation ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).
- In the late 1950s he converted a WWII M-5 anti-aircraft gun director into an analog animation machine, with design templates on rotating tables photographed by a rotating camera ([Socks-Studio](https://socks-studio.com/2010/11/25/john-whitney-inventor-and-father-of-computer-animation/)).
- *Vertigo* titles (1958, with Saul Bass), Motion Graphics Inc. (1960) and *Catalog* (1961) ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).
- *Homage to Rameau*, his first computer film, named after Rameau's treatise on harmony. IBM made him its first artist in residence in 1966, and Jack Citron programmed IBM 360 computers for him ([Visual Music Archive](https://visualmusicarchive.org/reading/r/digital-harmony.html)).
- *Permutations* (1966): coloured dots in evolving patterns, each point at its own speed. Made on an IBM 360 with an IBM 2250 display, colour added later with an optical printer ([Visual Music Archive](https://visualmusicarchive.org/works/w/permutations.html)).
- *Matrix III* (1972, 11 min): triangles, squares and hexagons follow invisible paths from a Lissajous figure applied symmetrically over the frame. Music by Terry Riley ([Light Cone](https://lightcone.org/en/film-1555-matrix-iii)).
- *Arabesque* (1975): described as the peak of his digital films, with blooming colour forms ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).
- Whitney-Reed RDTD (Radius-Differential Theta Differential), his later audio-visual composition program, used in works such as *Moon Drum* (1989-1995) ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).
- Book: *Digital Harmony* (1980), with code examples ([Visual Music Archive](https://visualmusicarchive.org/reading/r/digital-harmony.html)). Full text on [archive.org](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt).

![Still from Permutations by John Whitney](https://computerhistory.org/wp-content/uploads/2022/05/Permutations-2-760x0-c-default.jpg)
*Permutations, John Whitney, 1966. © Whitney Editions, LLC. Shown as a quotation for reference; source: [Computer History Museum](https://computerhistory.org/exhibits/technology-art/).*

## Visual language

- Many small dots or short lines on true black, in flat saturated colour.
- At the start of a cycle all dots stand in one figure: a ray from the centre, a single point or a regular polygon. Then the ray curls into a spiral, the spiral splits into 2, 3, 5 and 7 arms, the arms dissolve into a shimmering rosette cloud, and the field snaps back.
- Radial symmetry, spirals, rosettes, Lissajous-like weaves.
- Each dot draws a short comet tail, so the whole field reads as turning.
- Colour follows *Permutations*: dye-like magenta, amber and cyan, with a slight RGB misregistration like an optical printer.
- Mood: calm, precise, hypnotic. The tension and release is the whole show.

## See it

- [Permutations (Visual Music Archive)](https://visualmusicarchive.org/works/w/permutations.html) — page on the 1966 film of coloured dot patterns.
- [Matrix III (Light Cone)](https://lightcone.org/en/film-1555-matrix-iii) — film record with the Lissajous description.
- [Matrix III analysis (Motion Design Archive)](https://luckydumpling.github.io/motion-archive/entries/john-whitney-matrix/) — embeds the film, notes ties to Processing and TouchDesigner.
- [Digital Harmony full text](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt) — the book with the theory.
- [Whitney overview (Socks-Studio)](https://socks-studio.com/2010/11/25/john-whitney-inventor-and-father-of-computer-animation/) — machine and film images.

## Code design

N dots (24 to 360, default 120) share one global phase φ, measured in turns. Dot k sits at angle `2π · m · k · φ`, so it turns k times faster than dot 1. At whole-number φ every dot is aligned. At φ = p/q the dots fall onto only q angles and show a q-arm figure or q-gon. A consonance value c, from 1 in a clean figure to 0 in the cloud, drives brightness and whitening.

Tension and release comes from the phase driver. *Breath* swings φ out and back over 1, 2 or 4 bars. *Sweep* drifts slowly for ambient breaks. *Resolve* is a trigger that glides φ to the nearest simple fraction. Placement modes: spiral arm, ring, symmetry order 1 to 12, an RDTD-style radius differential (the program's exact formulas are not documented in the sources) and a Lissajous mode for *Matrix III*. A speed clamp avoids wagon-wheel strobing. Analytic comet arcs give trails without feedback. A chord mapper reads up to three voices from held notes.

The design runs in passes. P0 is the phase clock (logic only). P1 draws the field per pixel in HDR. P2 is an optional feedback bloom. P3 does tone map and optical-printer colour offset.

=== "Resolume Wire"
    - Clock: BPM and beat position, divided by 4 times bars, then fract, triangle and a cubic ease, times `bloom`, gives φ. A parallel accumulator gives Sweep, and a `Switch` picks the driver.
    - If Wire has no delay node, approximate φ_prev as φ minus the derivative times τ. It is exact for Sweep and close for Breath.
    - Render with one ISF node, "WhitneyField" (P1 and P3 in one pass), then Bloom. Multipass ISF support varies by Wire version, so do not rely on feedback. Wire has no vertex shaders, so Lissajous mode stays an N-loop in the fragment shader, N up to 128.
    - Audio: low band to an envelope (attack 5 ms, release 150 ms) for the pulse. High band onsets step the colour ramp.
    - MIDI: KeyStep 37 held notes, sorted, give intervals above the lowest note, then a lookup table gives the voice multipliers.
    - In Arena, put the ISF source on layer 2 with Add over a black layer 1. Resolume documents ISF in the context of Wire ([Resolume ISF](https://resolume.com/support/en/isf)), so test loading before the gig. Bloom is built in since 7.2 ([Resolume blog](https://resolume.com/blog/19880)). FeedbackPro (Arena 7.18.2+) is a stand-in for trails ([Resolume blog](https://www.resolume.com/blog/23113)).

=== "Unreal Engine 5.7"
    - Niagara emitter `NE_Harmonics`: GPU Compute Sim, one instantaneous burst of N particles (maximum 1024), infinite lifetime. A Scratch Pad module reads Phase, PhasePrev, Mult, Mode and Symmetry from User parameters and sets each position.
    - Sprite Renderer with an unlit additive material and colour from a Curve Linear Color at k/N. For tails, add a velocity-aligned stretched sprite instead of ribbons, which get expensive above N = 300.
    - A Blueprint actor computes φ, φ_prev and c each tick from BPM and the driver.
    - Post-process: Bloom 1.0 to 2.0, Film Grain 0.1 to 0.2, Chromatic Aberration 0.1 to 0.3 as the printer offset, manual exposure so black stays black.
    - 3D variant: each voice gets a Z offset and the CineCamera dollies through the axis on the aligned downbeat.
    - Remote Control preset exposes bloom, focus, ratio, symmetry, tail, cycle, resolve and BPM, bound to OSC and MIDI. Output by Spout or NDI into Arena layer 2, Add.

=== "GODOT"
    - 2D route: a full-viewport `ColorRect` with a CanvasItem shader implementing P1 and P3. Uniforms include phase, phase_prev, consonance, dot_count, mult, symmetry and a palette gradient texture.
    - Trails: a `SubViewport` at half size, HDR 2D, clear mode Never, holding a decay and zoom shader of its own previous frame plus the field.
    - 3D or Lissajous: `GPUParticles3D` with a process shader setting position from INDEX, or `MultiMeshInstance2D` for N up to 500.
    - A GDScript autoload holds the clock. MIDI arrives through `InputEventMIDI`. Glow in `WorldEnvironment`, additive.
    - Spout needs a third-party plugin. Otherwise capture the window in Arena or use an NDI addon.

## On stage

**Audio reactivity.** Never drive φ from audio. The tension and release needs φ smooth and rule-bound. Audio changes size, radius, tail and colour, and triggers Resolve.

| band | drives | smoothing |
|------|--------|-----------|
| kick (40-120 Hz) | dot size pulse +40 % | attack 5 ms, release 150 ms |
| bass (60-250 Hz) | radius R ±4 % | attack 40 ms, release 400 ms |
| snare / mid (1-3 kHz) | tail τ +30 % per hit | attack 10 ms, release 300 ms |
| hats (6-12 kHz) | colour ramp step +0.02 | stepped, 60 ms hold-off |
| 4-bar energy | upper limit of bloom | 4-bar average |

**MIDI.**

- BeatStep Pro knob 1: `bloom`. Knob 2: `focus`, stepped 24, 60, 120, 240, 360. Knob 3: `symmetry`. Knob 4: `tail`. Knob 5: `cycle`.
- BeatStep Pro pad: `resolve`.
- KeyStep 37: held notes set the ratios in chord mode. An octave is 1:2, a fifth 2:3, a fourth 3:4, a major third 4:5, a minor third 5:6. A major triad gives m = 4, 5, 6 and a minor triad 10, 12, 15, so the figure follows the chord.

**Arc of a set.**

- Build-up: raise `bloom` from 0.01 to 0.3 over 8 to 16 bars and N from 60 to 240. The field passes more complex figures and ends in the cloud.
- Drop: Resolve with Q_snap 1 on the first downbeat, hold one beat, swap the palette, then Breath with a 1-bar cycle.
- Break: Sweep at 0.001 turns/s, tail 1.2 s, feedback on.
- Every 8 bars step symmetry (1, 3, 5, 6) and rotate the colour ramp by a third. Ease jumps over one beat, except snaps.

**Layering.** Add over footage, or Screen at 70 % on bright footage. As a luma matte, the video shows only through the dots and arcs, so the image is revealed at each resolution. In Arena the quick version is Multiply. Send c by OSC to drive another layer's opacity.

**Tips.**

- Keep true black, saturation at 85 to 95 % and peak luma near 0.9 for LED walls.
- Size dots from the shorter screen side. Keep a minimum dot radius of 1.5 physical pixels and lower N instead.
- Clamp the per-frame phase step. Wagon-wheel effects start earlier at 50 Hz.
- On walls wider than 3:1, use two figure centres instead of one.
- Boring risk: a ratio that never changes, or a pattern never allowed to dissolve.

## Variations

1. Chord mapper: MIDI notes set the ratios per ring, so a major chord shows a different figure than a minor one.
2. Detune drift: slightly off-integer ratios, so the resolution slowly precesses and never repeats exactly.
3. 3D rings in UE Niagara, with the camera dollying through the aligned moment.
4. Video sampling: each point takes its colour from a live input at its position.
5. Multi-ring polyrhythm: concentric rings with different ratios, aligned only every N bars.

## Did you know?

- Whitney's analog computer began as a WWII anti-aircraft gun director ([Socks-Studio](https://socks-studio.com/2010/11/25/john-whitney-inventor-and-father-of-computer-animation/)).
- IBM gave him its first artist-in-residence position, in 1966 ([Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))).
- *Permutations* was shot from a black-and-white 2250 display, and the colour was added afterward in an optical printer ([Visual Music Archive](https://visualmusicarchive.org/works/w/permutations.html)).
- *Matrix III* is built on one Lissajous figure applied symmetrically over the frame ([Light Cone](https://lightcone.org/en/film-1555-matrix-iii)).

## Sources

- [John Whitney (animator), Wikipedia](https://en.wikipedia.org/wiki/John_Whitney_(animator))
- [Digital Harmony, Visual Music Archive](https://visualmusicarchive.org/reading/r/digital-harmony.html)
- [Permutations, Visual Music Archive](https://visualmusicarchive.org/works/w/permutations.html)
- [Matrix III, Light Cone](https://lightcone.org/en/film-1555-matrix-iii)
- [Digital Harmony, full text](https://archive.org/stream/DigitalHarmony_201611/Digital%20Harmony_djvu.txt)
- [Socks-Studio on Whitney](https://socks-studio.com/2010/11/25/john-whitney-inventor-and-father-of-computer-animation/)
- [Matrix III analysis, Motion Design Archive](https://luckydumpling.github.io/motion-archive/entries/john-whitney-matrix/)
- [Resolume ISF](https://resolume.com/support/en/isf)
- [Resolume blog: Bloom](https://resolume.com/blog/19880)
- [Resolume blog: FeedbackPro](https://www.resolume.com/blog/23113)
- [Lissajous curve, Wikipedia](https://en.wikipedia.org/wiki/Lissajous_curve)
- [ISF JSON reference](https://docs.isf.video/ref_json.html)
