---
tags:
  - glitch
  - video
  - compression
  - datamosh
---

# Datamosh Compression Glitch

> Break the codec on purpose, and one picture wears another's motion.

![A real video decoding error in a 4K player (VLC): blocks of different shapes and colours. An accidental compression glitch, not an art piece.](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e7/Decodierfehler_Videowiedergabe_4K-Video_20230824.png/1280px-Decodierfehler_Videowiedergabe_4K-Video_20230824.png)
*A real video decoding error in a 4K player (VLC): blocks of different shapes and colours. An accidental compression glitch, not an art piece. Image: [PantheraLeo1359531](https://commons.wikimedia.org/wiki/File:Decodierfehler_Videowiedergabe_4K-Video_20230824.png), [Public domain (marked ineligible for copyright, machine output)](https://commons.wikimedia.org/wiki/File:Decodierfehler_Videowiedergabe_4K-Video_20230824.png).*

## The idea

Video codecs store an I-frame, a full image like a keyframe. After it come P-frames, which only store changes from the previous frame, and B-frames, which predict from earlier and later frames. Remove or corrupt an I-frame and the decoder applies the new scene's motion data to the old scene's picture. Old pixels move the way the new shot moves: colour melts, ghosts linger, motion smears ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).

Datamoshing began as accident. Early codecs such as DivX produced glitches after signal loss or faulty encoding, and in the early 2000s artists started triggering them on purpose ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing), [New Atlas](https://newatlas.com/art-technology-digital-datamoshing/48152)). Two classic moves: I-frame removal makes the old clip bleed into the next, a morph-like transition. P-frame duplication repeats a P-frame so its motion compounds into bloom and trails ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).

On a club screen it works as a transition and as decay. A kick can cut the scene while the old picture stays and gets dragged around. The look is hard block edges next to soft melting colour. The risk is mush: constant full-strength smear gets boring, so it needs reset moments and moving, high-contrast source material.

This chapter gives two routes. **Route A** is real datamosh, made offline, baked and played as ordinary clips. **Route B** is a real-time emulation that needs no codec, so any live input can be moshed and the kick can trigger it.

## Origins and references

- [Takeshi Murata, *Monster Movie* (2005)](https://www.eai.org/titles/monster-movie/video-intro): 3:55, built frame by frame from footage of the 1981 film *Caveman*, which dissolves into churning colour and shape. Sound by Plate Tectonics.
- [Takeshi Murata, *Untitled (Pink Dot)* (2007)](https://landmarks.utexas.edu/video-art/takeshi-murata): made from *First Blood*. Also *No Match* (2009), a game-show clip stretched to 999 seconds.
- [Chairlift, "Evident Utensil" (2009)](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/), dir. Ray Tintori: the band breaks through the frame in a mosh. Murata is named as a key influence.
- [Kanye West, "Welcome to Heartbreak" (2009)](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/), dir. NABIL, post by Ghost Town Media: a grittier mosh. The team rendered uncompressed, compressed to AVI and deleted chosen I-frames.
- Early pioneers named by [Wikipedia](https://en.wikipedia.org/wiki/Compression_artifact): Bertrand Planes and Christian Jacquemin (DivXPrime, 2006), Sven König, Takeshi Murata, Jacques Perconte, Paul B. Davis (Paperrad). The [Motionographer article](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/) also lists David O'Reilly, Owi Mahn and Laura Baginski, and Kris Moyes.
- [Datamoshing with Optical Flow](https://yaksoy.github.io/datamosh) (Careaga, Reddy, Aksoy, SIGGRAPH Asia 2023 poster): accurate optical flow on mismatched content gives moshing without codec blocks.

## Visual language

A clean clip plays, the kick hits, the scene cuts, but the old picture stays. A face from the last shot is dragged across the frame by the new shot's movement. Whole 16-pixel squares of skin and jacket slide sideways, stretch into stripes at the frame edge and pile up where the new motion stops. Where the new clip moves fast, colour pulls away from shapes and runs ahead of them. Where nothing moves, the old image hangs on like a ghost.

On a held note one movement repeats and grows: a head turn becomes a blooming smear that swallows the frame. On the drop the picture snaps back to sharp and clean, a fresh "keyframe", and the next mosh starts from there.

Mood: digital decay. A few blocks of decode-error green, blacks that stay black. No blur, no bloom, no grain: they would kill the block edges, which are the whole point.

Palette: the colour comes from the sources. The design only adds accents and a stage grade.

| role | hex | use |
|------|-----|-----|
| base / black | #000000 | background, crushed blacks |
| shadow | #0B0B12 | floor of the grade |
| decode green | #008700 | corrupt-block bursts (what zeroed YCbCr data decodes to) |
| chroma magenta | #FF2BD6 | optional tint on chroma slip |
| chroma cyan | #00E5FF | partner of magenta for the slip split |
| highlight | #F2F2F2 | peak white limit |

## See it

- [Takeshi Murata at EAI: Monster Movie](https://www.eai.org/titles/monster-movie/video-intro) — distributor page for the 2005 work.
- [Motionographer: Tintori and NABIL](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/) — article with stills and links to both 2009 videos and the artist works.
- [huff](https://schwwaaa.github.io/huff/) — real-time datamosh app, shows the live-tool look.

## Code design

The model behind both routes: a codec builds each frame as "previous frame, blocks moved by their vectors, plus a small residual". Delete the I-frame at a cut and the new vectors act on the old picture. Repeat one P-frame and the same vectors compound into a bloom ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).

**Route A, prepared clips (the robust gig option).**

1. Encode the source clips without B-frames, so each frame depends only on the previous one. The open-source [datamoshing](https://github.com/tiberiuiancu/datamoshing) scripts do this with ffmpeg into AVI with `-bf 0` ([mosh.py](https://raw.githubusercontent.com/tiberiuiancu/datamoshing/master/mosh.py)).
2. Put clip A and clip B back to back, then remove the I-frames from the cut onwards. B's motion now runs on A's last picture.
3. For blooms, use P-frame repeat. A repeat block of 1 to 4 frames gives a steady push. 15 to 30 gives a rhythmic looping smear.
4. For finer control, edit the motion vectors with [FFglitch](https://ffglitch.org/). Its [averaging script](https://ffglitch.org/2024/06/mv_avg_010.html) turns jitter into smooth liquid pushes. Avidemux 2.7.0+ can copy and paste P-frames by hand ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).
5. Bake the result: decode the broken file once and convert it to DXV in Alley with "DXV Normal Quality, No Alpha" ([Resolume: Alley](https://resolume.com/article/81)). Never give Arena the broken AVI. The mosh only exists in the decoder, and every decode can differ. The Kanye video needed about 45 to 50 renders for that reason ([Motionographer](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)).
6. Render many versions per transition, keep the best 5 to 10, and trim each to whole bars (4 bars at 128 BPM is 7.5 s).

**Route B, real-time emulation.** Five passes, P0 to P4:

- **P0 luma downsample** to quarter resolution (480x270 for 1080p), keeping the previous frame.
- **P1 motion estimation**: one vector per 16x16 macroblock. Block matching tests offsets in a +-6 pixel window at quarter res and scores them by sum of absolute differences. It prefers zero motion in flat areas, as encoders do. Without that, sky and black fill with random vectors. A gradient optical-flow variant gives a liquid look instead of blocks.
- **P2 motion shaping**: gain, hold, smoothing (`tail`), audio push, jitter.
- **P3 mosh update**: a persistent buffer, `M_new(p) = M_old(p - mvBlock(p))`. All pixels of a block move together, which gives hard block edges. Chroma slip samples colour at a larger offset, so colour runs ahead of shapes. Intra refresh pulls random blocks from the live image. A keyframe event copies the clean source for one frame.
- **P4 composite**: optional decode-green blocks, wet/dry mix, black crush.

Hold copies the motion field and keeps applying it, which is the P-frame repeat. Releasing hold fires a keyframe. Optical-flow distortion for datamosh has precedent: [Andrew Benson's 2009 Jitter/GLSL patch](https://cdm.link/2009/03/liquidify-video-live-optical-flow-glsl-datamosh-technique/), and [Buck](https://buck.co/sandbox/moshing) added block structure on top of a flow-driven feedback loop.

Use either Route A or Route B per song section. Both at once turns into mush.

=== "Resolume Wire"
    **Route A in Arena.** Layer 1 holds the clean clips, layer 2 the baked mosh clips. A mosh clip's first frame must match the clean clip that played before it. Name clips like `A>B_mosh_4bar_v3`. Set clip transport to BPM sync with the rendered length in beats, and launch on the downbeat. At its end, trigger the clean target on layer 1 and clear layer 2. DXV is decoded on the GPU ([Resolume DXV](https://www.resolume.com/software/codec)), so baked moshes stay cheap.

    **Route B in Wire.** Node chain: texture input (source A), optional second input as motion source B, then ISF nodes P0 luma, P1 motion, P2 shape, P3 mosh, P4 composite, then output. Since 7.11 the Buffer and Ring Buffer nodes accept textures, which allows one-frame delays ([Resolume 7.11](https://resolume.com/blog/21445)). Use one for the previous luma, one for the mosh feedback, one for the held vectors. Resolume documents ISF only inside Wire, and Wire 7.22 does not support vertex shaders ([Resolume: ISF](https://resolume.com/support/en/isf)). The design only uses fragment passes. ISF event inputs fit keyframe and hold. A search found no built-in optical flow node in Wire, so P1 is a custom shader.

    Exposed parameters: mosh, hold, keyframe, block, refresh, source swap, wet. Control side: a beat/bar counter for keyframes, audio low band for kick and gain, high band for refresh bursts. Node names differ between Wire versions, so check the node browser.

    Two alternatives to a pure Wire patch: run [huff](https://schwwaaa.github.io/huff/) (paid; displaces tiles from a 60-frame ring buffer; MIDI CC/notes, OSC on UDP 9000, Spout out) as a Spout source, treated as a 720p layer because the page says Spout goes through the CPU and is reliable at 720p/30 fps. Or use TouchDesigner over Spout, with the Optical Flow TOP for vectors (needs an Nvidia RTX 3000 or newer, Windows only, per the [Derivative docs](https://derivative.ca/UserGuide/Optical_Flow_TOP)).

    **Test before the gig:** that a persistent ISF buffer keeps its content between frames in your Wire version. If not, use the Buffer node for the ping-pong.

=== "Unreal Engine 5.7"
    Use it to mosh a rendered 3D scene (move the camera to make the motion) or a Media Texture / NDI input.

    - Render targets: luma A/B 480x270 R16F, motion 120x68 RG16F (nearest filter), held motion 120x68, mosh A/B 1920x1080 RGBA8. Ping-pong by a Blueprint bool.
    - Input: Scene Capture 2D with "Final Color (LDR) in RGB", or the Media Texture.
    - Per Tick, Draw Material to Render Target in order: luma, motion (block search in a Custom node, or the gradient flow in standard nodes), shape, mosh (reads the other parity), composite. All Unlit materials.
    - Optional: SceneTexture Velocity can replace P0 and P1 for camera and object motion. Check its encoding and sign in your build first. A commercial option is [Petyu's Datamosh plugin](https://petyu.itch.io/uedatamosh), listed up to UE 5.7.
    - Niagara is optional: a GPU sprite system sampling the motion target can spawn square debris on high-motion blocks.
    - Remote Control: a Material Parameter Collection with Gain, Hold, Block, Refresh, Chroma, Melt, Wet. Keyframe and SourceSwap as Blueprint functions. Expose in a Remote Control Preset, bind to OSC and MIDI.
    - Output: the final render target through Spout or NDI to Arena at 1920x1080, 60 fps.

=== "GODOT"
    Godot 4, 2D only, Compatibility or Forward+. All SubViewports use `UPDATE_ALWAYS` ([Godot docs](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)).

    - `Luma` (480x270): the source with a luma `canvas_item` shader. `LumaPrev` shows Luma's texture, one frame late.
    - `Motion` (120x68): block-match shader reading both.
    - `MotionHeld` (120x68, clear mode never): shows Motion. A script switches it to `UPDATE_DISABLED` while hold is on, so it keeps the last vectors.
    - `Mosh` (1920x1080, clear mode never) runs the P3 shader reading `MoshPrev`. `MoshPrev` shows Mosh's texture. This gives one-frame feedback without a viewport reading itself.
    - Final: a full-screen ColorRect with the P4 shader.
    - Source: VideoStreamPlayer (Theora) or CameraTexture. Audio: AudioEffectSpectrumAnalyzer on the music bus.

    Godot does not guarantee viewport draw order, so the copy may lag one extra frame. That only scales mosh speed slightly. Compensate with gain.

## On stage

**Audio reactivity**

| band | drives |
|------|--------|
| kick, 40-120 Hz | keyframe trigger (every n-th kick) |
| bass, 60-250 Hz | gain +0 to +1, and a push along a fixed direction (default left) |
| sustained bass or pad | hold on; release on the drop |
| hats, 6-12 kHz | refresh burst for 2 frames, block jitter one size up for 1 frame |
| snare | green blocks for 3 frames, fills only |

Kick keyframes and bar-grid keyframes are mutually exclusive. Pick one per section, or the image never decays.

**MIDI mapping (KeyStep 37 / BeatStep Pro)**

| control | maps to |
|---------|---------|
| BeatStep Pro knob 1 | mosh (gain) |
| KeyStep sustain or a BeatStep pad (gate) | hold; release fires a keyframe |
| kick, or pad 1 | keyframe; a knob picks every 1/2/4/8 kicks |
| knob 2 (stepped) | block size 16 / 32 / 64 px |
| knob 3 | refresh; hats burst it to 0.08 |
| pad 2 | source swap, quantised to the next beat |

For Route A: put mosh clips on BeatStep Pro pads and the clean clips on the next row.

**Layering and transitions**

- Use the mosh as a transition between two clips, or lay it over a clean image at low opacity. Wet over dry in Normal. Add or Screen would double the static parts and wash out the blocks.
- "Mosh only where it moves": use the motion length as the wet alpha, so still regions show the dry image. Good on a camera pointed at a static stage.
- The cut back to clean is a hard cut on the downbeat, never a crossfade. A crossfade reads as a broken transition.
- Build-up: gain 1 to 2.5 over 8 bars, keyframes stop for the last 4 bars so the picture decays, green blocks rise on the last bar.
- Drop: keyframe on the downbeat to the new source, hold for 2 beats at gain 2 for a bloom, release, then normal moshing with a keyframe every bar.
- Break: switch to gradient flow with high melt for slow melting instead of blocks. Or play Route A blooms on a clean loop.

**Tips**

- Never run one mosh longer than 8 bars without a keyframe or refresh burst. Cap hold at 4 bars, then force a keyframe.
- Source matters. Low-contrast or static sources give no motion. Noisy cameras give random vectors, so raise the flat threshold.
- Frame rate: vectors are in pixels per frame. If output drops from 60 to 30 fps, the mosh looks twice as fast.
- LED walls: crush blacks fully and add contrast, since held frames pile up into a grey haze. Scale block size with the canvas (about canvas height / 68 as the default, then 2x and 4x). On coarse outdoor walls use 32 to 64.
- Route B cost at 1080p, desktop GPU, **estimates only, measure on your machine**: full block search about 1 to 2 ms (about 15 M reads with coarse-to-fine), mosh pass about 0.2 ms, under 25 MB VRAM.
- Keep Route B inside one app when possible. Each Spout hop adds a frame of latency.
- **Test before the gig:** Route A clips from the actual DXV files on the show machine, and Route B at the real output resolution and frame rate.

## Variations

1. Mosh between two live sources: the kick swaps the source while old pixels keep moving.
2. Audio-driven motion vectors: bass energy drives smear direction and strength.
3. Per-band moshing: low bands mosh large blocks, highs mosh small ones.
4. Mosh in UE5 or Niagara: feed the 3D scene through the ring-buffer smear and move the camera to create motion.
5. Hold and bloom: freeze a P-frame on a held note, release with a snap back to a clean image on the drop.
6. Stylised colour: push the chroma-slipped sample toward magenta and the luma sample toward cyan for a codec colour split.

## Did you know?

- *Monster Movie* is reported to be in the Smithsonian's permanent collection, and Kanye West reportedly saw it in a gallery ([New Atlas](https://newatlas.com/art-technology-digital-datamoshing/48152)).
- A 2009 Fader article, citing a letter from a Deitch gallery director, credits Murata with developing the technique used in both music videos. It says he showed it at *Mailorder Monsters* (Deitch Projects, September 2007) ([The Fader](https://www.thefader.com/2009/02/25/speaking-of-kanye-meet-the-dude-who-did-not-direct-his-latest-video)).
- The Kanye video took 45 to 50 renders because each I-frame deletion gave a different result ([Motionographer](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)).
- Datamosh 2 is an After Effects plugin with 60+ algorithms ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).

## Sources

- [Spotlight FX: What is datamoshing](https://spotlightfx.com/blog/what-is-datamoshing)
- [Wikipedia: Compression artifact](https://en.wikipedia.org/wiki/Compression_artifact)
- [New Atlas](https://newatlas.com/art-technology-digital-datamoshing/48152)
- [Motionographer, 2009](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)
- [The Fader, 2009](https://www.thefader.com/2009/02/25/speaking-of-kanye-meet-the-dude-who-did-not-direct-his-latest-video)
- [EAI: Monster Movie](https://www.eai.org/titles/monster-movie/video-intro)
- [UT Landmarks: Takeshi Murata](https://landmarks.utexas.edu/video-art/takeshi-murata)
- [huff](https://schwwaaa.github.io/huff/)
- [Derivative: Datamoshing in TouchDesigner](https://derivative.ca/community-post/datamoshing-touchdesigner-%E2%80%93-part-1/65167)
- [Derivative: Optical Flow TOP](https://derivative.ca/UserGuide/Optical_Flow_TOP)
- [Wikimedia Commons lead image](https://commons.wikimedia.org/wiki/File:Decodierfehler_Videowiedergabe_4K-Video_20230824.png)
- [tiberiuiancu/datamoshing](https://github.com/tiberiuiancu/datamoshing), [mosh.py](https://raw.githubusercontent.com/tiberiuiancu/datamoshing/master/mosh.py)
- [FFglitch](https://ffglitch.org/), [FFglitch: averaging motion vectors](https://ffglitch.org/2024/06/mv_avg_010.html)
- [Datamoshing with Optical Flow (SIGGRAPH Asia 2023 poster)](https://yaksoy.github.io/datamosh)
- [CDM: Liquidify video, optical flow GLSL datamosh](https://cdm.link/2009/03/liquidify-video-live-optical-flow-glsl-datamosh-technique/)
- [Buck: Moshing](https://buck.co/sandbox/moshing)
- [Notch: Motion Datamosh](https://manual.notch.one/0.9.23/en/docs/nodes/post-fx/stylisation/motion-datamosh/)
- [Resolume: Alley](https://resolume.com/article/81), [Resolume DXV](https://www.resolume.com/software/codec), [Resolume: ISF](https://resolume.com/support/en/isf), [Resolume 7.11](https://resolume.com/blog/21445)
- [Petyu Datamosh for UE](https://petyu.itch.io/uedatamosh)
- [Godot: SubViewport](https://docs.godotengine.org/en/stable/classes/class_subviewport.html)
