# Datamosh Compression Glitch — research

## Origin
Datamoshing grew out of accidental glitches in early digital video codecs such as DivX, often caused by signal loss or faulty encoding. In the early 2000s artists began triggering these glitches on purpose ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing), [New Atlas](https://newatlas.com/art-technology-digital-datamoshing/48152)). Wikipedia names Bertrand Planes and Christian Jacquemin (DivXPrime, 2006), Sven König, Takeshi Murata, Jacques Perconte and Paul B. Davis (Paperrad) as pioneers ([Wikipedia: Compression artifact](https://en.wikipedia.org/wiki/Compression_artifact)). Takeshi Murata's *Monster Movie* (2005) is the best-known early art piece ([EAI](https://www.eai.org/titles/monster-movie/video-intro)). The technique reached the mainstream in early 2009 with Chairlift's "Evident Utensil" (dir. Ray Tintori) and Kanye West's "Welcome to Heartbreak" (dir. NABIL) ([Motionographer, 19 Feb 2009](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)).

## Key works and artists
- **Takeshi Murata, *Monster Movie* (2005)**: 3:55, built frame by frame from footage of the 1981 film *Caveman*, which dissolves into churning colour and shape; sound by Plate Tectonics ([EAI](https://www.eai.org/titles/monster-movie/video-intro)). Also *Untitled (Pink Dot)* (2007), made from *First Blood* ([UT Landmarks](https://landmarks.utexas.edu/video-art/takeshi-murata)).
- **Chairlift, "Evident Utensil" (2009)**, dir. Ray Tintori: the band breaks through the frame in a mosh; Murata is named as a key influence ([Motionographer](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)).
- **Kanye West, "Welcome to Heartbreak" (2009)**, dir. NABIL, post by Ghost Town Media: a grittier mosh. The team shot on green screen, rendered uncompressed, compressed to AVI and deleted chosen I-frames, with about 45-50 versions because each render was unique ([Motionographer](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)).
- **Paperrad, David O'Reilly, Owi Mahn and Laura Baginski, Kris Moyes, Sven König**: other early explorers listed in the same article ([Motionographer](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)).

## How it works
Video codecs store an I-frame (a full image, like a keyframe), then P-frames that only store changes from the previous frame, and B-frames that predict from earlier and later frames. If an I-frame is removed or corrupted, the decoder applies the remaining motion data to the wrong picture, so pixels from the old scene move according to the new scene's motion: colour melting, ghosting, smeared motion ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).
Classic methods:
- **I-frame removal**: the old clip bleeds into the next, a morph-like transition.
- **P-frame duplication**: repeating a P-frame compounds its motion, giving bloom and trails.
- **Motion hijacking, frame interleaving, transcoding tricks** ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).

Real-time emulation does not need a codec. The tool huff, for example, displaces image tiles over time from a 60-frame ring buffer, with smear and scatter controls ([huff](https://schwwaaa.github.io/huff/)). TouchDesigner has community datamosh builds too ([Derivative](https://derivative.ca/community-post/datamoshing-touchdesigner-%E2%80%93-part-1/65167)).

## Visual language
Blocky macroblock tiles, colour bleeding, smeared and stretched motion, ghost trails, and one image wearing another's motion. Frames break apart and reform thirty times a second ([EAI](https://www.eai.org/titles/monster-movie/video-intro)). It reads as digital decay, with hard block edges next to soft melting colour.

## Live potential
- **Kick**: reset or force a "keyframe" (cut to the new clip while keeping the old pixels), so each kick starts a new mosh.
- **Bass or sustain**: smear amount or motion-vector gain; hold P-frame repeat while the bass sustains.
- **Hi-hats**: block-size jitter, tile scatter, small corruption bursts.
- **BPM**: freeze and release on bar boundaries; ring-buffer length in beats.
- **MIDI faders**: glitch amount, freeze, block size, source A/B crossfade. huff already accepts MIDI CC/notes and OSC on UDP 9000 and outputs through Spout to Resolume or TouchDesigner ([huff](https://schwwaaa.github.io/huff/)).
- **Layering**: use the mosh as a transition between two clips, or put it on top of a clean image at low opacity; it works as an effect on any source in Resolume.
- **Boring risk**: constant full-strength smear turns into mush. It needs reset moments, and moving, high-contrast source material.

## Variations
1. Mosh between two live sources: the kick swaps the source while old pixels keep moving.
2. Audio-driven motion vectors: bass energy drives the smear direction and strength.
3. Per-band moshing: low bands mosh large blocks, highs mosh small ones.
4. Mosh in UE5 or Niagara: feed the 3D scene through the ring-buffer smear and move the camera to create the motion.
5. Hold-and-bloom: freeze a P-frame on a held note, and release with a snap back to a clean image on the drop.

## Facts
- *Monster Movie* is reported to be in the Smithsonian's permanent collection ([New Atlas](https://newatlas.com/art-technology-digital-datamoshing/48152)).
- Kanye West reportedly saw *Monster Movie* in a gallery ([New Atlas](https://newatlas.com/art-technology-digital-datamoshing/48152)).
- A 2009 Fader article, citing a letter from a Deitch gallery director, credits Murata with developing the technique used in both music videos and says he showed it at *Mailorder Monsters* (Deitch Projects, September 2007) ([The Fader](https://www.thefader.com/2009/02/25/speaking-of-kanye-meet-the-dude-who-did-not-direct-his-latest-video)).
- The Kanye video took 45-50 renders because each I-frame deletion gave a different result ([Motionographer](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/)).
- Avidemux (2.7.0+) can copy and paste P-frames manually; Datamosh 2 is an After Effects plugin with 60+ algorithms ([Spotlight FX](https://spotlightfx.com/blog/what-is-datamoshing)).
- Murata also made *No Match* (2009), a game-show clip stretched to 999 seconds ([UT Landmarks](https://landmarks.utexas.edu/video-art/takeshi-murata)).

## Visual references
- [Takeshi Murata at EAI: Monster Movie](https://www.eai.org/titles/monster-movie/video-intro) — distributor page for the 2005 work.
- [Motionographer: Tintori and NABIL](https://motionographer.com/2009/02/19/tintori-and-nabil-breaking-your-internets/) — article with stills and links to both 2009 videos and the artist works.
- [huff](https://schwwaaa.github.io/huff/) — real-time datamosh app, shows the live-tool look.

## Lead image
- file_page: https://commons.wikimedia.org/wiki/File:Decodierfehler_Videowiedergabe_4K-Video_20230824.png
- image_url: https://upload.wikimedia.org/wikipedia/commons/thumb/e/e7/Decodierfehler_Videowiedergabe_4K-Video_20230824.png/1280px-Decodierfehler_Videowiedergabe_4K-Video_20230824.png
- shows: A real video decoding error in a 4K player (VLC): blocks of different shapes and colours. An accidental compression glitch, not an art piece.
- author: PantheraLeo1359531
- license: Public domain (marked ineligible for copyright, machine output)
- license_url: https://commons.wikimedia.org/wiki/File:Decodierfehler_Videowiedergabe_4K-Video_20230824.png

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
- [Wikimedia Commons lead image](https://commons.wikimedia.org/wiki/File:Decodierfehler_Videowiedergabe_4K-Video_20230824.png)
