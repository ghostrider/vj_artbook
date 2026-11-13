# Variable Font Kinetic Typography — research

## Origin
- Variable fonts were announced as OpenType 1.8 on 14 September 2016 at the ATypI conference in Warsaw, jointly by Google, Microsoft, Apple and Adobe. Google proposed reviving Apple's TrueType GX variations. Microsoft led the spec, Apple contributed its tech, Adobe made CFF2, Google did tooling and testing ([Google Open Source Blog](https://opensource.googleblog.com/2016/09/introducing-opentype-font-variations.html)).
- The motive was efficiency: many weights and widths in one file. Animation became a side effect, because axis values are continuous and CSS can animate them ([variablefonts.io](https://variablefonts.io/about-variable-fonts/), [CSS-IRL](https://css-irl.info/variable-font-animation-with-css-and-splitting-js/)).

## Key works and artists
- **Kablammo** (Vectro Type Foundry for Google; Travis Kochel, Lizy Gershenzon). A variable font with a Morph axis (0 to 60) that changes the letter shapes through four named states: Zoink, Bloop, Splat, Eek. Contextual alternates vary repeated letters ([Google Design](https://design.google/library/kablammo-the-dancing-font-from-outer-space)).
- **COLLINS x San Francisco Symphony.** The identity uses variable type so each character changes form in reaction to sound and music. The audio-responsive "Symphosizer" is a web platform ([Typeroom](https://www.typeroom.eu/collins-san-francisco-symphony-variable-type)).
- **Animated pixelated variable fonts** (Toshi Omagari, 2017). Short film clips (a cat, Chaplin, Muybridge motion studies, a goldfish) play by sliding one axis. Each pixel is an outline whose size shows brightness ([tosche.net](https://tosche.net/AnimatedPixelatedVF/)).
- **Variable Time** (studio Pointer with type designer Celine Hurka). Open-source tool for type in motion. Parameters can be driven by Volume, Pitch or Clarity with min/max and smoothing. It exports MP4 or frames ([Variable Time](https://variabletime.pointer.click/)).
- **Type Physics+** (type.today). Browser tool where words bounce, orbit or collapse, variable axes animate with the motion, hotkeys trigger effects, and it can record ([Type Physics+](https://typephysics.type.today/)).
- **Gluk's audio visualizers** (2025). A variable colour font and a second variable font used as audio visualizers ([TypeDrawers](https://typedrawers.com/discussion/comment/70367)).

## How it works
A font file stores a few extreme designs ("masters") plus the rules to blend them. Each axis has a min and max and any value between is valid ([variablefonts.io](https://variablefonts.io/about-variable-fonts/)). Registered axes: `wght` weight, `wdth` width, `ital`, `slnt` slant, `opsz` optical size. Designers can add custom uppercase axes (e.g. GRAD, CASL). Axes are set per text run (CSS `font-variation-settings`) and can be animated continuously, per character if the text is split into spans ([CSS-IRL](https://css-irl.info/variable-font-animation-with-css-and-splitting-js/)). Kablammo shows the extreme case: a custom axis that morphs the letter shapes themselves ([Google Design](https://design.google/library/kablammo-the-dancing-font-from-outer-space)).

## Visual language
Letters that breathe, swell, squash and lean without the jumps of swapping fonts. Per-letter phase offsets create waves running through a word. Weight and width changes reflow lines, so text feels elastic. Morph-style fonts feel playful and cartoonish; classical axes feel editorial and calm. Bold monochrome type fills a big screen well; type is also readable as a texture when oversized and cropped.

## Live potential
- Mapping idea (my design suggestion): kick drives weight (impact), bass or low band drives width, hi-hats drive slant or a small jitter, BPM drives a per-letter wave phase, MIDI faders set axis min/max, smoothing and which axis is active.
- Variable Time's audio mapping (volume, pitch, clarity with smoothing) shows the same model works in practice ([Variable Time](https://variabletime.pointer.click/)).
- Layering: use white type on black as a luma or alpha mask for other video (text as a window), or as a displacement or feedback source.
- Tooling gap: TouchDesigner has no confirmed native variable axis control. A 2018 forum thread suggests driving CSS through the Web Render TOP as a workaround ([Derivative forum](https://forum.derivative.ca/t/variable-typography/11167)). Resolume support was not found. A practical route is a browser or Rust/web page rendering text and sending frames via Spout or NDI (my suggestion, not verified here).
- Boring risks: pure amplitude mapping feels mechanical (my judgement). Avoid constant motion on every letter; vary rhythm, hold, then hit. Legibility is optional in VJing, so go oversized.

## Variations
1. Lyric or keyword strobe: one word per beat, weight punches on the kick.
2. Per-letter phase wave locked to BPM, travelling along the word.
3. Morph-axis fonts (Kablammo-style) as characters that "act" on the drops.
4. Text as mask or window over live video, axes driven by faders.
5. Physics typography: letters bounce and collide, axes track velocity (Type Physics+ concept).
6. Pixel-axis font: play a looping clip as a one-axis font (Omagari's idea) for a typographic video layer.

## Facts
- OpenType 1.8 variable fonts were announced 14 Sep 2016 by four companies at once ([Google Open Source Blog](https://opensource.googleblog.com/2016/09/introducing-opentype-font-variations.html)).
- Only five axes are registered (wght, wdth, ital, slnt, opsz); everything else is custom ([variablefonts.io](https://variablefonts.io/about-variable-fonts/)).
- Kablammo took about a year and a half from idea to final glyphs ([Google Design](https://design.google/library/kablammo-the-dancing-font-from-outer-space)).
- A whole animation can live inside a font: Omagari's 2017 demo plays clips by changing one axis ([tosche.net](https://tosche.net/AnimatedPixelatedVF/)).
- Variable Time's MP4 export is capped at 1920x1080 total pixels ([Variable Time](https://variabletime.pointer.click/)).
- A single variable file is usually larger than one regular font file, so it only pays off when the axes are used ([CSS-IRL](https://css-irl.info/variable-font-animation-with-css-and-splitting-js/)).

## Visual references
- [Kablammo, Google Design](https://design.google/library/kablammo-the-dancing-font-from-outer-space) — morphing letterforms of the Morph axis.
- [Animated pixelated variable fonts](https://tosche.net/AnimatedPixelatedVF/) — film clips played by one font axis.
- [Type Physics+](https://typephysics.type.today/) — interactive words bouncing with axes animating.
- [Variable Time](https://variabletime.pointer.click/) — open-source type-in-motion tool with audio mapping.
- [Variable, color font as audio visualizer (YouTube, channel Gluk)](https://youtu.be/qAvhR4Owsxk) — variable colour font reacting to audio; title confirmed via oEmbed.
- [Variable font (WiktoriV) as audio visualizer (YouTube, channel Gluk)](https://youtu.be/biNWMPaUkYA) — second audio visualizer test; title confirmed via oEmbed.

## Sources
- https://opensource.googleblog.com/2016/09/introducing-opentype-font-variations.html
- https://variablefonts.io/about-variable-fonts/
- https://design.google/library/kablammo-the-dancing-font-from-outer-space
- https://www.typeroom.eu/collins-san-francisco-symphony-variable-type
- https://typedrawers.com/discussion/comment/70367
- https://typephysics.type.today/
- https://variabletime.pointer.click/
- https://tosche.net/AnimatedPixelatedVF/
- https://css-irl.info/variable-font-animation-with-css-and-splitting-js/
- https://forum.derivative.ca/t/variable-typography/11167
