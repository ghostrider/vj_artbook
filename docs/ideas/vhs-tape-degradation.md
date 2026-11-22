---
tags:
  - look
  - vhs
  - analog-video
  - retro
  - glitch
---

# VHS Tape Degradation

> Any image, played back as a worn rental tape that is slowly dying on a tired deck.

![A photographed television glitch with static-like colour patterns](https://images.unsplash.com/photo-1610337673044-720471f83677?w=1400)
*Similar look, not the original work. Photo: [Michael Dziedzic](https://unsplash.com/photos/colorful-digital-glitch-static-pattern-0W4XLGITrHg) on [Unsplash](https://unsplash.com/license).*

## The idea

This look treats any source as if it had been recorded to VHS and played back after years of wear. Colour smears to the right of every edge. Brightness stays sharper but low-res. A noisy band rolls down the frame and tears the picture sideways. White flecks flash on single lines.

It comes from the physics of the format. A spinning two-head drum writes diagonal tracks on magnetic tape, luma is stored frequency-modulated, and chroma is squeezed into a low "color-under" band ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)). Wear, stretching and misalignment then add dropouts, tracking errors and noise. As a choice, it is a retro move: Rozenkrantz describes an "age of retrospectacle" in which simulating analogue video is a dominant visual trend ([Alphaville, Issue 12](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html)).

On a club screen it works as memory. The picture looks like a 1990s night out that is physically decaying while you watch it. It suits bright, saturated footage: live camera, clips, or Unreal scenes arriving over Spout or NDI. Clean, high-contrast sources give the bleed and halos something to smear. Its one risk is standing still. A constant filter reads as a stock overlay, so the look needs a wear curve that moves across the set.

## Origins and references

