# VHS Tape Degradation — research

## Origin
- JVC introduced VHS on 9 September 1976; the first VHS recorder, the HR-3300, went on sale in Japan on 31 October 1976 ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)).
- The look comes from the format's physical limits: a spinning two-head drum writes diagonal tracks on magnetic tape, luma is stored frequency-modulated, and chroma is squeezed down to a low "color-under" band ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)). Wear, stretching and misalignment then add dropouts, tracking errors and noise.
- Dot crawl and rainbow artefacts belong to the composite video signal that VHS decks output, where luma and chroma share one frequency range ([Wikipedia: Dot crawl](https://en.wikipedia.org/wiki/Dot_crawl)).
- As an aesthetic choice it is a "retro" move: Rozenkrantz describes an "age of retrospectacle" in which simulating analogue video is a dominant visual trend, and argues the nostalgia is largely a digital phenomenon because digitisation keeps the signs of decay ([Alphaville, Issue 12](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html)).

## Key works and artists
- **Mark Leckey, *Fiorucci Made Me Hardcore* (1999)**: a 15-minute video built only from found tape footage of UK dance floors from the 1970s to the 1990s, first shown at the ICA London ([Wikipedia](https://en.wikipedia.org/wiki/Fiorucci_Made_Me_Hardcore), [Frieze](https://frieze.com/article/mark-leckeys-iconic-video-fiorucci-made-me-hardcore-returns-youtube-after-copyright-claim)). Discussed as a case of analogue-video aesthetics in [Rozenkrantz](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html).
- **Chase & Status, "Blind Faith", dir. Daniel Wolfe (2010)**: a staged rave recreation shot to pass as faux home video of a 1990 night out in Manchester, so the degraded-tape look is a deliberate costume for new footage ([Promonews](https://www.promonews.tv/videos/2010/12/08/chase-status-feat-liam-bailey%E2%80%99s-blind-faith-daniel-wolfe/7134)). Rozenkrantz also lists it, with Joey Bada$$ "Fromdatomb$" (2012) and the film *Kung Fury* (2015), where tracking errors and dropouts are placed on purpose ([Alphaville](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html)).
- **Nam June Paik, Bruce Nauman, David Hall**: the Apollo piece on video art and nostalgia names Paik's *TV Buddha* (1974), Nauman's *Clown Torture* (1987) and Hall's *1001 TV Sets (End Piece)* (2012), and glitch artist Rob Feulner as people working with the medium's analogue openness ([Apollo](https://apollo-magazine.com/nostalgia-vhs-television-video-art/)).
- **DecayingElectronics, VHS Dirty Video Mixer**: a hardware box that takes two RCA video inputs (VHS player, console) and glitches them with a knob ([Boing Boing](https://boingboing.net/2021/05/31/glitch-video-the-hardware-way-with-the-vhs-dirty-video-mixer.html/amp)).
- **Pushkar Brand**: VHS look driven live from Resolume Arena plus analogue glitch synced to Eurorack (see videos below).

## How it works
- **Chroma is low-resolution.** Baseband chroma bandwidth on VHS is about 300 kHz against up to 3 MHz for luma, so colour is much blurrier than brightness; standard VHS resolves roughly 240 TV lines ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)). This gives chroma bleed and soft luma.
- **Helical scan and head switching.** Two heads on a drum each write one field; the machine must switch between them, normally hidden in the vertical blanking interval ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)). This is the source of the noisy, skewed strip at the bottom of the frame.
- **Tracking error.** Playback heads fail to follow the recorded tracks ([AV Artifact Atlas](https://www.avartifactatlas.com/artifacts/tracking_error.html)); on screen this appears as a noisy band that scrolls and tears the picture.
- **Dot crawl and rainbows.** Luma/chroma crosstalk in composite video makes moving checkerboard patterns on colour edges, and rainbow colours on fine detail ([Wikipedia: Dot crawl](https://en.wikipedia.org/wiki/Dot_crawl)).
- **Shader recipe seen in the wild.** Existing shaders shift horizontal bars over time, add blocky line noise and offset the colour channels ([Godot Shaders: VHS Glitch](https://godotshaders.com/shader/vhs-glitch/)); a TouchDesigner GLSL tutorial combines chromatic aberration, colour drift, noise and temporal jitter ([Derivative](https://derivative.ca/community-post/tutorial/touchdesigner-glitch-effect-glsl/73842)).

## Visual language
- Soft, smeared colour with a sharper-edged but low-res luma, slight ghosting, scanline structure, 4:3 feeling even in 16:9.
- Horizontal motion: scrolling tracking bands, line wobble, bottom-edge head-switching skew, dropout white flecks.
- Rainbow fringes and crawling dots on saturated edges; red-cyan colour drift.
- Rozenkrantz lists low resolution, dropout, tracking lines and rainbow effects as the traits that read as "the past" ([Alphaville](https://www.alphavillejournal.com/Issue12/HTML/ArticleRozenkratz.html)).
- The mood is nostalgic and haunted: the image looks like a memory that is physically decaying.

## Live potential
(Design suggestions from this research, not sourced claims.)
- **Kick:** trigger a tracking band that sweeps down the frame, or a short horizontal tear; kick-scaled jitter.
- **Bass / sub:** drive chroma bleed distance and luma softness; slow wobble of line offset.
- **Hi-hats:** fine dropout flecks and noise-line density; dot-crawl intensity.
- **BPM:** lock the tracking band scroll speed to bar length; "pause-frame" freeze on the one.
- **Faders / KeyStep / BeatStep Pro:** wear amount (clean to destroyed), bleed, tracking band count, noise, head-switch height; a pad for a one-shot tape-chew glitch.
- **Layering:** works as a final treatment over any source (live camera, UE via Spout/NDI); also stacks with feedback, datamosh and CRT looks. Put it last in the chain so everything below inherits the tape.
- **Boring risk:** a constant static filter. Without evolution it looks like a stock overlay; automate the wear curve across a set, and vary band speed and position instead of a fixed offset.

## Variations
- **Wear arc:** map "tape age" to a macro that goes from fresh rental to a chewed tape over a track or set.
- **Generation loss:** feed the output back through the filter several times for copy-of-a-copy decay.
- **Pitch-coloured bleed:** let the MIDI note choose the hue direction of chroma smear.
- **Selective tape:** apply the effect only through a mask (a Wire or UE shape), so a clean image shows through a degraded window.
- **Hybrid hardware pass:** run the output through a physical glitch device or analogue mixer such as the Dirty Video Mixer idea and re-capture it.

## Facts
- JVC launched VHS in September 1976 ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)).
- VHS chroma bandwidth is about a tenth of luma bandwidth (300 kHz vs up to 3 MHz) ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)).
- The VHS drum spins at 1,798.2 rpm for NTSC and 1,500 rpm for PAL, one revolution per frame ([Wikipedia: VHS](https://en.wikipedia.org/wiki/VHS)).
- Dot crawl is mostly associated with NTSC; digital comb filters in the 1990s greatly reduced it, but no comb filter removes all NTSC artefacts ([Wikipedia: Dot crawl](https://en.wikipedia.org/wiki/Dot_crawl)).
- "Blind Faith" (Chase & Status) looks like archive footage but is a recreation ([Promonews](https://www.promonews.tv/videos/2010/12/08/chase-status-feat-liam-bailey%E2%80%99s-blind-faith-daniel-wolfe/7134)).
- Leckey's *Fiorucci Made Me Hardcore* had over 203,000 YouTube views before a copyright claim briefly removed it ([Frieze](https://frieze.com/article/mark-leckeys-iconic-video-fiorucci-made-me-hardcore-returns-youtube-after-copyright-claim)).

## Visual references
- [Pushkar Brand, Honey Breadcrump (VHS Resolume Arena and analog glitch sync to Eurorack)](https://www.youtube.com/watch?v=P9Jf11zMSoo) — VHS look made in Resolume Arena with analogue glitch; channel PUSHKAR BRAND.
- [Pushkar Brand, Minor Lights Ahead (VHS Resolume videobending glitch videoclip)](https://www.youtube.com/watch?v=1YwEu6xkzRk) — video-bent VHS glitch clip; channel PUSHKAR BRAND.
- [VHS Tracking Effect Tutorial (After Effects)](https://www.youtube.com/watch?v=binY4qJwVto) — tutorial for the tracking-band look; channel Frikanih FX.
- [Godot Shaders: VHS Glitch](https://godotshaders.com/shader/vhs-glitch/) — shader page with preview of bar shifts, line noise and channel offset.
- [TD-VHShader on GitHub](https://github.com/ogrew/td-vhshader) — TouchDesigner project of 12 combined VHS shaders.
- [Boing Boing: VHS Dirty Video Mixer](https://boingboing.net/2021/05/31/glitch-video-the-hardware-way-with-the-vhs-dirty-video-mixer.html/amp) — hardware glitch device and demo.

## Lead image
- kind: similar
- image_url: https://images.unsplash.com/photo-1610337673044-720471f83677?w=1400
- page_url: https://unsplash.com/photos/colorful-digital-glitch-static-pattern-0W4XLGITrHg
- shows: a photographed television glitch with static-like colour patterns (the page calls it "Television Glitch #3")
- author: Michael Dziedzic
- license: Unsplash License, https://unsplash.com/license

## Quotation images
none

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
