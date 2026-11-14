---
tags:
  - color-theory
  - op-art
  - bauhaus
  - geometry
---

# Albers Color Interaction

> A colour never sits alone. Swap the ground on the beat and the middle square seems to change, though its pixels never do.

![Two identical grey inner rectangles look different on different surrounds (simultaneous contrast)](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0a/Simultaneous_Contrast.svg/1280px-Simultaneous_Contrast.svg.png)
*Two identical grey inner rectangles that look different because of their surrounding shades, the effect Albers' exercises build on. An illustration of simultaneous contrast, not an Albers work. Image: [Qef (SVG), Xanzzibar (bitmap), after K. P. Miyapuram](https://commons.wikimedia.org/wiki/File:Simultaneous_Contrast.svg), [Public domain](https://commons.wikimedia.org/wiki/File:Simultaneous_Contrast.svg).*

## The idea

Josef Albers taught that a colour is almost never seen as it really is. Colour is the most relative medium in art. His method puts experience before theory: you look, compare, and are surprised ([The Marginalian](https://www.themarginalian.org/index.php/2013/08/16/interaction-of-color-josef-albers-50th-anniversary/)). His book *Interaction of Color* (1963) "crowned thirty years of effort" ([Albers Foundation](https://www.albersfoundation.org/alberses/teaching/interaction-of-color)).

This chapter is an original piece built on his exercises, not a reproduction of his paintings. The wall is divided into square cells. Each cell holds three or four nested squares. Nothing glides. Colours are swapped on musical positions, and the audience sees things that are not there: a centre that "changes" while its value stays fixed, two different squares that suddenly read as one, a grey screen that fills with a violet ghost after a long yellow hold.

On a club screen this works because it is the opposite of the usual flood of motion. Flat, opaque fields and hard edges give the eye something to compare. A cut on the kick, then a held bar, then another cut. The mood is calm, exact and slightly uncanny, like a lecture demonstration turned into a light show.

## Origins and references

- [Josef Albers](https://en.wikipedia.org/wiki/Josef_Albers) (1888-1976): taught at the Bauhaus from 1920, left Germany after the school closed in 1933, then led painting at Black Mountain College until 1949 and Yale's design department from 1950.
- [Interaction of Color (1963)](https://www.albersfoundation.org/alberses/teaching/interaction-of-color): exercises plus about 200 reproductions, mostly made by his students, with Albers' commentary. The first edition was a silkscreen edition with 150 plates ([Yale News](https://news.yale.edu/2013/07/25/art-book-ipad-app-josef-albers-classic-work-undergoes-magical-transformation)).
- [Homage to the Square](https://en.wikipedia.org/wiki/Homage_to_the_Square) (1950-1976): over 1,000 paintings of three or four nested squares, oil on Masonite, paint straight from the tube.
- [Simultaneous contrast](https://en.wikipedia.org/wiki/Simultaneous_contrast): older than Albers. Chevreul's 1839 book is a standard reference, and earlier observations go back to Ibn al-Haytham, Goethe and Müller.
- [Interaction of Color app (2013)](https://www.commarts.com/project/22915/interaction-of-color): Yale University Press, the Albers Foundation and the studio Potion; over 125 interactive studies.
- [Complete Digital Edition (2022)](https://www.albersfoundation.org/news/yale-press-launches-the-interaction-of-color-complete-digital-edition): a Yale website with 122 interactive plates.
- [HTTS by Cassiel (Nick Rothwell)](https://cassiel.com/project/htts): an algorithmic, evolving Homage, commissioned by Studio Wayne McGregor with the Albers Foundation.
- [AutoAlbers](https://www.github.com/y-a-v-a/autoalbers): a JavaScript canvas project that makes a new Homage every 12 seconds.

## Visual language

Flat, unmixed colour, hard edges, a strict schema. The squares sit low in the cell: the margin under each square is smaller than the space above it, and a frame surrounds the whole ([Wikipedia: Homage to the Square](https://en.wikipedia.org/wiki/Homage_to_the_Square)). Albers called squares useful because they do not occur in nature and "sit", which lets colour stand out. The same palette in a different order gives a different picture.

Rhythm comes from colour steps, not movement. Swap one colour and every neighbour reads differently. The exercises used here:

- **One colour looks like two.** A fixed figure on a ground that flips.
- **Two colours look like one.** Two different figures, compensated for their grounds, read as identical. Then the grounds are cut away and they split apart.
- **Bezold effect.** Thin stripes of one colour across a field shift the field. Narrow stripes pull colours toward their neighbours (assimilation), wide ones push them apart (contrast) ([Wikipedia: Bezold effect](https://en.wikipedia.org/wiki/Bezold_effect)).
- **Transparency.** Two overlapping rectangles with a third, perceptually mixed colour in the overlap read as a see-through sheet.
- **Afterimage.** Hold saturated discs, then cut to neutral grey. The complementary ghost appears ([Wikipedia: Afterimage](https://en.wikipedia.org/wiki/Afterimage)).

No texture, no glow, no blur, no grain, no feedback. A swap must stay a cut.

## See it

- [Logic and Magic of Color, videos](https://albersfoundation.org/alberses/teaching/josef-albers/logic-magic-of-color) — four short animated-plate videos from the Albers Foundation, ending with the Bezold effect.
- [Cassiel, HTTS](https://cassiel.com/project/htts) — animated GIFs of algorithmically generated Homages.
- [AutoAlbers (GitHub)](https://www.github.com/y-a-v-a/autoalbers) — source for a canvas that makes a new Homage every 12 seconds.
- [Interaction of Color app, Communication Arts](https://www.commarts.com/project/22915/interaction-of-color) — project page for the interactive plate app.
- [An iconic book reimagined: Josef Albers' "Interaction of Color"](https://www.youtube.com/watch?v=H-hFOZ3XdIQ) — Yale University Channels; a video on the reimagined book.

## Code design

The whole look is one fragment shader (P0): cells, nested squares, exercise geometry, palette lookup, OKLab easing and the output transform. An optional second pass (P1) samples four colours from a Spout, NDI or camera feed and sorts them by lightness. There is no feedback buffer, no bloom and no blend mode between colour fields.

Per pixel: find the cell and local coordinates, find the region (nested square, split side, stripe, overlap, disc, dot), look up its slot in the current and next palette, convert both to OKLab, mix, convert back, and reduce chroma at fixed lightness and hue if the result leaves the gamut. Then apply per-channel trim and black lift, encode sRGB, and add a Bayer dither of about half a code value, only while a morph is running. Edges are snapped to whole pixels with no anti-aliasing: on an LED wall a 1 px blend shows as a soft seam, and Albers' fields need hard edges.

The nested-square geometry, with step m = 0.1 and four levels, gives sides of 1.0, 0.8, 0.6 and 0.4 of the cell. The inner square ends 0.15 from the bottom and 0.45 from the top, which produces the "smaller margin below" rule.

=== "Resolume Wire"
    - Build P0 as an ISF fragment shader inside a Wire patch and use the patch in Arena as a source. [Resolume documents ISF support](https://resolume.com/support/en/isf) in Wire, and states that Wire (version 7.22) does not support vertex shaders. P0 is fragment-only and single-pass, so that is fine.
    - Clock: BPM or beat phase into bar and phrase counters, which select mode and palette and fire the swap trigger with a rate selector. Node names differ between Wire versions; use the beat-clock and counter nodes from the node browser.
    - Audio: low-band analysis into an envelope (attack 5 ms, release 150 ms), a threshold, then the swap trigger, gated by a `build` toggle.
    - Morph: compute beat phase since the swap divided by morph length, clamp 0..1, smoothstep, and pass it in. The OKLab mix itself stays in the shader.
    - Palette sampler: a second ISF node at 4 × 1 output fed by a Spout or NDI input node, smoothed with Wire's feedback node, routed to P0's `sampled` input.
    - Arena side: composition equals the LED pixel map, Advanced Output slice 1:1, clip Scale 100 %, Rotation 0. One clip per palette (A to F) with colours stored in the clip; triggering the clip on the beat is a palette cut. No Arena effects, layer opacity or crossfader on the Albers layer.
    - Without Wire: stack Arena Solid Color sources as nested layers at 100, 80, 60 and 40 % scale, Position Y shifted by the margin rule, all Normal at 100 %. This covers Homage, One-as-two and Two-as-one. Bezold, transparency mixes and OKLab morphs need the shader. Precompute overlap colours by hand.

=== "Unreal Engine 5.7"
    - Material: a post-process material with Blendable Location "Replacing the Tonemapper", so tonemapper, exposure and colour grading cannot alter the hex values. Alternative: an unlit material on a full-screen plane, captured with tonemapping disabled.
    - Build P0 from ScreenPosition × ViewSize for pixel coordinates, Floor and Frac for cells, comparisons for the square schema, and a Custom node for sRGB to OKLab and back (Ottosson's two matrices plus cube root and cube).
    - Parameters: a Material Parameter Collection `MPC_Albers` with Mode, Contrast, HuePush, Match, Stripe, Morph, Depth and the palette slots as vectors. A Blueprint holds the palette library as a data table of hex values, converts to OKLab once per palette change, and writes the collection.
    - Niagara is not needed; the look is flat fields.
    - Post-process volume: bloom 0, vignette 0, film grain 0, auto exposure off, motion blur off, Screen Percentage 100.
    - Remote Control: a preset exposing the collection scalars. OSC from Resolume for swap triggers and the beat clock; MIDI for contrast, hue push, match and stripe (KeyStep knobs) and mode (BeatStep pads).
    - Output at native wall resolution via Spout (for example a third-party plugin such as Off World Live) or NDI.

=== "GODOT"
    - A `ColorRect` fills the viewport with a `canvas_item` shader holding P0. Use `FRAGCOORD.xy` for pixel coordinates. Uniforms: `mode`, `nest`, `rows`, `margin`, palette slots as `vec3` already in OKLab (computed in script), `morph`, `contrast`, `hue_push`, `match_amt`, `stripe_px`, `depth`, `trim_gain`, `black_lift`.
    - Do the linear to sRGB conversion explicitly in the shader and confirm it with the hex round-trip test. Do not rely on uniform colour hints, which may convert implicitly.
    - For authoring palettes in GDScript, [`Color.from_ok_hsl`](https://docs.godotengine.org/en/stable/classes/class_color.html) is handy. OKHSL is a related space, not OKLab, so convert to OKLab in script before passing values to the shader.
    - Palette sampler: a 4 × 1 px `SubViewport` with a second `ColorRect` shader sampling the camera or video texture, read back as a `ViewportTexture`.
    - Window: viewport equals wall resolution, stretch mode disabled, MSAA and FXAA off, no `WorldEnvironment` glow.
    - Control: `InputEventMIDI` for knobs and pads; beat clock from an OSC listener over UDP, or low-band onsets from `AudioEffectSpectrumAnalyzer`.

### Why OKLab

Albers' exercises isolate one variable: change hue and keep lightness, or change lightness and keep hue. That only works in a space where lightness, chroma and hue act as independent perceptual axes, which is the design goal of OKLab ([Björn Ottosson](https://bottosson.github.io/posts/oklab/)). All mixing and easing happens in OKLab; authoring uses OKLCh.

- sRGB averages come out dark and muddy. Rose #C24A6A plus teal #2F7F8C: sRGB mid #78647B, OKLab mid #8A6C7B.
- Yellow #F0C800 plus blue #2E3FA0: the sRGB mid is olive #8F8450. The OKLab mid is the near-neutral #878A84 you expect from complements.
- HSV and HSL lightness is not perceptual, and hue sweeps pass through unrelated hues. CIELAB shifts blue blends toward purple; OKLab stays more even in Ottosson's white-to-blue test. Two of the palettes below are blue-heavy.
- Polar easing is used for one thing only: the `hue push` fader, which rotates hue at fixed lightness and chroma. Palette morphs use straight OKLab lines, so a swap between complements passes through grey, not through a rainbow.

### Palettes

These palettes are original to this piece. L / C / h are OKLab lightness, chroma and hue angle, computed from the hex values with Ottosson's published matrices.

**A. One colour as two.** The figure never changes. It sits between the grounds in lightness (about 0.25 above and below), with its hue on the axis between them. Low-chroma figures (C below 0.04) move the most.

| role | hex | L / C / h | effect |
|------|-----|-----------|--------|
| figure F (fixed) | #8A8270 | 0.61 / 0.03 / 87 | warm grey, never changes |
| ground G1 | #3B2F5C | 0.34 / 0.08 / 294 | deep violet: F reads lighter and yellower |
| ground G2 | #D9B84A | 0.79 / 0.13 / 92 | ochre: F reads darker and greyer-violet |
| alt figure F' | #9A8088 | 0.63 / 0.03 / 358 | rose-grey, second phrase |
| alt ground G1' | #7A1F2E | 0.39 / 0.13 / 16 | wine: F' reads cooler, greener |
| alt ground G2' | #3E8C84 | 0.59 / 0.08 / 186 | teal: F' reads pinker |

**B. Two colours as one.** The figure on the light ground is set lighter and warmer, the figure on the dark ground darker and bluer, to cancel what the grounds take away. The two figures differ by about 0.08 in L and clearly in hue. The `match` control scales that difference from 0 to 1 around their midpoint.

| role | hex | L / C / h | effect |
|------|-----|-----------|--------|
| ground light-warm | #E6CF6A | 0.85 / 0.12 / 97 | pushes its figure darker and more violet |
| figure on light ground | #A8946E | 0.68 / 0.06 / 83 | set lighter and yellower to compensate |
| ground dark-cool | #26325A | 0.33 / 0.07 / 269 | pushes its figure lighter and more orange |
| figure on dark ground | #7B7C8E | 0.59 / 0.03 / 283 | set darker and bluer to compensate |
| reveal surround | #7F7F7F | 0.60 / 0 / n.a. | both grounds cut to this on the reveal |

**C. Homage ramps** (outer to inner). A light centre on a dark surround advances. Reverse the slot order and the centre recedes. Reordering the same four colours is the cheapest big change in the set: no new colour, a new picture.

| ramp | 0 outer | 1 | 2 | 3 inner | L, outer to inner |
|------|---------|---|---|---------|-------------------|
| warm | #7A3B22 | #C9702E | #E8B04A | #F2E3B3 | 0.43, 0.63, 0.79, 0.92 |
| cool | #1F3B45 | #4C7A78 | #8FB3A8 | #DDE6E0 | 0.34, 0.55, 0.74, 0.92 |

**D. Transparency.** Rose #C24A6A (L 0.58) and teal #2F7F8C (L 0.55). The overlap is a third flat colour mixed in OKLab, never a blend mode.

| weight w | overlap hex |
|----------|-------------|
| 0.3 | #A26175 |
| 0.5 | #8A6C7B |
| 0.7 | #6F7582 |

**E. Afterimage.** Discs #F0C800 (L 0.84, C 0.17) on #2E3FA0 (L 0.42, C 0.16). Release field #7F7F7F. Fixation dot #121214.

**F. Bezold.**

| field | stripes |
|-------|---------|
| #B8322A (L 0.52) | #F2EEE6 (L 0.95) versus #141414 (L 0.19) |
| #2F5FA8 (L 0.49) | #E6CF6A versus #141414 |

**Frame.** Off-white #D6D2C8 for stage, or #121214 for club. Pure #FFFFFF frames glare on LED walls and pull the eye from the cells. Darkest content value is #121214 (L 0.18), peak white at most #F4F1EA.

### Techniques in short

- **One as two.** Split mode: left half ground G1, right half G2, same F in both. A thin bridge bar (2 % of cell height) in F can join the two figures to prove they are one colour. Flip mode: one ground for the wall, swapped on the beat.
- **Two as one.** The reveal: on the drop, cut both grounds to #7F7F7F and the "identical" figures split apart. Tune `match` by eye on the actual wall.
- **Bezold.** Stripe duty 30 to 40 %. Size stripes for viewing distance: `stripe_mm ≈ 0.29 × distance_m × arcmin`, aiming for 3 to 8 arcmin. Example: a P3.9 wall seen from 15 m at 4 arcmin gives 17 mm, about 4 px.
- **Transparency.** Sweep `depth` w from 0.2 to 0.8. Which sheet seems to lie on top changes along the sweep. Find the flip point by eye and put it on a beat.
- **Afterimage.** The source examples use 25 to 30 s and 5 to 60 s of staring. At 128 BPM, 16 bars is about 30 s. Hold with a still fixation dot, then hard-cut to #7F7F7F for 1 to 2 bars. Large images and a steady eye help, so fill the wall. Announce nothing.

## On stage

**Audio.** Audio picks the moments; colour values stay fixed between them. Continuous modulation of lightness or hue would make every field shimmer and erase the comparison the eye needs.

| band | drives | notes |
|------|--------|-------|
| kick, 40 to 100 Hz | ground swap trigger, in builds only; otherwise quantised to the bar | attack 5 ms, release 150 ms, max one swap per 1/8 note |
| bass, 60 to 250 Hz | in Homage mode, a bar-long average picks "advance" (light centre) or "recede" order | no continuous drive |
| mid, 300 Hz to 2 kHz | optional: stripe width over a build, wide to narrow | attack 500 ms, release 2 s |
| hi-hat, 6 to 12 kHz | bridge bar on/off in One-as-two; Bezold stripe colour step | max one per 1/8 note |
| silence | arms Afterimage mode; the drop onset fires the release | full-band level under -24 dBFS for 2 bars |

**MIDI, KeyStep 37 and BeatStep Pro.**

| control | range | default | mapping |
|---------|-------|---------|---------|
| swap | trigger plus rate (off, bar, 1/2, 1/4, 1/8) | bar | BeatStep Pro pad for manual swap; rate on a knob, quantised to the clock |
| contrast | 0 to 1 | 0.7 | fader; raise for projectors and big rooms |
| hue push | -60 to +60 degrees | 0 | KeyStep knob; never audio-driven |
| match | 0 to 1 | 1.0 | knob, set at soundcheck from the audience area |
| stripe | 2 to 40 px | 4 px | fader; automate from wide to narrow over a build |
| morph | 0 (cut) to 1 bar | 0 | knob; 0 for most of the set, longer in breaks |

Pads, not faders: mode (exercise), nest (3, 4 or grid), reverse slot order, fixation dot, afterimage release.

**Set order and timing.** Nothing glides. Changes are cuts on musical positions, held for at least 1 bar, typically 2 to 4. Change exercise and palette per 16-bar phrase. A suggested run: Homage warm, One-as-two (A), Bezold, Two-as-one (B, reveal on the drop), Transparency, Grid, Afterimage (break, released on the drop).

- **Build (8 bars).** Swap rate doubles every 2 bars (bar, half, quarter, eighth). `stripe` narrows so the field colour seems to drift. Grid cells go from 1 to 4 to 16.
- **Drop.** Two proof moments: the afterimage release (16-bar hold, cut to grey for 1 to 2 bars, then full-colour Homage) or the two-as-one reveal.
- **Break.** Afterimage hold with fixation dot, or One-as-two held still with the bridge bar on, hi-hats toggling it.
- **Safety.** Hard full-wall swaps between very light and very dark grounds at 1/8 note are close to strobe territory. Keep fast rates to short bursts (2 bars) and keep the L difference of full-wall swaps under 0.5 above 1/4 note.

**Layering.** Keep the Albers layer opaque and at the bottom. Host-side blend modes and opacity fades mix in the host's space, not OKLab, and change exactly the values the illusion depends on. Never crossfade two clips with different palettes; morph inside the shader. Type, particles or feedback from other chapters may sit in the frame area outside the cells, Normal blend, 100 %. Over the cells they destroy the effect. The cell shape can also act as a luma-key matte on a separate output.

**LED walls and projectors.** These are practical guidelines; verify on site.

- *Gamut.* LED walls are often wider than sRGB. Without gamut mapping, ochres and teals oversaturate and near-neutral figures (C 0.03) pick up a tint, which breaks One-as-two. Set the processor to a Rec.709 or sRGB target.
- *White point.* Walls are often set cool, which turns #8A8270 toward grey-blue. Trim with `trimGain` until the two halves of the split mode look as different as possible while the bridge bar still looks like one colour.
- *Dimming and bit depth.* Dimming in content throws away 8-bit codes, and low-grey steps on LED drivers are coarse and can tint. Dim at the processor and keep content at designed values.
- *Cabinet variance.* Neighbouring cabinets can differ in low greys, and the figures differ by only 0.05 to 0.1 in L. Keep compared figures on the same cabinet row where possible.
- *Viewing angle.* Colour shifts off-axis. Tune `match` from a typical seat, not only from FOH centre.
- *Projectors.* Projector black plus room light lifts dark grounds, so #26325A and #3B2F5C turn grey-blue. Raise figure contrast, prefer the light-ground exercises and use the dark frame. DLP white boost brightens whites more than colours. Coloured walls tint everything; measure with a white field first.
- *Calibration plate.* A hidden mode 7 shows all slots of the active palette as equal chips on #7F7F7F, plus the split-mode test. Tune trims by eye from the audience area. Then run the hex round-trip test: pick the output colour from the Spout or NDI feed; #8A8270 must come out as #8A8270. A lighter or darker value means a double or missing sRGB encode.

**Performance tips.**

- Render at the native pixel map, 1:1. Any scaling turns hard edges into blended seams, and those seams are new colours.
- Squares must stay square. On 16:9, fit cells to the short side: one 1080 px cell plus two 420 px frame bands, or two rows of 540 px cells. Never stretch the schema.
- Snap edges to integer pixels. With margin 0.1 and S = 1152 px, edges land at 57.6 px; unsnapped edges flicker between two columns during morphs.
- Cost is trivial: about 40 to 60 ALU per pixel with gamut mapping. Precompute palettes in OKLab on the CPU and spend the GPU budget elsewhere.
- Pitfalls: bloom or glow anywhere in the chain, a tonemapper or auto exposure in Unreal, double sRGB encoding, processor gamut mapping off, content-side dimming, an afterimage hold shorter than about 16 bars, and swapping on every kick for minutes.
- The boring risk is static colour on a big screen. The fix is a strong swap timed to the music, with held pauses so the illusion has time to work.

## Variations

1. **One colour, two lives.** The same middle colour held fixed while the ground flips on each kick.
2. **Palette from a camera or Spout feed.** Sample four colours from another layer, sort by lightness, and re-order them in the Albers schema.
3. **Grid of Homages.** Dozens of small nested-square cells, each with its own palette, swapped in waves across the grid. Each cell still cuts hard; the wave is in the timing, not in a fade.
4. **Afterimage mode.** Hold, then flash to neutral grey on the drop.
5. **Bezold mode.** A fixed stripe pattern whose colour shifts as one added colour runs across it.

## Did you know?

- The Homage to the Square series ran from 1950 to 1976 and has over 1,000 works ([Wikipedia](https://en.wikipedia.org/wiki/Homage_to_the_Square)).
- Albers painted on the rough side of Masonite panels with at least six coats of white primer ([Wikipedia](https://en.wikipedia.org/wiki/Homage_to_the_Square)).
- Most plates in *Interaction of Color* were made by his students ([Albers Foundation](https://www.albersfoundation.org/alberses/teaching/interaction-of-color)).
- The 2013 iPad app had more than 75,000 downloads ([Communication Arts](https://www.commarts.com/project/22915/interaction-of-color)).

## Sources

- [Albers Foundation, Interaction of Color](https://www.albersfoundation.org/alberses/teaching/interaction-of-color)
- [Wikipedia: Josef Albers](https://en.wikipedia.org/wiki/Josef_Albers)
- [Wikipedia: Homage to the Square](https://en.wikipedia.org/wiki/Homage_to_the_Square)
- [Wikipedia: Simultaneous contrast](https://en.wikipedia.org/wiki/Simultaneous_contrast)
- [Wikipedia: Bezold effect](https://en.wikipedia.org/wiki/Bezold_effect)
- [Wikipedia: Afterimage](https://en.wikipedia.org/wiki/Afterimage)
- [Yale News, app](https://news.yale.edu/2013/07/25/art-book-ipad-app-josef-albers-classic-work-undergoes-magical-transformation)
- [Communication Arts, app](https://www.commarts.com/project/22915/interaction-of-color)
- [Albers Foundation, digital edition](https://www.albersfoundation.org/news/yale-press-launches-the-interaction-of-color-complete-digital-edition)
- [Albers Foundation, Logic and Magic of Color](https://albersfoundation.org/alberses/teaching/josef-albers/logic-magic-of-color)
- [The Marginalian](https://www.themarginalian.org/index.php/2013/08/16/interaction-of-color-josef-albers-50th-anniversary/)
- [Cassiel, HTTS](https://cassiel.com/project/htts)
- [AutoAlbers](https://www.github.com/y-a-v-a/autoalbers)
- [An iconic book reimagined (YouTube, Yale University Channels)](https://www.youtube.com/watch?v=H-hFOZ3XdIQ)
- [Björn Ottosson: A perceptual color space for image processing](https://bottosson.github.io/posts/oklab/)
- [Resolume ISF support](https://resolume.com/support/en/isf)
- [Godot Color class](https://docs.godotengine.org/en/stable/classes/class_color.html)
- [Wikimedia Commons: Simultaneous Contrast.svg](https://commons.wikimedia.org/wiki/File:Simultaneous_Contrast.svg)
