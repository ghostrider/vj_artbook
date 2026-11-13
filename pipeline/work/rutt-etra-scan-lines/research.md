# Rutt-Etra Scan-Line Displacement — research

## Origin
- The Rutt/Etra Video Synthesizer (also "Scan Processor") was an analog raster manipulator that changed a video image by electromagnetic deflection of the electron beam in a CRT. Co-inventors: Steve Rutt and Bill Etra ([Wikipedia](https://en.wikipedia.org/wiki/Rutt/Etra_Video_Synthesizer)).
- Timeline per a video-history interview compilation: Etra apprenticed at the WNET TV Lab in 1972, WNET granted $3,000 R&D in January 1973, and the prototype was finished by end of 1973 at about $13,000. Roughly 20 units were sold for about $7,000-8,000 (Rutt says 17, Etra says 22) ([Video History Project](https://www.videohistoryproject.org/node/5541)).
- It was described as a more controllable successor to Nam June Paik's Wobbulator, adding magnetic force to the scanning beam. The original had no recordable output and was black and white ([Video History Project](https://www.videohistoryproject.org/node/5541)).
- Etra: born 27 March 1947, died 26 August 2016; in 1971 he and Steina and Woody Vasulka started a performance space at The Kitchen ([Wikipedia: Bill Etra](https://en.wikipedia.org/wiki/Bill_Etra)).
- A patent dispute with Computer Image ended in a settlement, and legal costs ended operations ([Video History Project](https://www.videohistoryproject.org/node/5541)).

## Key works and artists
- **Woody Vasulka**: after moving to Buffalo in 1974 his work shifted toward tools like the Rutt/Etra; his solo works from 1974 include *Reminiscence* and *C-Trend* ([Wikipedia: Steina and Woody Vasulka](https://en.wikipedia.org/wiki/Steina_and_Woody_Vasulka)). The page does not say which work used which tool.
- **Bill and Louise Etra, *Narcissikon***: Etra called it his first Rutt-Etra piece ([Wikipedia: Bill Etra](https://en.wikipedia.org/wiki/Bill_Etra)).
- **Steve Nalepa, "Daytime" (2012 video)**: made with Bill Etra's own unit, said to be one of fewer than a dozen remaining; by Benton-C Bainbridge, V Owen Bush and Nalepa at dvlabs, New York. Abstract clouds and planets drift out of lace-like line tendrils ([CDM, 30 Jan 2012](https://cdm.link/?p=43365)). The same article embeds a TV on the Radio "Staring At The Sun" video made the same way.
- **Felix Turner, Rutt-Etra-Izer (2011)**: WebGL/Three.js emulation, in Chrome Experiments ([Chrome Experiments](https://experiments.withgoogle.com/rutt-etra-izer), [CAN](https://www.creativeapplications.net/project/rutt-etra-izer-scripts-webapp/)). Turner also made an audio-reactive Processing version ([Airtight](https://www.airtightinteractive.com/2011/06/rutt-etra-izer/)).

## How it works
- Original: analog signals derived from the raster drive the deflection of a CRT beam, so image brightness bends the scan lines in 3 axes; the picture is re-scanned by a camera ([Wikipedia](https://en.wikipedia.org/wiki/Rutt/Etra_Video_Synthesizer), [Video History Project](https://www.videohistoryproject.org/node/5541)).
- Software: scan the source image top to bottom, one polyline per row spaced by "Line Separation"; each vertex's z depends on pixel brightness, so bright areas lift out and dark recede ([Chrome Experiments](https://experiments.withgoogle.com/rutt-etra-izer), [Airtight](https://www.airtightinteractive.com/2011/06/rutt-etra-izer/)).
- Variant: the VDMX v002 version uses an RGB-to-XYZ displacement map instead of luma, with gradients that can be driven by audio waveform/FFT or camera input and blended between shapes ([VDMX tutorial](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques)).

## Visual language
Dense parallel luminous lines on black, which turn into mountain ridges, terrain or waves where brightness is high. Rotating the camera reveals the relief; faces and silhouettes read as contour maps. Colour is usually monochrome or colourised lines (the original was black and white per the history source above; colour came via colorizers). Motion comes from the source video plus tilt and depth.

## Live potential
- Parameters to map: line count/spacing, displacement depth, camera tilt/rotation, line thickness, colour, source clip crossfade.
- Audio: kick to depth pulse, bass level to depth or ridge amplitude, hi-hats to line flicker or thin line-width pulses, BPM to slow rotation or scroll of the source; FFT can drive the displacement map (as in the VDMX approach).
- Layering: black background plus additive lines makes it an ideal overlay on other material, or a mask over another layer.
- Boring risks (editorial judgement): static front view of a static image; too many lines becoming a grey haze; always the same face/landscape source. Needs a moving source and a moving camera.

## Variations
1. Audio-as-image: feed an FFT or waveform history strip as the source, so the relief is the sound itself.
2. Vertical and radial scan lines (concentric rings or spokes) instead of horizontal rows.
3. Recursive: render the line mesh, feed it back as the source (the original was camera re-scanned).
4. Colour from a second layer (video colourises lines) and depth from a first, like the RGB-to-XYZ approach.
5. Beat-locked rotation steps with line-count halving/doubling per bar.

## Facts
- About 20 units were built; counts differ by who you ask (17 vs 22) ([Video History Project](https://www.videohistoryproject.org/node/5541)).
- Original cost to build the prototype was about $13,000 and units sold for about $7,000-8,000 ([Video History Project](https://www.videohistoryproject.org/node/5541)).
- Etra's personal unit was one of fewer than a dozen believed to remain in 2012 ([CDM](https://cdm.link/?p=43365)).
- The first browser emulation (Turner) dates from June 2011 ([CAN](https://www.creativeapplications.net/project/rutt-etra-izer-scripts-webapp/)).
- The VDMX version can read audio waveforms/FFTs as displacement lookup ([VDMX](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques)).

## Visual references
- [Rutt-Etra-Izer demo page](https://experiments.withgoogle.com/rutt-etra-izer) — Chrome Experiments listing of the WebGL emulation, drag in an image and see the line relief.
- [Rutt-etra Video Synthesizer (YouTube, Sebastian Macchia)](https://www.youtube.com/watch?v=EZQiHuTbnes) — overview video of the original device, linked from Turner's write-up; title and channel confirmed via oEmbed.
- [Rutt Etra by SYNTHI - A (YouTube, FSOLdigital)](https://www.youtube.com/watch?v=JdDz2lcpPj0) — title and channel confirmed via oEmbed; content not viewed.
- [CDM: Steve Nalepa "Daytime"](https://cdm.link/?p=43365) — page embedding Rutt/Etra music videos.

## Sources
- [Wikipedia: Rutt/Etra Video Synthesizer](https://en.wikipedia.org/wiki/Rutt/Etra_Video_Synthesizer)
- [Wikipedia: Bill Etra](https://en.wikipedia.org/wiki/Bill_Etra)
- [Wikipedia: Steina and Woody Vasulka](https://en.wikipedia.org/wiki/Steina_and_Woody_Vasulka)
- [Video History Project: Notes Toward a History of Image-Processed Video](https://www.videohistoryproject.org/node/5541)
- [Chrome Experiments: Rutt-Etra-Izer](https://experiments.withgoogle.com/rutt-etra-izer)
- [Airtight Interactive: Rutt-Etra-Izer](https://www.airtightinteractive.com/2011/06/rutt-etra-izer/)
- [Creative Applications: Rutt-Etra-Izer](https://www.creativeapplications.net/project/rutt-etra-izer-scripts-webapp/)
- [CDM: Nalepa "Daytime"](https://cdm.link/?p=43365)
- [VDMX: Advanced v002 Rutt Etra techniques](https://vdmx.vidvox.net/tutorials/advanced-v002-rutt-etra-techniques)
- [EAI: Noisefields](https://www.eai.org/titles/noisefields)