- **JVC VHS**: introduced on 9 September 1976; the first recorder, the HR-3300, went on sale in Japan on 31 October 1976 ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)).
- **Mark Leckey, *Fiorucci Made Me Hardcore* (1999)**: a 15-minute video built only from found tape footage of UK dance floors from the 1970s to the 1990s, first shown at the ICA London ([Wikipedia](https://en.wikipedia.org/wiki/Fiorucci_Made_Me_Hardcore), [Frieze](https://frieze.com/article/mark-leckeys-iconic-video-fiorucci-made-me-hardcore-returns-youtube-after-copyright-claim)). Rozenkrantz discusses it as analogue-video aesthetics ([Alphaville](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html)).
- **Chase & Status, "Blind Faith", dir. Daniel Wolfe (2010)**: a staged rave recreation shot to pass as faux home video of a 1990 night out in Manchester, so the degraded look is a costume for new footage ([Promonews](https://www.promonews.tv/videos/2010/12/08/chase-status-feat-liam-bailey%E2%80%99s-blind-faith-daniel-wolfe/7134)). Rozenkrantz also lists Joey Bada$$ "Fromdatomb$" (2012) and the film *Kung Fury* (2015), where tracking errors and dropouts are placed on purpose ([Alphaville](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html)).
- **Nam June Paik, Bruce Nauman, David Hall**: Apollo names Paik's *TV Buddha* (1974), Nauman's *Clown Torture* (1987) and Hall's *1001 TV Sets (End Piece)* (2012), plus glitch artist Rob Feulner, as work with the medium's analogue openness ([Apollo](https://apollo-magazine.com/nostalgia-vhs-television-video-art/)).
- **DecayingElectronics, VHS Dirty Video Mixer**: a hardware box that takes two RCA video inputs (VHS player, console) and glitches them with a knob ([Boing Boing](https://boingboing.net/2021/05/31/glitch-video-the-hardware-way-with-the-vhs-dirty-video-mixer.html/amp)).
- **Pushkar Brand**: a VHS look driven live from Resolume Arena, with analogue glitch synced to Eurorack (see below).

## Visual language

- **Colour**: soft and smeared, trailing to the right of edges. Luma is sharper but low-res, with a faint halo from deck sharpening. Blacks are lifted to a milky violet-grey, highlights are warm off-white. Red-magenta and cyan fringes sit on saturated edges.
- **Structure**: scanlines, slight ghosting, a 4:3 feeling even in 16:9.
- **Motion**: each line wobbles a pixel or two. Tracking bands scroll down the frame and tear it. The bottom 3 % is always a skewed, noisy head-switching strip. Dropout flecks flash on single lines.
- **Crawl**: tiny checkerboard dots and rainbow fringes on colour edges and fine detail ([Wikipedia: Dot crawl](https://en.wikipedia.org/wiki/Dot_crawl)).
- **Cues for "the past"**: Rozenkrantz lists low resolution, dropout, tracking lines and rainbow effects ([Alphaville](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html)).
- **Mood**: nostalgic and a little haunted.

## See it

- [Pushkar Brand, Honey Breadcrump](https://www.youtube.com/watch?v=P9Jf11zMSoo) — VHS look made in Resolume Arena with analogue glitch synced to Eurorack.
- [Pushkar Brand, Minor Lights Ahead](https://www.youtube.com/watch?v=1YwEu6xkzRk) — video-bent VHS glitch clip.
- [VHS Tracking Effect Tutorial (After Effects)](https://www.youtube.com/watch?v=binY4qJwVto) — tutorial for the tracking-band look.
- [Godot Shaders: VHS Glitch](https://godotshaders.com/shader/vhs-glitch/) — shader page with a preview of bar shifts, line noise and channel offset.
- [TD-VHShader on GitHub](https://github.com/ogrew/td-vhshader) — TouchDesigner project of 12 combined VHS shaders.
- [Boing Boing: VHS Dirty Video Mixer](https://boingboing.net/2021/05/31/glitch-video-the-hardware-way-with-the-vhs-dirty-video-mixer.html/amp) — hardware glitch device and demo.

## Code design

The effect is one multi-pass treatment at a small work buffer (about 854×480 for 16:9), placed last in the chain. Order matters. First a time-base pass displaces lines (jitter, tracking band, head-switch skew). Then RGB is split into YIQ. Luma gets a horizontal blur and an unsharp halo. Chroma gets a wide, one-sided blur (about ten times the luma radius, matching the bandwidth ratio of roughly 300 kHz to 3 MHz), a shift to the right and some saturation loss ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)). Optional dot crawl and rainbow come from a subcarrier pattern. Noise, dropouts, snow and a ghost follow. The final pass upscales to wall resolution, adds scanlines and a vignette. An optional feedback buffer gives generation loss.

A single "wear" macro scales noise, dropouts, bleed, saturation loss and black lift together. Other stage parameters are tracking, bleed, jitter, generations and a one-shot chew trigger. Keep all blurs and offsets relative to roughly 480 lines, and use 16-bit float buffers so halos and negative chroma survive between passes.

=== "Resolume Wire"
    Build an Effect patch: Effect Input, resize to work resolution, four GLSL Shader nodes (time base, encode and luma, chroma, composite and damage), resize to output, a fifth GLSL Shader for scanlines and vignette, Effect Output. A Feedback node takes the damage pass back into the first mix for generation loss. A Time node feeds all shaders. A BPM/Beat node drives the band period and an attack-0, 150 ms decay envelope for jitter spikes. A trigger input drives the chew envelope. Expose wear, tracking, bleed, jitter, generations, chew and a "clear tape" reset as patch inputs. Remap nodes let wear drive noise, dropout rate, bleed, saturation and black floor together. Node names vary between Wire versions; the connection order is what matters.

    Quick path without custom code: on the Composition stack Reducto, Shifty (small amount, smoothing on), VHSifyer, a low Blur and Vignette, plus Screen Shake synced to BPM as the kick tear. Resolume introduced Reducto, Shifty, VHSifyer and Screen Shake as built-in effects ([Resolume blog](https://www.resolume.com/blog/23113)). This gets roughly 60 % of the look, without one-sided chroma bleed or a tracking band. The full path is a custom ISF effect: Resolume reads ISF shaders ([Resolume: ISF](https://resolume.com/support/en/isf)), and ISF supports multi-pass and persistent buffers.
=== "Unreal Engine 5.7"
    Use a Post Process Material (Blendable Location After Tonemapping) in an infinite-extent Post Process Volume, reading SceneTexture:PostProcessInput0. Compute the line index from ScreenPosition and Floor, add noise-driven offsets to the UV, and run 16-tap horizontal blurs in a Custom node. Do the YIQ conversion with three Dot nodes. Damage uses a tiled 256² noise texture, with Floor(Time × 30) as the frame index. For generation feedback, a Blueprint draws the material to a render target (RT_A) each tick, reads RT_B and swaps them. Parameters live in a Material Parameter Collection (`MPC_VHS`) bound to a Remote Control Preset over OSC and MIDI. Output via Spout (Off World Live plugin) or NDI into Resolume. Alternatively, send the scene clean into Resolume and apply one tape treatment there for all sources.
=== "GODOT"
    In Godot 4, a CanvasLayer holds a full-rect ColorRect with a `canvas_item` shader reading `hint_screen_texture` for a single-pass version. For multi-pass, chain SubViewports at work resolution, each reading the previous ViewportTexture. A second SubViewport with `render_target_clear_mode = Never` holds last frame for feedback. A final TextureRect with linear filtering runs the scanline shader. Drive uniforms (`wear`, `tracking`, `bleed`, `jitter`, `generations`, `chew`, `band_period`) from GDScript reading `InputEventMIDI`. The [Godot Shaders VHS Glitch](https://godotshaders.com/shader/vhs-glitch/) page is a starting point for the bar shift and channel offset.

## On stage

**Chain position**: last. Put it on the Composition or a top group so every layer below inherits the tape. Post order inside the chain is upscale, scanlines (Multiply 10-25 %), light bloom, vignette. Add no extra grain and no output sharpen; both already live inside the effect.

**Audio reactivity**

| band | drives |
|------|--------|
| Kick | jitter spikes to 1.0 (attack 0, release 150 ms), head-switch skew x1.5: a short horizontal tear |
| Bass | bleed +0 to 15 px, slow line wobble |
| Snare / clap | one dropout burst (8 % of lines for 2 frames) |
| Hi-hats | noise and dot crawl, gated at -30 dB so silence stays clean |
| Overall loudness | wear offset of about 0.1, very slow (2 s attack, 6 s release) |

Do not drive tracking from audio. It turns into nervous noise. Keep it on the bar: one band pass per bar (`bandPeriod = 60 / BPM * 4` s), with its strength breathing over 8 bars.

**MIDI mapping**

| control | parameter |
|---------|-----------|
| KeyStep 37 mod strip (or a fader) | wear, with slow automation across the set |
| BeatStep Pro knob 1 | tracking |
| BeatStep Pro knob 2 | bleed |
| Fader | generations, raised in breaks |
| BeatStep Pro pad | chew, the one-shot tape-eating glitch (0.5 to 1 bar) |

**Transitions**: fire chew on the last beat before the drop, then hit the drop clean-ish (wear -30 %) for one bar and let wear creep back. Contrast sells the drop. In a break, raise wear by 0.2 over 8 bars, push generations to 0.4, and freeze the frame on the one for a paused-tape look with heavy jitter.

**Tips**
- Work buffer stays near 480 lines. Never run the blurs at 4K; if you must, scale every pixel value by H / 480.
- On LED walls keep the black floor at 5-7 % (lifted blacks read as bright grey in a dark room) and pull output to about 85 % so dropout flashes do not blind the front row. Cap dropout luma at 0.9.
- If scanlines beat with the LED pixel pitch, use an integer divisor of the wall height or switch them off.
- Clear the feedback buffer on clip changes, or old content leaks for seconds.

## Variations

- **Wear arc**: map tape age to one macro, from fresh rental to chewed tape over a track or a set.
- **Generation loss**: feed the output back through the filter several times for copy-of-a-copy decay.
- **Pitch-coloured bleed**: let the MIDI note choose the hue direction of the chroma smear.
- **Selective tape**: apply the effect through a mask (a Wire or UE shape), so a clean image shows through a degraded window. Feather the edge horizontally so it smears too.
- **Warm or cold tape**: nudge I (towards orange) or Q for a tinted cast.
- **Hybrid hardware pass**: run the output through a glitch device such as the Dirty Video Mixer and re-capture it.
- **Pre-rendered**: [ntsc-rs](https://github.com/valadaptive/ntsc-rs), an open-source Rust NTSC/VHS emulator, can bake clips offline and shows pass order and parameter ranges.

## Did you know?

- VHS chroma bandwidth is about a tenth of luma: roughly 300 kHz against up to 3 MHz ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)).
- The VHS drum spins at 1,798.2 rpm for NTSC and 1,500 rpm for PAL, one revolution per frame ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)).
- Dot crawl is mostly associated with NTSC. Digital comb filters in the 1990s greatly reduced it, but none removes all NTSC artefacts ([Wikipedia: Dot crawl](https://en.wikipedia.org/wiki/Dot_crawl)).
- "Blind Faith" looks like archive footage but is a recreation ([Promonews](https://www.promonews.tv/videos/2010/12/08/chase-status-feat-liam-bailey%E2%80%99s-blind-faith-daniel-wolfe/7134)).
- Leckey's *Fiorucci Made Me Hardcore* had over 203,000 YouTube views before a copyright claim briefly removed it ([Frieze](https://frieze.com/article/mark-leckeys-iconic-video-fiorucci-made-me-hardcore-returns-youtube-after-copyright-claim)).

## Sources

- [Apollo: nostalgia, VHS, television, video art](https://apollo-magazine.com/nostalgia-vhs-television-video-art/)
- [Wikipedia: Dot crawl](https://en.wikipedia.org/wiki/Dot_crawl)
- [Rozenkrantz, Analogue Video in the Age of Retrospectacle (Alphaville 12)](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html)
- [Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)
- [AV Artifact Atlas: Tracking error](https://www.avartifactatlas.com/artifacts/tracking_error.html)
- [Wikipedia: Fiorucci Made Me Hardcore](https://en.wikipedia.org/wiki/Fiorucci_Made_Me_Hardcore)
- [Frieze: Fiorucci Made Me Hardcore returns to YouTube](https://frieze.com/article/mark-leckeys-iconic-video-fiorucci-made-me-hardcore-returns-youtube-after-copyright-claim)
- [Promonews: Blind Faith by Daniel Wolfe](https://www.promonews.tv/videos/2010/12/08/chase-status-feat-liam-bailey%E2%80%99s-blind-faith-daniel-wolfe/7134)
- [Boing Boing: VHS Dirty Video Mixer](https://boingboing.net/2021/05/31/glitch-video-the-hardware-way-with-the-vhs-dirty-video-mixer.html/amp)
- [Godot Shaders: VHS Glitch](https://godotshaders.com/shader/vhs-glitch/)
- [Derivative: TouchDesigner glitch effect GLSL](https://derivative.ca/community-post/tutorial/touchdesigner-glitch-effect-glsl/73842)
- [TD-VHShader](https://github.com/ogrew/td-vhshader)
- [Resolume blog: 9 new effects](https://www.resolume.com/blog/23113)
- [Resolume: ISF](https://resolume.com/support/en/isf)
- [ntsc-rs](https://github.com/valadaptive/ntsc-rs)
